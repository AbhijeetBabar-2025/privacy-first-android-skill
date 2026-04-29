# Privacy-First Architecture

This guide defines how to design Android app architecture with **user privacy as the
foundational constraint**. Every architectural decision must pass the privacy test:
_"Does this minimize data exposure and user surveillance?"_

## Table of Contents

1. [Data Classification Framework](#data-classification-framework)
2. [Data Minimization Checklist](#data-minimization-checklist)
3. [Privacy-Aware Module Design](#privacy-aware-module-design)
4. [Privacy Data Flow Rules](#privacy-data-flow-rules)
5. [Encrypted Storage Patterns](#encrypted-storage-patterns)
6. [Privacy-Safe Logging](#privacy-safe-logging)
7. [Data Retention Policies](#data-retention-policies)
8. [User Data Lifecycle](#user-data-lifecycle)
9. [Privacy Dashboard Pattern](#privacy-dashboard-pattern)
10. [Anti-Patterns](#anti-patterns)

## Data Classification Framework

Classify **every** piece of data your app handles.

| Tier | Examples | Storage | Logging | Retention | User Control |
|------|----------|---------|---------|-----------|-------------|
| **Public** | App version, device model | Plain SharedPreferences | Allowed | Indefinite | N/A |
| **Internal** | Feature flags, UI prefs | DataStore | Key only | Until uninstall | Settings |
| **Confidential** | Email, name, photo URL | EncryptedSharedPreferences | **Never** | Per policy | Export + Delete |
| **Restricted** | Auth tokens, SSN, payment | EncryptedSharedPreferences + Keystore | **Never** | Minimum necessary | Export + Delete + Re-auth |

### Applying Classification in Code

```kotlin
enum class DataClassification {
    PUBLIC, INTERNAL, CONFIDENTIAL, RESTRICTED
}

@Target(AnnotationTarget.PROPERTY, AnnotationTarget.FIELD)
@Retention(AnnotationRetention.SOURCE)
annotation class Classified(val level: DataClassification)
```

## Data Minimization Checklist

**Run for every feature before writing code:**

### Before Collecting

- [ ] Is this data **strictly necessary** for the feature?
- [ ] Can the feature work with **less specific** data?
- [ ] Can it work with **derived/aggregated** data instead of raw?
- [ ] Is there a **privacy-preserving alternative** (e.g., on-device ML)?
- [ ] Have you documented **why** this data is needed?

### Before Storing

- [ ] What is the **minimum retention period**?
- [ ] Is storage encrypted per Data Classification?
- [ ] Is the data excluded from cloud backups?
- [ ] Can the data be **derived on-demand** instead?

### Before Transmitting

- [ ] Is the connection HTTPS with certificate pinning?
- [ ] Sending only **minimum fields** the API requires?
- [ ] Is PII in the request **body** (not URL parameters)?

### Before Logging/Analytics

- [ ] Does the log/event contain **zero PII**?
- [ ] Are IDs anonymized or hashed?
- [ ] Has the user consented to analytics?

## Privacy-Aware Module Design

```
app/
├── core/
│   ├── privacy/       # Consent, classification, retention
│   ├── security/      # Encryption, Keystore, biometrics
│   ├── data/          # Repositories, data sources
│   ├── domain/        # Use cases, repository interfaces
│   ├── network/       # API services, interceptors
│   ├── database/      # Room, DAOs, encrypted entities
│   ├── ui/            # Shared composables, theme
│   └── testing/       # Test utilities, fakes
├── feature/
│   ├── auth/          # Login, registration (Restricted)
│   ├── profile/       # User profile (Confidential)
│   ├── settings/      # Privacy settings, consent
│   └── onboarding/    # Privacy-first onboarding
```

### The `core/privacy` Module

```kotlin
// ConsentManager.kt
interface ConsentManager {
    suspend fun hasConsent(purpose: ConsentPurpose): Boolean
    suspend fun requestConsent(purpose: ConsentPurpose): ConsentResult
    suspend fun revokeConsent(purpose: ConsentPurpose)
    fun getActiveConsents(): Flow<List<ConsentRecord>>
}

enum class ConsentPurpose {
    ANALYTICS, CRASH_REPORTING, PERSONALIZATION, MARKETING,
    THIRD_PARTY_SHARING
}

// DataRetentionManager.kt
interface DataRetentionManager {
    suspend fun enforceRetentionPolicies()
    suspend fun deleteAllUserData(): Result<Unit>
    suspend fun exportUserData(): Result<UserDataExport>
}

// PrivacyGuard.kt — central enforcement point
class PrivacyGuard @Inject constructor(
    private val consentManager: ConsentManager
) {
    suspend fun <T> withConsent(
        purpose: ConsentPurpose, fallback: T, operation: suspend () -> T
    ): T = if (consentManager.hasConsent(purpose)) operation()
        else fallback

    fun sanitize(input: String): String = input
        .replace(
            Regex("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"),
            "[EMAIL]"
        )
        .replace(Regex("\\b\\d{3}[-.]?\\d{3}[-.]?\\d{4}\\b"), "[PHONE]")
        .replace(Regex("\\b\\d{3}-\\d{2}-\\d{4}\\b"), "[SSN]")
        .replace(
            Regex("\\b\\d{4}[- ]?\\d{4}[- ]?\\d{4}[- ]?\\d{4}\\b"),
            "[CARD]"
        )
}
```

## Privacy Data Flow Rules

### Inbound Data (Collection)

```
User Input → Consent Check → Classify → Encrypt (if needed) → Store
```

### Outbound Data (API Calls)

```
Read → Decrypt → Transform (minimize fields) → HTTPS + Pinning → Server
```

- **Never** send more fields than the API requires
- **Never** put PII in URL query parameters
- Implement request signing for Restricted tier

### Analytics Data

```
Event → Consent Check → PII Scrub → Anonymize IDs → Send (or Drop)
```

```kotlin
class PrivacySafeAnalytics @Inject constructor(
    private val analyticsProvider: AnalyticsProvider,
    private val privacyGuard: PrivacyGuard
) {
    suspend fun trackEvent(
        name: String,
        params: Map<String, Any> = emptyMap()
    ) {
        privacyGuard.withConsent(ConsentPurpose.ANALYTICS, Unit) {
            val safeParams = params.mapValues { (_, v) ->
                if (v is String) privacyGuard.sanitize(v) else v
            }
            analyticsProvider.logEvent(name, safeParams)
        }
    }
}
```

## Encrypted Storage Patterns

| Data Type | Storage |
|-----------|---------|
| Small secrets (tokens, keys) | EncryptedSharedPreferences |
| Structured relational data | Room 3 + field-level encryption |
| Files (documents, images) | EncryptedFile |
| Simple preferences (Internal) | DataStore (no encryption) |
| Simple preferences (Confidential+) | EncryptedSharedPreferences |

## Privacy-Safe Logging

```kotlin
// ❌ NEVER
Log.d(TAG, "User logged in: email=$email, token=$token")

// ✅ ALWAYS
Log.d(TAG, "User logged in successfully")
```

### R8 Log Stripping

```proguard
-assumenosideeffects class android.util.Log {
    public static int v(...);
    public static int d(...);
    public static int i(...);
}
```

## Data Retention Policies

```kotlin
@HiltWorker
class DataRetentionWorker @AssistedInject constructor(
    @Assisted context: Context,
    @Assisted params: WorkerParameters,
    private val userDao: UserDao
) : CoroutineWorker(context, params) {
    private val policies = listOf(
        RetentionPolicy("analytics_events", 90, ExpiryAction.DELETE),
        RetentionPolicy("search_history", 30, ExpiryAction.DELETE),
        RetentionPolicy("cached_profiles", 7, ExpiryAction.DELETE)
    )

    override suspend fun doWork(): Result {
        policies.forEach { policy ->
            val cutoff = System.currentTimeMillis() -
                (policy.maxAgeDays * 86_400_000L)
            // Execute cleanup per policy.dataType
        }
        return Result.success()
    }
}
```

## User Data Lifecycle

```
Create Account → Consent → Encrypt & Store → Active Usage
   ↓                                              ↓
Retention Cleanup (periodic)              User Requests Export/Delete
   ↓                                              ↓
Auto-purge expired data                  Anonymize → Hard Delete (30 days)
```

## Privacy Dashboard Pattern

Every app should include a Privacy Dashboard in Settings:

- Active consent preferences (toggles)
- Data categories stored (with encryption status)
- "Export My Data" action
- "Delete My Data" action (destructive, requires confirmation)
- Privacy Policy link

## Anti-Patterns

### ❌ Never

- Collect data "just in case"
- Log PII at any level
- Store tokens in plain SharedPreferences
- Send PII in URL parameters
- Track analytics without consent
- Use `android:allowBackup="true"` with sensitive data
- Use hardware IDs (IMEI, MAC) for tracking

### ✅ Always

- Collect only what's needed
- Use PrivacyLogger wrapper
- Use EncryptedSharedPreferences
- Send PII in HTTPS request body
- Gate analytics behind consent
- Exclude sensitive data from backups
- Use account-based or random UUID identity
- Provide graceful degradation when permissions denied
