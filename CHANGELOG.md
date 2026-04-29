# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-04-29

### Added

- Initial release of Privacy-First Android Skill
- **SKILL.md** — Entry point with decision tree for Claude Code
- **Privacy Architecture** — Data classification framework, minimization checklists, PrivacyGuard, consent management, data retention policies, user data lifecycle, Privacy Dashboard pattern
- **Privacy Compliance** — GDPR/CCPA patterns, consent UI, data subject rights (export/delete), privacy-safe analytics, crash reporting PII scrubbing, Play Console Data Safety guidance
- **Security** — Network security config, EncryptedSharedPreferences, Android Keystore, biometric authentication, screenshot prevention, manifest hardening, Play Integrity
- **Permissions** — Just-in-time requests, rationale UI patterns, graceful degradation, Photo Picker
- **Architecture** — Clean architecture with privacy-aware data flows, ViewModel/Repository patterns
- **Modularization** — Privacy-first module structure with `core/privacy` module
- **Compose Patterns** — Screen architecture, state management, SecureScreen wrapper
- **Material 3 Theming** — Color system, typography, dynamic colors, accessibility
- **Navigation3** — Type-safe routing with privacy considerations
- **Kotlin Patterns** — Privacy-safe data classes with `Redactable` interface
- **Coroutines** — Flow patterns with consent-aware cancellation
- **Gradle Setup** — Convention plugins, version catalog, R8 log stripping
- **Testing** — Privacy-specific test patterns (PII leak detection, consent gating)
- **Accessibility** — TalkBack, WCAG compliance
- **Performance** — Play Vitals, benchmarking, privacy performance considerations
- **Debugging** — Privacy-safe logging, LeakCanary, R8 de-obfuscation
- **Crash Reporting** — Provider-agnostic with PII scrubbing
- **Code Quality** — Detekt with privacy-specific custom rules
- **Templates** — Network security config, data extraction rules, ProGuard rules
- **CI** — GitHub Actions security scanning workflow
