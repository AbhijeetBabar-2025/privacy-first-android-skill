# Privacy Compliance (GDPR / CCPA / Play Store)

Patterns for building Android apps that comply with GDPR, CCPA,
and Google Play policies. All patterns assume the `core/privacy`
module from [privacy-architecture.md](privacy-architecture.md).

## Table of Contents

1. [Consent Management](#consent-management)
2. [Data Subject Rights](#data-subject-rights)
3. [Privacy-Safe Analytics](#privacy-safe-analytics)
4. [Crash Reporting with PII Scrubbing](#crash-reporting-with-pii-scrubbing)
5. [Play Console Data Safety](#play-console-data-safety)
6. [Third-Party SDK Audit](#third-party-sdk-audit)
7. [Privacy Policy Integration](#privacy-policy-integration)

## Consent Management

### Consent UI Flow

Show consent **before** any data collection begins. Never
pre-check consent boxes.

```kotlin
@Composable
fun ConsentScreen(
    onConsentComplete: () -> Unit,
    viewModel: ConsentViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    Column(modifier = Modifier.padding(24.dp)) {
        Text(
            "Your Privacy Matters",
            style = MaterialTheme.typography.headlineMedium
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text("We respect your privacy. Please choose what data we " +
            "can use:")

        ConsentToggle(
            title = "Analytics",
            description = "Help us improve the app with anonymous " +
                "usage data",
            checked = uiState.analyticsConsent,
            onCheckedChange = {
                viewModel.setConsent(ConsentPurpose.ANALYTICS, it)
            }
        )
        ConsentToggle(
            title = "Crash Reporting",
            description = "Send crash reports to help us fix bugs " +
                "faster",
            checked = uiState.crashConsent,
            onCheckedChange = {
                viewModel.setConsent(ConsentPurpose.CRASH_REPORTING, it)
            }
        )

        Spacer(modifier = Modifier.weight(1f))

        Button(
            onClick = {
                viewModel.saveConsents()
                onConsentComplete()
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Continue")
        }

        TextButton(
            onClick = {
                viewModel.denyAll()
                onConsentComplete()
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Skip — Use app with minimal data")
        }
    }
}
```

### Consent Storage

```kotlin
class ConsentManagerImpl @Inject constructor(
    private val securePrefs: SecurePreferences
) : ConsentManager {
    override suspend fun hasConsent(
        purpose: ConsentPurpose
    ): Boolean {
        return securePrefs.getBoolean(
            "consent_${purpose.name}", false
        )
    }

    override suspend fun requestConsent(
        purpose: ConsentPurpose
    ): ConsentResult {
        return if (hasConsent(purpose))
            ConsentResult.Granted else ConsentResult.Denied
    }

    override suspend fun revokeConsent(
        purpose: ConsentPurpose
    ) {
        securePrefs.putBoolean("consent_${purpose.name}", false)
        securePrefs.remove("consent_${purpose.name}_timestamp")
        // Trigger data cleanup for this purpose
    }

    override fun getActiveConsents():
        Flow<List<ConsentRecord>> = flow {
        val consents = ConsentPurpose.entries.map { purpose ->
            ConsentRecord(
                purpose = purpose,
                granted = hasConsent(purpose),
                grantedAt = securePrefs
                    .getLong("consent_${purpose.name}_timestamp", 0L)
                    .takeIf { it > 0 }
                    ?.let { Instant.fromEpochMilliseconds(it) },
                expiresAt = null
            )
        }
        emit(consents)
    }
}
```

## Data Subject Rights

### Right to Access (Data Export)

```kotlin
class DataExportUseCase @Inject constructor(
    private val userRepo: UserRepository,
    private val activityRepo: ActivityRepository,
    private val encryption: AesGcmEncryption
) {
    suspend fun exportAllData(userId: String): Result<File> {
        val profile = userRepo.getUserProfile(userId)
        val activities = activityRepo.getUserActivities(userId)

        val exportData = buildJsonObject {
            put("exported_at", Clock.System.now().toString())
            put("format_version", "1.0")
            putJsonObject("profile") {
                put("name", profile?.displayName ?: "")
                put("email", profile?.email ?: "")
            }
            putJsonArray("activities") {
                activities.forEach { activity ->
                    addJsonObject {
                        put("type", activity.type)
                        put(
                            "timestamp",
                            activity.timestamp.toString()
                        )
                    }
                }
            }
        }
        // Write to encrypted temp file and return
        return Result.success(writeToEncryptedFile(exportData))
    }
}
```

### Right to Erasure (Data Deletion)

```kotlin
class DataDeletionUseCase @Inject constructor(
    private val userDao: UserDao,
    private val analyticsDao: AnalyticsDao,
    private val securePrefs: SecurePreferences,
    private val consentManager: ConsentManager
) {
    suspend fun deleteAllUserData(userId: String): Result<Unit> {
        return try {
            // 1. Delete from all databases
            userDao.deleteUser(userId)
            analyticsDao.deleteByUser(userId)

            // 2. Clear encrypted preferences
            securePrefs.clearAll()

            // 3. Revoke all consents
            ConsentPurpose.entries.forEach {
                consentManager.revokeConsent(it)
            }

            // 4. Notify server of deletion request
            // api.requestAccountDeletion(userId)

            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

## Privacy-Safe Analytics

### Rules

- **Never** log PII (email, name, phone, location coordinates)
- **Never** log user IDs that can be tied to real identities
- **Always** check consent before sending events
- **Always** sanitize event parameters
- Use anonymous session IDs (regenerated periodically)

### Implementation

```kotlin
interface AnalyticsProvider {
    fun logEvent(name: String, params: Map<String, Any>)
    fun setEnabled(enabled: Boolean)
}

class FirebaseAnalyticsProvider @Inject constructor(
    private val analytics: FirebaseAnalytics
) : AnalyticsProvider {
    override fun logEvent(name: String, params: Map<String, Any>) {
        val bundle = Bundle().apply {
            params.forEach { (key, value) ->
                when (value) {
                    is String -> putString(key, value)
                    is Int -> putInt(key, value)
                    is Long -> putLong(key, value)
                    is Boolean -> putBoolean(key, value)
                }
            }
        }
        analytics.logEvent(name, bundle)
    }

    override fun setEnabled(enabled: Boolean) {
        analytics.setAnalyticsCollectionEnabled(enabled)
    }
}
```

## Crash Reporting with PII Scrubbing

```kotlin
// Wrap crash reporter to scrub PII
class PrivacySafeCrashReporter @Inject constructor(
    private val delegate: CrashReporter,
    private val consentManager: ConsentManager
) : CrashReporter {
    override fun recordException(throwable: Throwable) {
        if (runBlocking {
                consentManager.hasConsent(
                    ConsentPurpose.CRASH_REPORTING
                )
            }) {
            delegate.recordException(throwable.scrubPii())
        }
    }

    override fun log(message: String) {
        if (runBlocking {
                consentManager.hasConsent(
                    ConsentPurpose.CRASH_REPORTING
                )
            }) {
            delegate.log(scrubPii(message))
        }
    }

    private fun scrubPii(text: String): String = text
        .replace(
            Regex("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"),
            "[EMAIL]"
        )
        .replace(Regex("Bearer\\s+\\S+"), "Bearer [REDACTED]")
        .replace(Regex("\\b\\d{10,}\\b"), "[REDACTED_NUM]")

    private fun Throwable.scrubPii(): Throwable =
        Exception(scrubPii(message ?: "unknown"), cause?.scrubPii())
}
```

## Play Console Data Safety

### Checklist Before Submission

- [ ] List **every** data type collected (per SDK and your code)
- [ ] Mark data as "Required" or "Optional" accurately
- [ ] Declare all **sharing** with third parties
- [ ] Declare if data is **encrypted in transit** (should be yes)
- [ ] Declare if data is **encrypted at rest** (should be yes for
  PII)
- [ ] Declare if user can **request deletion** (should be yes)
- [ ] Privacy policy URL is current and accessible
- [ ] All declared practices match actual app behavior

### Common SDK Data Collection

| SDK | Data Collected | Declare In Data Safety |
|-----|---------------|----------------------|
| Firebase Analytics | App interactions, device info | Yes — Analytics |
| Firebase Crashlytics | Crash logs, device state | Yes — App diagnostics |
| Google Ads SDK | Advertising ID | Yes — Advertising |
| Firebase Auth | Email, phone (if used) | Yes — Personal info |
| Firebase Cloud Messaging | FCM token | Yes — App functionality |

## Third-Party SDK Audit

Before adding any SDK:

1. **Review privacy policy** of the SDK provider
2. **Check data collection** — what does it send and to where?
3. **Verify compliance** — is the SDK GDPR/CCPA compliant?
4. **Check opt-out support** — can data collection be disabled?
5. **Document** the SDK in your Data Safety form

```kotlin
// Example: Disable SDK collection when consent is revoked
class SdkPrivacyManager @Inject constructor(
    private val consentManager: ConsentManager
) {
    suspend fun syncSdkSettings() {
        val analyticsConsent =
            consentManager.hasConsent(ConsentPurpose.ANALYTICS)
        FirebaseAnalytics.getInstance(context)
            .setAnalyticsCollectionEnabled(analyticsConsent)

        val crashConsent =
            consentManager.hasConsent(
                ConsentPurpose.CRASH_REPORTING
            )
        FirebaseCrashlytics.getInstance()
            .setCrashlyticsCollectionEnabled(crashConsent)
    }
}
```

## Privacy Policy Integration

### In-App Access (Required)

```kotlin
@Composable
fun PrivacyPolicyLink(url: String) {
    val uriHandler = LocalUriHandler.current
    TextButton(onClick = { uriHandler.openUri(url) }) {
        Icon(Icons.Default.Policy, contentDescription = null)
        Spacer(Modifier.width(8.dp))
        Text(
            "Privacy Policy",
            style = MaterialTheme.typography.bodyMedium
        )
    }
}
```

### Required Locations

- Settings screen
- Registration/consent screen
- Play Store listing
- First launch onboarding

Required:

- Consent must be obtained before any non-essential data
  collection.
- Users must be able to export and delete their data.
- Crash reports must have PII scrubbed before sending.
- Play Console Data Safety must accurately reflect actual
  behavior.
- Every third-party SDK must be audited for privacy compliance.

Forbidden:

- Pre-checked consent boxes.
- Collecting data before consent is given.
- Sending crash reports without crash reporting consent.
- Misrepresenting data collection in Play Console Data Safety.
