# 🛡️ Privacy-First Android Skill for Claude Code

<p align="center">
  <img src="assets/banner.png" alt="Privacy-First Android Skill" width="600"/>
</p>

<p align="center">
  <strong>An Agent Skill for building privacy-respecting Android apps with Kotlin & Jetpack Compose</strong>
</p>

<p align="center">
  <a href="#features">Features</a> •
  <a href="#installation">Installation</a> •
  <a href="#usage">Usage</a> •
  <a href="#philosophy">Philosophy</a> •
  <a href="#structure">Structure</a> •
  <a href="#contributing">Contributing</a>
</p>

---

## 🎯 What Is This?

This is an **Agent Skill** package for [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) that teaches Claude how to build **privacy-first Android applications** using modern best practices.

Inspired by [claude-android-ninja](https://github.com/Drjacky/claude-android-ninja), this skill adds a **privacy-by-design** layer on top of comprehensive Android development guidance — covering everything from architecture to deployment, with user privacy as the foundational principle.

> **Learn more about the Agent Skills format:** [agentskills.io](https://agentskills.io/home)

## ✨ Features

### 🔒 Privacy-First (What Makes This Unique)
- **Data Classification Framework** — Public, Internal, Confidential, Restricted tiers with enforcement rules
- **GDPR/CCPA Compliance Patterns** — Consent management, data subject rights, retention policies
- **Privacy-Safe Analytics** — Event tracking without PII leakage
- **Data Minimization Checklists** — Built into every feature creation workflow
- **Encrypted-by-Default Storage** — EncryptedSharedPreferences, field-level Room encryption, Android Keystore
- **Permission Justification Flows** — Contextual explanations before every permission request

### 📱 Modern Android Development
- Modular architecture (feature-first, core modules, strict dependencies)
- Domain/Data/UI layering with privacy-aware data flows
- Jetpack Compose patterns, state management, animation, Material 3
- Navigation3 with adaptive navigation and large-screen support
- Accessibility (TalkBack, WCAG compliance)
- Kotlin coroutines, Flow, structured concurrency
- Hilt dependency injection
- Room 3 with KSP and encrypted storage
- Retrofit + OkHttp with certificate pinning
- Gradle convention plugins and version catalogs
- Testing with fakes, Hilt testing, Turbine
- Detekt code quality with Compose rules
- Performance benchmarking (Macrobenchmark, Baseline Profiles)
- Crash reporting with PII scrubbing

### 🔐 Security
- Network security configuration (HTTPS-only, no cleartext)
- Certificate pinning (XML config + OkHttp programmatic)
- Android Keystore, TEE & StrongBox integration
- Biometric authentication with Compose
- Play Integrity API (Standard & Classic)
- Play Console Data Safety guidance
- Screenshot prevention for sensitive screens
- Secure WebView, Content Provider, and Clipboard patterns
- R8/ProGuard hardening with log stripping
- CI/CD security (secrets management, dependency scanning)

## 🏗️ Philosophy

> **"Every line of code should ask: Does this respect the user's privacy?"**

### The Five Pillars

| Pillar | Principle | Implementation |
|--------|-----------|----------------|
| 🎯 **Minimize** | Collect only what you need | Data Classification + Minimization Checklists |
| 🔐 **Encrypt** | Protect data at rest and in transit | EncryptedSharedPreferences, Keystore, TLS 1.2+ |
| 👁️ **Transparency** | Users know what you collect and why | Consent UI, Privacy Dashboard, Policy Links |
| 🎮 **Control** | Users can export, delete, and opt-out | Data Subject Rights API, Settings Screen |
| 🚨 **Fail Closed** | When in doubt, protect the user | Default deny, graceful degradation |

## 📁 Structure

```
privacy-first-android-skill/
├── SKILL.md                              # Entry point — decision tree for Claude
├── README.md                             # This file
├── LICENSE                               # MIT License
├── CHANGELOG.md                          # Version history
├── CONTRIBUTING.md                       # Contribution guidelines
│
├── references/                           # Deep-dive guides
│   ├── privacy-architecture.md           # 🆕 Privacy-by-design architecture
│   ├── privacy-compliance.md             # 🆕 GDPR/CCPA compliance patterns
│   ├── android-security.md               # Encryption, Keystore, biometrics, Play Integrity
│   ├── android-permissions.md            # Runtime permissions with privacy justification
│   ├── architecture.md                   # Clean architecture layers
│   ├── modularization.md                 # Module structure & dependency rules
│   ├── compose-patterns.md               # Jetpack Compose best practices
│   ├── android-theming.md                # Material 3 theming
│   ├── android-navigation.md             # Navigation3 patterns
│   ├── kotlin-patterns.md                # Kotlin best practices
│   ├── coroutines-patterns.md            # Coroutines & Flow
│   ├── gradle-setup.md                   # Build setup & conventions
│   ├── testing.md                        # Testing strategies
│   ├── android-accessibility.md          # Accessibility & WCAG
│   ├── android-performance.md            # Performance & Play Vitals
│   ├── android-debugging.md              # Debugging guide
│   ├── crashlytics.md                    # Crash reporting with PII scrubbing
│   └── code-quality.md                   # Detekt & code analysis
│
├── assets/                               # Templates & config files
│   ├── libs.versions.toml.template       # Version catalog
│   ├── settings.gradle.kts.template      # Project settings
│   ├── proguard-rules.pro.template       # R8/ProGuard rules
│   ├── detekt.yml.template               # Detekt configuration
│   ├── network_security_config.xml       # Network security template
│   ├── data_extraction_rules.xml         # Backup exclusion rules
│   └── convention/                       # Gradle convention plugins
│       └── QUICK_REFERENCE.md            # Setup instructions
│
└── .github/                              # GitHub config
    ├── ISSUE_TEMPLATE/
    │   ├── bug_report.md
    │   └── feature_request.md
    └── workflows/
        └── security-check.yml            # CI security scanning
```

## 🚀 Installation

### Option 1: Claude Code (Manual)

Clone or download this repo, then place it in Claude's skills folder:

```bash
# Global skills (shared across all projects)
git clone https://github.com/AbhijeetBabar-2025/privacy-first-android-skill.git \
  ~/.claude/skills/privacy-first-android-skill/

# Or project-local skills
git clone https://github.com/AbhijeetBabar-2025/privacy-first-android-skill.git \
  .claude/skills/privacy-first-android-skill/
```

Refresh skills in Claude Code after installation.

### Option 2: OpenSkills CLI

```bash
# Project-local install
npx openskills install AbhijeetBabar-2025/privacy-first-android-skill
npx openskills sync

# Global install
npx openskills install AbhijeetBabar-2025/privacy-first-android-skill --global
```

## 💡 Usage

Once installed, Claude Code automatically uses this skill when you ask about Android development. The privacy-first patterns are applied by default.

### Example Prompts

```
"Create a new Android project with user authentication"
→ Claude will set up EncryptedSharedPreferences, biometric auth,
  consent management, and privacy-safe analytics from the start.

"Add a user profile feature"
→ Claude will classify data fields, implement field-level encryption
  for PII, add data export/delete capabilities, and create a
  privacy dashboard.

"Set up crash reporting"
→ Claude will configure provider-agnostic crash reporting with
  automatic PII scrubbing, consent-aware reporting, and
  breadcrumb sanitization.
```

## 📋 Privacy Checklist for Every Release

- [ ] **Data Minimization** — Only collecting necessary data
- [ ] **Encryption** — All sensitive data encrypted at rest
- [ ] **Consent** — User consent obtained before data collection
- [ ] **Transparency** — Privacy policy current and accessible
- [ ] **Control** — Users can export and delete their data
- [ ] **Permissions** — Only necessary permissions requested with justification
- [ ] **Analytics** — No PII in analytics events or crash reports
- [ ] **Network** — HTTPS enforced, certificate pinning on critical endpoints
- [ ] **Storage** — Cloud backup excludes sensitive data
- [ ] **Play Store** — Data Safety form matches actual behavior

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Suggest a Privacy Pattern
If you have a privacy pattern or best practice to add, please [create a feature request](../../issues/new?template=feature_request.md).

### Report Issues
Found outdated guidance or a bug? Please [report it](../../issues/new?template=bug_report.md).

## 📜 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

## 🙏 Acknowledgments

- Inspired by [claude-android-ninja](https://github.com/Drjacky/claude-android-ninja) by Hossein Abbasi
- Android Developer documentation
- OWASP Mobile Security guidelines
- GDPR and CCPA regulatory frameworks

---

<p align="center">
  <strong>Built with ❤️ for user privacy</strong>
</p>
