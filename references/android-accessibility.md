# Accessibility

## Core Requirements

- Minimum 48dp × 48dp touch targets
- `contentDescription` for all icons and images
- Semantic properties for TalkBack
- Minimum 4.5:1 contrast ratio for text, 3:1 for non-text
- Support for font scaling (up to 200%)
- Keyboard navigation support

## Compose Patterns

```kotlin
Icon(
    imageVector = Icons.Default.Lock,
    contentDescription = stringResource(R.string.cd_privacy_lock)
)

Modifier.semantics {
    heading()
    stateDescription = if (isEnabled) "Enabled" else "Disabled"
}
```

## Testing

- Test with TalkBack enabled
- Run Accessibility Scanner
- Enable Espresso accessibility checks in instrumented tests
