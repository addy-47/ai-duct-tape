# GCP uptime check alerts that actually notify

Build and debug Google Cloud Monitoring uptime checks and their alerting policies,
so a long outage produces a stream of notifications instead of one email.

Provider: Google Cloud (Cloud Monitoring API v3). Stack: REST + `gcloud`.

---

## 1. The failure mode this recipe exists for

**Symptom.** The site was down for many hours. The alert fired once, at the start.
No alert arrived for the rest of the outage. The retest window was short, the
threshold was breached the whole time, and the metric graph showed the breach
continuously.

**Cause.** Metric-based alerting is **edge-triggered**, not level-triggered.

- Monitoring evaluates a condition continuously.
- It sends a notification on the **transition** into a breach: an incident opens.
- While the incident stays open, the condition stays true and nothing is sent.
- The retest window (`duration`) controls **when an incident opens**. It is not a
  re-notification interval.

A policy with no repeat-notification configuration therefore notifies exactly
**once per incident**, no matter how long the breach lasts.

**Three independent gaps cause this in practice:**

| Gap | Result |
|---|---|
| `alertStrategy.notificationPrompts` = `["OPENED"]` only | No "resolved" message. You never learn the outage ended. |
| No `alertStrategy.notificationChannelStrategy` | One notification per incident, then silence. |
| `conditionThreshold.evaluationMissingData` unset | A silent check (no data) never opens an incident. |

**Fix all three** (section 6). The second gap is the one that eats a long outage.

---

## 2. APIs and CLI surfaces

Base URL: `https://monitoring.googleapis.com/v3`
All requests need `Authorization: Bearer $(gcloud auth print-access-token)`.

| Task | API | CLI |
|---|---|---|
| List uptime checks | `GET /v3/projects/{PROJECT}/uptimeCheckConfigs` | `gcloud monitoring uptime list-configs --project=PROJECT --format=json` |
| Get one uptime check | `GET /v3/projects/{PROJECT}/uptimeCheckConfigs/{CHECK_ID}` | `gcloud monitoring uptime describe CHECK_ID --project=PROJECT` |
| List alerting policies | `GET /v3/projects/{PROJECT}/alertPolicies` | `gcloud alpha monitoring policies list --project=PROJECT --format=json` |
| Get one policy | `GET /v3/projects/{PROJECT}/alertPolicies/{POLICY_ID}` | `gcloud alpha monitoring policies describe POLICY_ID --project=PROJECT` |
| Update one policy | `PATCH /v3/projects/{PROJECT}/alertPolicies/{POLICY_ID}?updateMask=FIELDS` | `gcloud alpha monitoring policies update POLICY_ID --project=PROJECT --enabled=false` |
| Update policy from a file | same | `gcloud alpha monitoring policies update POLICY_ID --policy-from-file=policy.json --fields=alertStrategy,conditions` |
| List alert instances (incidents) | `GET /v3/projects/{PROJECT}/alerts` | `gcloud monitoring alerts list --project=PROJECT --format=json` |
| List time series | `GET /v3/projects/{PROJECT}/timeSeries?filter=...&interval.startTime=...&interval.endTime=...` | `gcloud monitoring time-series list --filter=... --project=PROJECT` |
| List notification channels | `GET /v3/projects/{PROJECT}/notificationChannels` | `gcloud alpha monitoring channels list --project=PROJECT --format=json` |
| List snoozes | `GET /v3/projects/{PROJECT}/snoozes` | `gcloud monitoring snoozes list --project=PROJECT --format=json` |

Notes that save time:

- `projects.incidents.list` returns **404** on the v3 API. Use `projects.alerts.list`.
- `timeSeries.list` is a **GET** with query parameters. A JSON body with `query` /
  `interval` / `aggregation` keys is rejected with `Unknown name "query"`.
- `view` accepts `FULL` or `HEADERS` only. There is no `RAW`. `FULL` **interpolates**
  gaps onto a finer grid, so point counts are synthetic. Use `HEADERS` to count series.
- `PATCH` with `updateMask=conditions` replaces the whole `conditions` array. Include
  each condition's `name` to update in place instead of recreating it.
- Snoozes suppress notifications. Check them before diagnosing a "no notification".

---

## 3. Uptime check object

`uptimeCheckConfigs.create` / `.patch` take this shape. `name` is server-assigned;
the last path segment is the **check ID** used by alert policy filters.

```json
{
  "name": "projects/PROJECT/uptimeCheckConfigs/CHECK_ID",
  "displayName": "human readable name",
  "monitoredResource": {
    "type": "uptime_url",
    "labels": { "host": "example.com", "project_id": "PROJECT" }
  },
  "httpCheck": {
    "path": "/",
    "port": 443,
    "requestMethod": "GET",
    "useSsl": true,
    "validateSsl": true,
    "acceptedResponseStatusCodes": { "statusClass": "2xx" },
    "contentMatchers": [],
    "authInfo": { "type": "NONE" }
  },
  "period": "60s",
  "timeout": "10s",
  "checkerType": "STATIC_IP_CHECKERS",
  "selectedRegions": [],
  "logCheckFailures": true
}
```

| Field | Meaning |
|---|---|
| `name` | Resource path. Last segment = check ID. Read it from here, never guess it. |
| `displayName` | Label in the console. The check ID is derived from it. |
| `monitoredResource.type` | `uptime_url` for HTTP/TCP probes. The metric is written against this resource type. |
| `monitoredResource.labels.host` | The hostname being probed. |
| `httpCheck.path` | Path probed. Only this path is covered. |
| `httpCheck.port` / `useSsl` | Target port and TLS on/off. |
| `httpCheck.requestMethod` | HTTP method. |
| `httpCheck.validateSsl` | When true, a bad certificate counts as a failure. |
| `httpCheck.acceptedResponseStatusCodes` | Which codes count as success. **Default is any 2xx.** A 200 error page counts as UP. |
| `httpCheck.contentMatchers` | Optional match on the response body. Empty means status code only. |
| `period` | Probe interval. `60s` is the practical minimum. |
| `timeout` | Per-probe deadline. A response slower than this counts as a failure. |
| `checkerType` | `STATIC_IP_CHECKERS` (Google-hosted global regions) or `DYNAMIC_IP_CHECKERS` (one region from a fixed list). |
| `selectedRegions` | Pin the checker regions. Empty = Google picks the set. |
| `logCheckFailures` | Write probe failures to Cloud Logging. |

**Metric produced.** `monitoring.googleapis.com/uptime_check/check_passed`,
type `BOOL`, resource type `uptime_url`.

| Metric label | Meaning |
|---|---|
| `check_id` | Which check produced the point. **This is the filter key for alert policies.** |
| `checker_location` | Which region ran the probe. **Each region is a separate time series.** |
| `checked_resource_id` | The probed host. |

The number of checker locations is the number of series. Count them before you
reason about thresholds.

---

## 4. Alert policy object

```json
{
  "name": "projects/PROJECT/alertPolicies/POLICY_ID",
  "displayName": "policy name",
  "enabled": true,
  "combiner": "OR",
  "severity": "CRITICAL",
  "documentation": { "content": "markdown shown in the notification", "mimeType": "text/markdown" },
  "userLabels": { "app": "service-name" },
  "conditions": [
    {
      "name": "projects/PROJECT/alertPolicies/POLICY_ID/conditions/CONDITION_ID",
      "displayName": "condition name",
      "conditionThreshold": { "...": "see below" }
    }
  ],
  "notificationChannels": ["projects/PROJECT/notificationChannels/CHANNEL_ID"],
  "alertStrategy": { "...": "see below" }
}
```

| Field | Meaning |
|---|---|
| `enabled` | `false` keeps the policy but stops all evaluation and notifications. |
| `combiner` | `OR` / `AND` / `AND_WITH_MATCHING_RESOURCE` across conditions. |
| `severity` | `CRITICAL` / `ERROR` / `WARNING` / `INFO`. Shown in the incident. |
| `documentation` | Markdown injected into the notification. Put the runbook here. |
| `userLabels` | Key/value labels carried into incidents and notifications. |
| `notificationChannels` | Channels that receive notifications. An empty list means the policy fires and notifies nobody. |

### 4.1 `conditionThreshold` (metric-threshold condition)

```json
{
  "filter": "metric.type=\"monitoring.googleapis.com/uptime_check/check_passed\" AND metric.label.check_id=\"CHECK_ID\" AND resource.type=\"uptime_url\"",
  "aggregations": [
    {
      "alignmentPeriod": "300s",
      "perSeriesAligner": "ALIGN_FRACTION_TRUE",
      "crossSeriesReducer": "REDUCE_MEAN"
    }
  ],
  "comparison": "COMPARISON_LT",
  "thresholdValue": 0.8,
  "duration": "300s",
  "trigger": { "count": 1 },
  "evaluationMissingData": "EVALUATION_MISSING_DATA_ACTIVE"
}
```

| Field | Meaning |
|---|---|
| `filter` | Selects the series to evaluate. Must set `metric.type` and the resource type. Scoping by `metric.label.check_id` is what keeps one policy per check. |
| `aggregations[].alignmentPeriod` | Bucket size per series. Minimum `60s`. At `60s` with a `60s` probe period, each window holds one point, so a fraction aligner degrades to that point. Use `300s` or more. |
| `aggregations[].perSeriesAligner` | Reduces points inside one series to one value. For a BOOL metric use `ALIGN_FRACTION_TRUE` (fraction of points that were true). `ALIGN_NEXT_OLDER` and `ALIGN_MEAN` also work. |
| `aggregations[].crossSeriesReducer` | Combines the series into one. `REDUCE_MEAN` gives the **global pass rate**. `REDUCE_COUNT_FALSE` gives the **count of failing regions**. `REDUCE_COUNT_TRUE` gives the count of passing regions. |
| `aggregations[].groupByFields` | Which labels survive grouping. **Omitted = every series of the resource type collapses into one.** Set it (e.g. `metric.label.checker_location`) when you want one series per region. |
| `comparison` | `COMPARISON_LT` or `COMPARISON_GT`. The metric sits on the left, `thresholdValue` on the right. |
| `thresholdValue` | The number compared against. For a pass rate, `0.5` means "below half the probes passed". |
| `duration` | **Retest window.** The condition must hold for this long before an incident opens. Multiple of one minute. `0` means one measurement. Requires `duration >= 60s` to use `evaluationMissingData`. |
| `trigger.count` | How many output series must breach. With one aggregated series, `1`. |
| `evaluationMissingData` | How to treat a window with no data. See 4.2. |

### 4.2 `evaluationMissingData` values

| Value | Behaviour when data stops arriving |
|---|---|
| `EVALUATION_MISSING_DATA_UNSPECIFIED` | Same as `NO_OP`. The default. |
| `EVALUATION_MISSING_DATA_NO_OP` | Do not evaluate the condition. Open alerts stay open. **New alerts are not opened.** |
| `EVALUATION_MISSING_DATA_ACTIVE` | Evaluate the condition as **true**. The condition behaves like a metric-absence condition: no data for the retest window opens an incident. **Use this to catch a silent check.** |
| `EVALUATION_MISSING_DATA_INACTIVE` | Evaluate the condition as **false**. Open alerts close. |

### 4.3 `alertStrategy`

```json
{
  "autoClose": "3600s",
  "notificationPrompts": ["OPENED", "CLOSED"],
  "notificationChannelStrategy": [
    {
      "notificationChannelNames": ["projects/PROJECT/notificationChannels/CHANNEL_ID"],
      "renotifyInterval": "3600s"
    }
  ]
}
```

| Field | Meaning |
|---|---|
| `autoClose` | How long an incident stays open **after data stops arriving**. The close timer starts after a delay of at least 15 minutes. |
| `notificationPrompts` | `OPENED` and/or `CLOSED`. `OPENED` alone means no "resolved" message. |
| `notificationChannelStrategy[]` | Per-channel repeat notifications. |
| `notificationChannelStrategy[].notificationChannelNames` | Channels to repeat to. Must already be in the policy's `notificationChannels`. |
| `notificationChannelStrategy[].renotifyInterval` | Repeat period. **Minimum `1800s`, maximum `86400s`.** Repeat notifications are sent while the incident is open or acknowledged. |
| `notificationRateLimit` | Required only for log-based policies. |

**`notificationChannelStrategy` is the field that makes a long outage visible.**

---

## 5. Diagnose "the alert did not fire"

Run in this order. Stop when the cause is identified.

**Step 1 — Did the incident open?**

```bash
gcloud monitoring alerts list --project=PROJECT --format=json
```

Each entry has `openTime`, `closeTime`, `state`, and `policy.name`.

- **No entry at all** → the condition never became true. Continue to step 3.
- **Entry open for the whole outage** → the alert worked. The notification logic is
  the problem. Go to section 6.

**Step 2 — Was the incident open the whole time?**

`openTime` and `closeTime` are the ground truth. An incident that opened at the
start of the outage and closed after the recovery means the breach was detected
and held. A notification sent once is the expected behaviour of an unconfigured
`alertStrategy`, not a failure to detect.

**Step 3 — Did the metric actually breach?**

```bash
curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://monitoring.googleapis.com/v3/projects/PROJECT/timeSeries?filter=metric.type%3D%22monitoring.googleapis.com%2Fuptime_check%2Fcheck_passed%22%20AND%20metric.label.check_id%3D%22CHECK_ID%22&interval.startTime=2026-01-01T00:00:00Z&interval.endTime=2026-01-01T06:00:00Z&view=FULL&pageSize=200000"
```

Group the points by `checker_location` and count true/false per region. This tells
you how many regions were failing, which is the input to the threshold maths in
section 7.

**Step 4 — Check the policy fields against section 4.**

**Step 5 — Check for snoozes and disabled policies.**

```bash
gcloud monitoring snoozes list --project=PROJECT --format=json
gcloud alpha monitoring policies list --project=PROJECT --format="value(name,displayName,enabled)"
```

---

## 6. Apply the fix

PATCH the policy. Only `alertStrategy` and `evaluationMissingData` change. The
condition's other fields and the `notificationChannels` list stay as they are.

```bash
POLICY_ID=POLICY_ID

curl -X PATCH \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/json" \
  "https://monitoring.googleapis.com/v3/projects/PROJECT/alertPolicies/$POLICY_ID?updateMask=alertStrategy,conditions" \
  -d '{
    "alertStrategy": {
      "autoClose": "3600s",
      "notificationPrompts": ["OPENED", "CLOSED"],
      "notificationChannelStrategy": [{
        "notificationChannelNames": ["projects/PROJECT/notificationChannels/CHANNEL_ID"],
        "renotifyInterval": "3600s"
      }]
    },
    "conditions": [{
      "name": "projects/PROJECT/alertPolicies/POLICY_ID/conditions/CONDITION_ID",
      "displayName": "condition name",
      "conditionThreshold": {
        "filter": "metric.type=\"monitoring.googleapis.com/uptime_check/check_passed\" AND metric.label.check_id=\"CHECK_ID\" AND resource.type=\"uptime_url\"",
        "aggregations": [{
          "alignmentPeriod": "300s",
          "perSeriesAligner": "ALIGN_FRACTION_TRUE",
          "crossSeriesReducer": "REDUCE_MEAN"
        }],
        "comparison": "COMPARISON_LT",
        "thresholdValue": 0.8,
        "duration": "300s",
        "trigger": { "count": 1 },
        "evaluationMissingData": "EVALUATION_MISSING_DATA_ACTIVE"
      }
    }]
  }'
```

**Read-modify-write.** Fetch the policy first, change the fields in the response,
and PATCH that object back. A hand-written `conditions` array that drops a field
silently changes behaviour. Keep each condition's `name` so the condition is
updated in place and its ID stays stable.

Read the current object:

```bash
curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://monitoring.googleapis.com/v3/projects/PROJECT/alertPolicies/$POLICY_ID"
```

**Verify the change landed:**

```bash
curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://monitoring.googleapis.com/v3/projects/PROJECT/alertPolicies/$POLICY_ID" \
  | jq '{alertStrategy, conditions: [.conditions[] | .conditionThreshold | {thresholdValue, duration, evaluationMissingData, aggregations}], notificationChannels: (.notificationChannels | length)}'
```

Expected: `notificationPrompts` contains `CLOSED`, `notificationChannelStrategy` has
a `renotifyInterval`, and `evaluationMissingData` is `EVALUATION_MISSING_DATA_ACTIVE`.

### 6.1 CLI equivalent

`gcloud alpha monitoring policies update` reads and writes the same JSON or YAML.
With `--fields`, only the listed fields change. Without it, the whole policy is
replaced.

```bash
gcloud alpha monitoring policies update POLICY_ID \
  --project=PROJECT \
  --policy-from-file=policy.json \
  --fields=alertStrategy,conditions
```

`--policy-from-file` accepts the object from section 4, so one file can be the
source of truth for the policy and a checked-in copy stays reviewable.

---

## 7. Threshold maths for a pass-rate policy

With `REDUCE_MEAN` over `N` checker regions and no `groupByFields`, the metric is
the **global pass rate** = (sum of per-region pass fractions) / N.

Condition fires when `pass rate < thresholdValue` for the whole retest window.

| Regions fully failing | Pass rate (N=6) | Fires at threshold 0.5 | Fires at threshold 0.8 |
|---|---|---|---|
| 1 | 0.833 | No | No |
| 2 | 0.667 | No | **Yes** |
| 3 | 0.500 | No (not `< 0.5`) | **Yes** |
| 4 | 0.333 | **Yes** | **Yes** |
| 5 | 0.167 | **Yes** | **Yes** |
| 6 | 0.000 | **Yes** | **Yes** |

Rules that fall out of the table:

- **The boundary is exact.** N regions failing gives a pass rate of `(N-f)/N`.
  At threshold `0.5` with 6 regions, 3 failing regions produce exactly `0.5` and
  do **not** fire, because the comparison is strictly less-than.
- **Partial failures count fractionally.** 3 regions down plus one region at 50%
  pass gives a sum of 3.5, a pass rate of 0.583, and no alert.
- **Threshold `0.8` with 6 regions = "at least 2 regions failing".** This matches
  Google's own default uptime policy, which notifies when two regions fail.
- **A region with no data is excluded from the mean.** It is not counted as
  passing. Silent regions therefore change the denominator, not the numerator.
- **One region failing never fires** at any threshold in this table. That is the
  intended behaviour: it filters single-region network noise.

Prefer an explicit region count over a pass rate when the requirement is
"at least 2 regions failed": use `REDUCE_COUNT_FALSE` with `COMPARISON_GT`,
`thresholdValue: 1`, and `groupByFields: ["metric.label.checker_location"]`.

---

## 8. Timing model

Budget this much latency between the real failure and the notification.

| Stage | Cost |
|---|---|
| Probe detection | Up to one `period`. |
| Metric data visibility | About 2 minutes on average for uptime check data. |
| Retest window | The full `duration`, measured from the first violating window. |
| Alerting computation | Up to about 5 minutes 30 seconds. |
| Channel delivery | Seconds to a few minutes, per channel. |

Measured on a real outage: first failed probe at `10:34:50`, incident opened at
`10:40:08` (5m18s), site recovered at `05:34:40`, incident closed at `05:36:39`
(about 2 minutes). A larger `alignmentPeriod` and `duration` add directly to the
detection delay.

---

## 9. Gotcha checklist

- **The retest window is not a re-notification interval.** It gates incident
  opening only.
- **A policy with no `notificationChannelStrategy` sends one message per
  incident.** Long outages then look like "the alert stopped working".
- **`notificationPrompts: ["OPENED"]` only** means no resolution message. Add
  `CLOSED`.
- **An empty `notificationChannels` list** means the policy fires into the void.
  Check the list, not just the trigger.
- **`enabled: false`** silences a policy with no other trace. Check it.
- **A snooze** suppresses notifications with no other trace. Check it.
- **`groupByFields` omitted collapses every series into one.** Read the
  aggregation as a global aggregate, not a per-region one.
- **`alignmentPeriod: 60s` with a `60s` probe period** puts one point in each
  window. A fraction aligner then means "that one point". Raise it to `300s`.
- **`view=FULL` interpolates** onto a finer grid. Point counts are synthetic.
  Use `view=HEADERS` to count series.
- **Any 2xx counts as UP.** A 200 error page is a pass. Add `acceptedResponseStatusCodes`
  or `contentMatchers`.
- **A slow site counts as down** when it exceeds `timeout`.
- **A certificate problem counts as down** when `validateSsl` is true.
- **Data latency is real.** The chart can show a breach before the incident opens.
- **Editing a policy takes minutes to propagate.** Old notifications can arrive
  after the change.
- **`PATCH` with `updateMask=conditions` replaces the array.** Include condition
  `name` values to update in place.

---

## 10. Reference

- Cloud Monitoring API v3: `projects.alertPolicies`, `projects.alerts`,
  `projects.uptimeCheckConfigs`, `projects.timeSeries`, `projects.notificationChannels`
- Console path: Monitoring → Alerting → Alert policies / Uptime checks
- CLI: `gcloud monitoring alerts list`, `gcloud alpha monitoring policies list`,
  `gcloud monitoring uptime list-configs`, `gcloud monitoring snoozes list`
