# Code Quality

## Detekt Setup
- Use convention plugin for consistent configuration across modules
- Start from `assets/detekt.yml.template`
- Enable Compose rules (`io.nlopez.compose.rules:detekt`)

## Key Rules
- `MaxLineLength`: 120
- `ComplexMethod`: threshold 10
- `LongParameterList`: threshold 6
- `MagicNumber`: ignore in Compose (annotate with `@Suppress`)
- Enable `ForbiddenComment` for `TODO` and `FIXME` in CI

## Privacy-Specific Rules
- Custom rule: Flag raw `Log.*` calls (use `PrivacyLogger` instead)
- Custom rule: Flag `toString()` on data classes with `@Classified(CONFIDENTIAL+)` fields
- Custom rule: Flag `SharedPreferences` usage (use `EncryptedSharedPreferences`)

## CI Integration
```yaml
- name: Run Detekt
  run: ./gradlew detekt
- name: Check for hardcoded secrets
  run: |
    if grep -rn "AIza\\|sk_live\\|-----BEGIN" --include="*.kt" --include="*.xml" app/; then
      echo "Potential secrets found!" && exit 1
    fi
```
