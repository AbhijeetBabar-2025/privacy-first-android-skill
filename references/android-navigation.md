# Navigation3

## Setup

- Use Navigation3 for type-safe routing
- Adaptive navigation with `NavigationSuiteScaffold`
- Support large screens (tablets, foldables)

## Patterns

- Define destinations as data classes/objects with `@Serializable`
- Use `NavKey` for type-safe arguments
- Feature modules expose `Destination` + `Navigator` interfaces
- Configure navigation graph in app module

## Privacy Considerations

- Never pass PII as navigation arguments
- Clear navigation back stack on logout
- Use deep links with HTTPS App Links only (no custom schemes for sensitive flows)
