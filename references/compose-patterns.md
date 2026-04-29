# Compose Patterns

## Screen Architecture
```kotlin
@Composable
fun FeatureScreen(viewModel: FeatureViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    FeatureContent(uiState = uiState, onAction = viewModel::onAction)
}

@Composable
private fun FeatureContent(uiState: FeatureUiState, onAction: (FeatureAction) -> Unit) {
    // Stateless composable — testable, previewable
}
```

## State Management
- Use `StateFlow` for state, `Channel` + `receiveAsFlow()` for one-shot commands
- Survive process death with `SavedStateHandle`
- Avoid full-screen spinners — use shimmer/skeleton layouts

## Key Patterns
- **Lists**: `LazyColumn`/`LazyRow` with stable keys and `contentType`
- **Animation**: `AnimatedVisibility`, `AnimatedContent`, `animate*AsState`
- **Side Effects**: `LaunchedEffect(key)` for state-driven, `rememberCoroutineScope` for event-driven
- **Modifiers**: Order: size → padding → drawing → interaction
- Use `Modifier.Node` for custom modifiers (not deprecated `Modifier.composed`)
- **Edge-to-Edge**: Mandatory on API 36; handle system bar insets properly

## Privacy-Specific Compose Patterns
- Use `SecureScreen` wrapper for sensitive content (see [android-security.md](android-security.md))
- Gate analytics tracking behind consent in click handlers
- Never display PII in Compose previews — use fake/anonymized preview data
