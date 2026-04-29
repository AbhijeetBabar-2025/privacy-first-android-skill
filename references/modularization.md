# Modularization

## Module Structure

```
app/                          # App module (entry point, DI wiring, navigation host)
├── core/
│   ├── privacy/              # Consent, classification, retention, PrivacyGuard
│   ├── security/             # Encryption, Keystore, biometrics, Play Integrity
│   ├── domain/               # Use case interfaces, repository interfaces
│   ├── data/                 # Repository implementations, data sources
│   ├── network/              # Retrofit services, OkHttp, interceptors
│   ├── database/             # Room database, DAOs, entities, migrations
│   ├── ui/                   # Shared composables, theme, design system
│   ├── common/               # Extensions, utilities, constants
│   └── testing/              # Test doubles, fakes, test utilities
├── feature/
│   ├── auth/                 # Login, registration, password reset
│   ├── profile/              # User profile, avatar, settings
│   ├── settings/             # App settings, privacy dashboard
│   ├── onboarding/           # First-launch, consent collection
│   └── <feature-name>/       # Additional feature modules
└── build-logic/
    └── convention/           # Gradle convention plugins
```

## Dependency Rules

```
feature/* → core/domain, core/ui, core/common, core/privacy
core/data → core/domain, core/network, core/database, core/privacy
core/network → core/common
core/database → core/common
core/privacy → core/common (NO other core dependencies)
core/ui → core/common
app → feature/*, core/*
```

**Privacy module has minimal dependencies** — it should be importable by any module without circular dependencies.

## Creating a New Feature Module

1. Create `feature/<name>/` directory
2. Add `build.gradle.kts` applying convention plugin
3. Create package structure: `presentation/`, `navigation/`
4. Register in `settings.gradle.kts`
5. Add dependency in `app/build.gradle.kts`
6. **Privacy check**: Run the Data Minimization Checklist from [privacy-architecture.md](privacy-architecture.md)
