# Coroutines Patterns

## Core Rules

- Use `StateFlow` for observable state, `Channel` for one-shot commands
- Use appropriate dispatchers: `IO` for network/disk, `Default` for CPU, `Main` for UI
- Always handle cancellation properly
- Use `viewModelScope` in ViewModels (auto-cancelled)

## Key Patterns

- `callbackFlow` to wrap callback APIs (connectivity, sensors, location)
- `suspendCancellableCoroutine` for one-shot callbacks (Play Services, biometrics)
- `combine()` to merge multiple Flows in ViewModels
- `shareIn` to share expensive upstream collectors
- Handle backpressure with `buffer`, `conflate`, `debounce`, or `sample`

## Privacy Considerations

- Cancel location/sensor flows when consent is revoked
- Use `onCompletion` to clean up sensitive data from memory
- Don't cache decrypted sensitive data in coroutine scope beyond immediate use
