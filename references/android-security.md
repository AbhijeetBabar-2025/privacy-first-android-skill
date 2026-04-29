# Android Security

For the comprehensive security reference, see [android-security.md in claude-android-ninja](https://github.com/Drjacky/claude-android-ninja/blob/master/references/android-security.md). This file provides a **privacy-focused summary** of the most critical patterns.

## Privacy-First Security Priorities

1. **Server is the trust boundary** — client never makes final authorization decisions
2. **Encrypt everything sensitive at rest** — EncryptedSharedPreferences, Keystore, field-level Room encryption
3. **HTTPS only, certificate pinning for critical endpoints**
4. **Minimal permissions, maximal encryption**
5. **Play Integrity for high-value flows** (server-decoded, `requestHash`/`nonce` binding)

## Quick Reference

### Network Security Config
```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

### EncryptedSharedPreferences
```kotlin
class SecurePreferences @Inject constructor(
    @ApplicationContext private val context: Context
) {
    private val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .setRequestStrongBoxBacked(true)
        .build()

    private val prefs = EncryptedSharedPreferences.create(
        context, "secure_prefs", masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )

    fun saveAuthToken(token: String) = prefs.edit().putString("auth_token", token).apply()
    fun getAuthToken(): String? = prefs.getString("auth_token", null)
    fun clearAll() = prefs.edit().clear().apply()
}
```

### Biometric Authentication
```kotlin
class BiometricAuthenticator {
    fun authenticate(
        activity: FragmentActivity,
        title: String, subtitle: String, negativeText: String,
        onSuccess: (BiometricPrompt.AuthenticationResult) -> Unit,
        onError: (Int, CharSequence) -> Unit,
        onFailed: () -> Unit
    ) {
        val prompt = BiometricPrompt(activity,
            ContextCompat.getMainExecutor(activity),
            object : BiometricPrompt.AuthenticationCallback() {
                override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) = onSuccess(result)
                override fun onAuthenticationError(code: Int, err: CharSequence) = onError(code, err)
                override fun onAuthenticationFailed() = onFailed()
            })
        prompt.authenticate(BiometricPrompt.PromptInfo.Builder()
            .setTitle(title).setSubtitle(subtitle).setNegativeButtonText(negativeText)
            .setAllowedAuthenticators(Authenticators.BIOMETRIC_STRONG or Authenticators.BIOMETRIC_WEAK)
            .build())
    }
}
```

### Manifest Security
```xml
<application
    android:allowBackup="false"
    android:fullBackupContent="false"
    android:dataExtractionRules="@xml/data_extraction_rules"
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="false">
```

### Screenshot Prevention
```kotlin
@Composable
fun SecureScreen(content: @Composable () -> Unit) {
    val activity = LocalContext.current as? Activity
    DisposableEffect(Unit) {
        activity?.window?.setFlags(
            WindowManager.LayoutParams.FLAG_SECURE,
            WindowManager.LayoutParams.FLAG_SECURE
        )
        onDispose { activity?.window?.clearFlags(WindowManager.LayoutParams.FLAG_SECURE) }
    }
    content()
}
```

### Security Checklist
- [ ] HTTPS enforced, certificate pinning on critical APIs
- [ ] Auth tokens in EncryptedSharedPreferences
- [ ] Sensitive DB fields encrypted
- [ ] No sensitive data in logs
- [ ] Cloud backup excludes sensitive data
- [ ] `android:allowBackup="false"`
- [ ] BiometricPrompt for sensitive actions
- [ ] R8/ProGuard enabled, log stripping in release
- [ ] `FLAG_SECURE` on sensitive screens
- [ ] Activities `android:exported="false"` except launcher
- [ ] Signing keys not in version control
- [ ] API keys not hardcoded

Required:
- Server is the trust boundary.
- Encrypt sensitive data at rest and in transit.
- Use Android Keystore over software-managed keys.

Forbidden:
- Logging tokens, PII, or sensitive payloads.
- Hardcoding API keys or secrets in source.
- Storing secrets in plain SharedPreferences.
