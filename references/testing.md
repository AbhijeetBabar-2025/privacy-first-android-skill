# Testing

## Philosophy
- Test behavior, not implementation
- Use fakes over mocks for repositories and data sources
- Keep test doubles in `core/testing`
- Use Turbine for testing Flow emissions

## Test Types
| Type | Location | Framework | Purpose |
|------|----------|-----------|---------|
| Unit | `test/` | JUnit 5, Truth | Business logic, ViewModels |
| Integration | `test/` | Hilt testing, Room 3 | Repository + DB |
| UI | `androidTest/` | Compose testing | Screen behavior |
| Screenshot | `screenshotTest/` | Compose Preview | Visual regression |

## Privacy-Specific Tests
- Test that PII is encrypted before storage
- Test that logs don't contain PII
- Test consent gate blocks data collection when denied
- Test data export includes all user data
- Test data deletion removes all user data
- Test permission denial flows show degraded UI

```kotlin
@Test
fun `data export includes all user data`() = runTest {
    val userId = "test-user"
    repository.saveProfile(testProfile)
    
    val export = dataExportUseCase.exportAllData(userId)
    
    assertThat(export.isSuccess).isTrue()
    assertThat(export.getOrThrow().profileData).isNotEmpty()
}

@Test
fun `analytics blocked without consent`() = runTest {
    consentManager.revokeConsent(ConsentPurpose.ANALYTICS)
    
    analytics.trackEvent("test_event")
    
    verify(analyticsProvider, never()).logEvent(any(), any())
}
```
