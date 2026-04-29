# Contributing to Privacy-First Android Skill

Thank you for your interest in contributing! This project aims to help developers build privacy-respecting Android applications.

## How to Contribute

### 🐛 Report a Bug
Found outdated guidance, incorrect patterns, or broken references?
[Open a bug report](../../issues/new?template=bug_report.md)

### 💡 Suggest a Feature
Have a privacy pattern or best practice to add?
[Open a feature request](../../issues/new?template=feature_request.md)

### 📝 Submit a PR

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-privacy-pattern`
3. Make your changes following the guidelines below
4. Commit with a descriptive message
5. Push and open a Pull Request

## Guidelines

### Content Standards
- **Privacy-first**: Every pattern must consider privacy implications
- **Practical**: Include working Kotlin code examples
- **Current**: Use latest stable Android/Kotlin/Compose APIs
- **Referenced**: Link to official Android documentation where applicable

### File Organization
- New reference topics go in `references/`
- New templates go in `assets/`
- Update `SKILL.md` decision tree when adding new references
- Update `README.md` structure section when adding files

### Code Examples
- Use Kotlin (no Java)
- Follow the project's Kotlin patterns (see `references/kotlin-patterns.md`)
- Include privacy annotations (`@Classified`) where relevant
- Never include real PII, tokens, or secrets in examples
- Use placeholder domains (`api.example.com`, `yourdomain.com`)

### Commit Messages
```
feat: add privacy-safe push notification patterns
fix: update Room 3 encryption to use SQLiteDriver
docs: clarify GDPR consent withdrawal flow
```

## Code of Conduct

Be respectful, constructive, and focused on building privacy-respecting software. We're all here to make Android apps better for users.
