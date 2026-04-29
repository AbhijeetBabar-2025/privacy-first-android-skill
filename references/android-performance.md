# Performance

## Play Vitals Thresholds

- Crash rate: < 1.09% (bad threshold)
- ANR rate: < 0.47%
- Startup: < 1s cold start target
- Frame budget: 16.67ms (60fps) or 8.33ms (120fps)

## Performance Checklist

- [ ] Macrobenchmark for startup and scrolling
- [ ] Baseline Profiles generated and included
- [ ] R8 enabled for release builds
- [ ] Compose recomposition optimized (deferred state reads, `@Stable`/`@Immutable`)
- [ ] `LazyColumn` with stable keys and `contentType`
- [ ] Images loaded with Coil3 (proper sizing, caching)

## App Startup

- Use `App Startup` library (`Initializer` interface)
- Lazy-initialize non-critical components
- Use `installSplashScreen()` with `setKeepOnScreenCondition`
- Avoid ContentProvider-based auto-initialization

## Privacy Performance Considerations

- Encryption adds latency — cache decrypted data in memory for active sessions
- Consent checks are fast (in-memory cache from encrypted prefs)
- Data retention worker runs on charging only to avoid battery drain
