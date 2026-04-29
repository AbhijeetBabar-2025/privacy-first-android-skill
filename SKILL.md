---
name: privacy-first-android-skill
description: Privacy-First Android Development Skill for Claude Code. Build production-quality Android apps with Kotlin, Jetpack Compose, MVVM, Hilt, Room 3, and multi-module architecture — with user privacy as the foundational design principle. Triggers on requests to create Android projects, modules, screens, ViewModels, or repositories.
license: MIT
metadata:
  author: AbhijeetBabar-2025
  version: 1.0.0
  documentation: https://github.com/AbhijeetBabar-2025/privacy-first-android-skill
  tags: [android, kotlin, compose, privacy, security, mvvm, hilt, room, datastore, gradle, mobile]
---

Use when building Android apps with Kotlin, Jetpack Compose, MVVM, Hilt, Room 3, DataStore, Paging 3, or multi-module projects — **especially** when user privacy, data protection, or GDPR/CCPA compliance is a concern.

Triggers on requests to create Android projects, screens, ViewModels, repositories, feature modules, privacy-compliant data flows, or asks about Android architecture patterns with a privacy-first mindset.

## 🛡️ Core Philosophy: Privacy by Design

**Every decision — from architecture to UI — must answer: "Does this respect the user's privacy?"**

1. **Collect only what you need** — minimize data at every layer
2. **Encrypt everything at rest** — tokens, PII, database fields
3. **Be transparent** — users must know what data you collect and why
4. **Give users control** — export, delete, and opt-out capabilities
5. **Fail closed** — when in doubt, protect the user

## Decision Tree

| Task | Reference File |
|------|---------------|
| Privacy-first architecture & data minimization | [privacy-architecture.md](references/privacy-architecture.md) |
| Privacy consent & GDPR/CCPA compliance | [privacy-compliance.md](references/privacy-compliance.md) |
| Data encryption & secure storage | [android-security.md](references/android-security.md) |
| Network security & certificate pinning | [android-security.md](references/android-security.md) |
| Permission handling & justification | [android-permissions.md](references/android-permissions.md) |
| Project structure & modules | [modularization.md](references/modularization.md) |
| Architecture layers (Domain, Data, UI, Common) | [architecture.md](references/architecture.md) |
| Compose patterns, Material motion, animation | [compose-patterns.md](references/compose-patterns.md) |
| Material 3 theming, spacing tokens, dynamic colors | [android-theming.md](references/android-theming.md) |
| Navigation3, adaptive navigation | [android-navigation.md](references/android-navigation.md) |
| Kotlin patterns, View lifecycle interop | [kotlin-patterns.md](references/kotlin-patterns.md) |
| Coroutine patterns | [coroutines-patterns.md](references/coroutines-patterns.md) |
| Gradle, product flavors, BuildConfig | [gradle-setup.md](references/gradle-setup.md) |
| Testing approach | [testing.md](references/testing.md) |
| Accessibility, TalkBack, WCAG | [android-accessibility.md](references/android-accessibility.md) |
| Performance, benchmarking, Play Vitals | [android-performance.md](references/android-performance.md) |
| Debugging, Logcat, ANR, R8, memory leaks | [android-debugging.md](references/android-debugging.md) |
| Crash reporting (provider-agnostic) | [crashlytics.md](references/crashlytics.md) |
| Code quality (Detekt) | [code-quality.md](references/code-quality.md) |

---

**Creating a new project?**
→ Start with [privacy-architecture.md](references/privacy-architecture.md) for privacy-first module design
→ Use `assets/settings.gradle.kts.template` for settings and module includes
→ Use `assets/libs.versions.toml.template` for the version catalog
→ Copy convention plugins from `assets/convention/` to `build-logic/`
→ Read [modularization.md](references/modularization.md) for structure and module types
→ Configure `.gitignore` with privacy-sensitive exclusions from [android-security.md](references/android-security.md)

**Building any screen or feature?**
→ **FIRST** check [privacy-architecture.md](references/privacy-architecture.md) → "Data Minimization Checklist"
→ Read [compose-patterns.md](references/compose-patterns.md) for screen architecture
→ Ensure no PII leaks through logs, crash reports, or analytics
→ Use [android-theming.md](references/android-theming.md) for Material 3

**Handling user data?**
→ Follow [privacy-architecture.md](references/privacy-architecture.md) → "Data Classification"
→ Encrypt at rest using [android-security.md](references/android-security.md)
→ Implement data retention policies
→ Provide export/delete capabilities per [privacy-compliance.md](references/privacy-compliance.md)

**Adding analytics or crash reporting?**
→ Follow [privacy-compliance.md](references/privacy-compliance.md) → "Privacy-Safe Analytics"
→ Use [crashlytics.md](references/crashlytics.md) with PII scrubbing
→ **Never** log PII, tokens, or user-identifiable data

**Requesting permissions?**
→ Follow [android-permissions.md](references/android-permissions.md) for just-in-time requests
→ Always explain **why** before requesting
→ Provide degraded-but-functional experience when denied
→ Track permissions in Play Console Data Safety

**Implementing authentication?**
→ Use [android-security.md](references/android-security.md) → Credential Manager & Biometrics
→ Store tokens in EncryptedSharedPreferences
→ Implement session timeout and re-authentication

**Setting up network calls?**
→ Follow [android-security.md](references/android-security.md) → Network Security
→ HTTPS only, certificate pinning for critical endpoints
→ No PII in URL parameters — use request body
→ Implement request signing for sensitive operations

**Preparing for Play Store submission?**
→ Complete [privacy-compliance.md](references/privacy-compliance.md) → "Play Console Data Safety Checklist"
→ Verify privacy policy URL is current
→ Ensure data deletion path exists
→ Review all SDK data collection declarations

**Writing any Kotlin code?**
→ **Always** follow [kotlin-patterns.md](references/kotlin-patterns.md)
→ Align with [architecture.md](references/architecture.md) and [modularization.md](references/modularization.md)
→ Check for privacy implications in data flows

**Testing?**
→ Read [testing.md](references/testing.md) for testing philosophy and patterns
→ Include privacy-specific test cases (PII leak detection, encryption verification)
→ Test permission denial flows thoroughly
