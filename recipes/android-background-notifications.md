# Background notifications in a Tauri Android app

Reminder-style local notifications that fire on time with the app closed.
No server, no push, no new dependencies. Tauri v2, Android, Kotlin + Rust + TS.

## 1. Manifest

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"
    android:maxSdkVersion="32" />
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.VIBRATE" />

<receiver android:name=".AlarmReceiver" android:exported="false">
    <intent-filter>
        <action android:name="com.example.app.action.REMINDER_ALARM" />
    </intent-filter>
</receiver>
<receiver android:name=".BootReceiver" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

`USE_EXACT_ALARM` is auto-granted (no user action) but needs a Play Console
declaration with justification. Without it, exact alarms need a manual grant in
Settings → Apps → Alarms & reminders (denied by default on Android 14+).

## 2. Kotlin scheduler

One `object` owns everything. One alarm per item, self-chaining:
each fire posts the notification **and** arms the next slot.

```kotlin
object ReminderScheduler {
    @JvmStatic fun applyPlan(context: Context, planJson: String)  // store + arm next per item
    @JvmStatic fun cancelAll(context: Context)
    @JvmStatic fun canScheduleExact(context: Context): Boolean
    @JvmStatic fun openExactAlarmSettings(context: Context)       // ACTION_REQUEST_SCHEDULE_EXACT_ALARM intent
    @JvmStatic fun diagnostics(context: Context): String          // JSON: armed count, next slot, last error
    @JvmStatic fun scheduleTestAlarm(context: Context): String    // one-shot +60s, outside the plan
}
```

Rules inside:

- `arm()`: `setExactAndAllowWhileIdle` when `canScheduleExact()`, else
  `setAndAllowWhileIdle`. `PendingIntent.FLAG_UPDATE_CURRENT or FLAG_IMMUTABLE`,
  request code derived from the item id.
- `handleFire()`: skip if the slot is done or dated to a previous day;
  otherwise post, then `armNext()`. Never silently drop a same-day slot —
  a late reminder beats no reminder.
- Receiver posts via `NotificationCompat` on a channel created with
  `IMPORTANCE_HIGH`. Boot receiver calls `rescheduleAll()` (alarms die on reboot).
- Record state in `SharedPreferences` on every arm: armed ids, exact/inexact,
  last error with `stackTraceToString()`, last test result.
- Wrap every `@JvmStatic` entry in `catch (t: Throwable)` and persist the
  trace. Never let an exception cross JNI — it arrives as an opaque wrapper.

## 3. Rust bridge (JNI)

```toml
[target.'cfg(target_os = "android")'.dependencies]
jni = "0.21"
ndk-context = "0.1"
```

One command per Kotlin entry, all `async`, all dispatched to the main thread:

```rust
#[tauri::command]
async fn schedule_reminders(app: tauri::AppHandle, plan: String) -> Result<(), String> {
    on_android_main(&app, move || { reminders::apply_plan(&plan) }).await
}
```

Where `on_android_main` uses `app.run_on_main_thread` + an `mpsc` channel with a
timeout, returning the `Result` so failures surface as strings, not panics.

JNI rules (each of these was a real production bug):

- Construct the VM with `JavaVM::from_raw(ptr)`. Never cast a pointer to `&JavaVM`.
- Resolve the class via the app context's loader, **with dots**:
  `getClass().getClassLoader().loadClass("com.example.app.ReminderScheduler")`.
  (`FindClass` wants slashes; `loadClass` wants the binary name.)
- Null-check both raw pointers before use.
- On any JNI error: `exception_occurred()` → `exception_clear()` **first**,
  then `toString()` the throwable. Calling anything else while an exception is
  pending silently no-ops.

## 4. ProGuard keep rules

Release builds minify. R8 cannot see string-based JNI calls, so it strips them
and Android throws `NoSuchMethodError` at runtime. In `proguard-rules.pro`:

```proguard
-keep class com.example.app.ReminderScheduler { *; }
-keep class com.example.app.AlarmReceiver { *; }
-keep class com.example.app.BootReceiver { *; }
```

Verify in the artifact, not by reasoning: `apkanalyzer dex code --class
com.example.app.ReminderScheduler app.apk` must list every entry point, and
`aapt2 dump permissions` must show the manifest permissions.

## 5. JS side

- Build the schedule from existing domain logic (epoch-millis slots +
  done-flags) and serialize to JSON. One function, unit-tested.
- Sync triggers: launch, resume (`visibilitychange`/`focus`, debounced),
  and every create/edit/toggle/complete.
- Permission: request `POST_NOTIFICATIONS` through the Tauri
  notification plugin once (track "asked" in `localStorage` to tell
  never-asked apart from denied). If exact alarms are denied, offer the
  Settings screen behind one explained tap — never auto-redirect.
- Keep a diagnostics panel showing armed count, next slot, timing mode, and
  the last error. Distinguish three states: no permission, bridge broken,
  armed-and-healthy. Never conflate "bridge unreachable" with "user denied".

## 6. Verify on device

1. Fresh install → grant → diagnostics show armed count, no errors.
2. Test button → immediate notification (proves posting).
3. 60s alarm → close the app **from recents** → arrives with the process dead
   (proves scheduling). This is the acceptance test.
4. Reboot → slot still fires (proves the boot receiver).

Force-stop is not the test: Android treats it as revoking background work.

## 7. Gotcha checklist

- Debug builds don't minify — a release-only failure with `NoSuchMethodError`
  is R8 until proven otherwise.
- `loadClass` = dots, `FindClass` = slashes.
- `JavaVM::from_raw`, never a reference cast.
- JNI on the main thread; Tauri commands `async`.
- Late beats never: don't drop deferred reminders with a tight cutoff.
- Swipe-away kills the process — that kills any JS-timer fallback, which is
  why the native path must own scheduling once armed.
