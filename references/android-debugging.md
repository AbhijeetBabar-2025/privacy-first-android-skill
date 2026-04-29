# Debugging

## Logcat Levels

| Level | Use For | Release Build |
|-------|---------|---------------|
| `v` | Verbose trace | Stripped by R8 |
| `d` | Debug info | Stripped by R8 |
| `i` | Informational | Stripped by R8 |
| `w` | Warnings | Keep |
| `e` | Errors | Keep |

## ANR Debugging

- Main thread blocked > 5 seconds
- `InputDispatching` timeout: 5s
- `BroadcastReceiver` timeout: 10s (foreground), 60s (background)
- Use `adb bugreport` and check traces.txt

## Common Gradle Errors

- `Could not resolve` → Check version catalog and repositories
- `Duplicate class` → Check dependency tree with `./gradlew dependencies`
- `KSP error` → Check annotation processor configuration

## LeakCanary

- Add to debug dependencies only
- Monitor Activity and Fragment leaks
- Check for ViewModel scope leaks with encrypted data

## R8 De-obfuscation

- Upload mapping.txt to Firebase/Sentry
- Manual: `retrace -verbose mapping.txt stacktrace.txt`

## Privacy Debugging

- Use `PrivacyLogger` wrapper instead of raw `Log.*`
- Check that release builds have log stripping enabled
- Verify encrypted storage with `adb shell` (data should be unreadable)
