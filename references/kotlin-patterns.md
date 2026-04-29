# Kotlin Patterns

## Must-Read Rules
- Prefer `val` over `var`; prefer immutable collections
- Use `data class` for value types, `sealed class`/`sealed interface` for state
- Use `Result<T>` for operations that can fail
- Use extension functions to keep classes focused
- Prefer composition over inheritance

## Privacy-Safe Patterns
- Never use `toString()` on objects containing PII for logging
- Use `copy()` with field masking for debug representations
- Implement `Redactable` interface for sensitive data classes

```kotlin
interface Redactable {
    fun redacted(): String
}

data class UserProfile(
    val id: String,
    val email: String,
    val name: String
) : Redactable {
    override fun redacted() = "UserProfile(id=$id, email=[REDACTED], name=[REDACTED])"
    override fun toString() = redacted() // Prevent accidental PII logging
}
```
