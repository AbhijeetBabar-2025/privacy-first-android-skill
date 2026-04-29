# Crash Reporting

## Provider-Agnostic Interface
```kotlin
interface CrashReporter {
    fun recordException(throwable: Throwable)
    fun log(message: String)
    fun setUserId(id: String)
    fun setCustomKey(key: String, value: String)
}
```

## Privacy-Safe Implementation
- **Always** scrub PII before sending crash reports
- Gate crash reporting behind `ConsentPurpose.CRASH_REPORTING`
- Use anonymized user IDs (never email or phone)
- Don't send breadcrumbs containing sensitive navigation args

## DI Binding
```kotlin
@Module @InstallIn(SingletonComponent::class)
abstract class CrashReportingModule {
    @Binds @Singleton
    abstract fun bindCrashReporter(impl: PrivacySafeCrashReporter): CrashReporter
}
```

## Scrubbing Rules
- Remove emails: `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` → `[EMAIL]`
- Remove tokens: `Bearer\s+\S+` → `Bearer [REDACTED]`
- Remove phone numbers: `\b\d{10,}\b` → `[REDACTED_NUM]`
- Remove file paths containing usernames
