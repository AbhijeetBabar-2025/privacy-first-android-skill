# Android Permissions (Privacy-First)

Request only what you need, explain why, and degrade gracefully when denied.

## Core Principles
1. **Just-in-time** — request when the user triggers the feature, not at launch
2. **Explain first** — show a rationale UI before the system dialog
3. **Degrade gracefully** — the app must work (with reduced functionality) when denied
4. **Never dark-pattern** — don't nag, guilt, or block core functionality

## Permission Request Flow

```
User triggers feature
  → Check if permission granted → Use feature
  → Check shouldShowRationale → Show explanation UI → Request
  → Permission denied? → Offer alternative or guide to Settings
```

### Compose Pattern

```kotlin
@Composable
fun PermissionGatedFeature(
    permission: String,
    rationale: String,
    onGranted: @Composable () -> Unit,
    onDenied: @Composable () -> Unit
) {
    val permissionState = rememberPermissionState(permission)

    when {
        permissionState.status.isGranted -> onGranted()
        permissionState.status.shouldShowRationale -> {
            RationaleDialog(
                message = rationale,
                onConfirm = { permissionState.launchPermissionRequest() },
                onDismiss = { /* show denied UI */ }
            )
        }
        else -> onDenied()
    }
}
```

### Photo Picker (No Permission Needed)
```kotlin
val pickMedia = rememberLauncherForActivityResult(
    PickVisualMedia()
) { uri -> uri?.let { handleSelectedImage(it) } }

Button(onClick = { pickMedia.launch(PickVisualMediaRequest(ImageOnly)) }) {
    Text("Select Photo")
}
```

## Privacy-Specific Rules

- **Camera/Microphone** — explain in-app AND in Play Console Data Safety
- **Location** — use coarse location unless fine is strictly needed; explain why
- **Contacts/Calendar** — rarely justified; document extensively
- **Phone state** — almost never needed; use alternatives
- **POST_NOTIFICATIONS** (API 33+) — check before showing notifications

## Manifest Declarations
```xml
<!-- Only declare permissions you actually use -->
<uses-permission android:name="android.permission.CAMERA" />
<!-- Explain in Data Safety why this is needed -->

<!-- Use maxSdkVersion to auto-remove on newer APIs -->
<uses-permission
    android:name="android.permission.READ_EXTERNAL_STORAGE"
    android:maxSdkVersion="32" />
```

Required:
- Explain every permission to the user before requesting.
- Provide degraded functionality when permission is denied.
- Declare all permissions in Play Console Data Safety.

Forbidden:
- Requesting permissions at app launch without context.
- Blocking the entire app when a permission is denied.
- Using permissions not strictly necessary for the feature.
