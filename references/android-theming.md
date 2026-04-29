# Material 3 Theming

## Setup

- Use `MaterialTheme` with custom `colorScheme`, `typography`, and `shapes`
- Support light/dark themes with user preference toggle
- Enable dynamic color (Material You) on API 31+
- Use semantic color roles from `MaterialTheme.colorScheme` — never hardcode colors
- Pair every fill with its `on*` partner
- Use 8dp spacing tokens

## Color System

- Declare full M3 color set in `Color.kt` (surface containers, dim/bright, fixed variants)
- Express depth via container tone first, shadows only for floating components
- Use `outline` for interactive borders, `outlineVariant` for decorative dividers
- Harmonize brand colors against `primary` for dynamic color

## Typography

- Use `Typography` with semantic text styles
- Material Symbols icons (via Iconify API or Google Fonts)
- Prefer `bodyMedium` for body text, `titleLarge` for screen titles

## Accessibility Considerations

- Support contrast slider on Android 14+ (API 34)
- Minimum 4.5:1 contrast ratio for text
- Minimum 3:1 for non-text elements
