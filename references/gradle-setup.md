# Gradle Setup

## Convention Plugins

Use `build-logic/convention/` for shared build configuration. Copy templates from `assets/convention/`.

## Version Catalog

Use `gradle/libs.versions.toml` for all dependency versions. Template in `assets/libs.versions.toml.template`.

## Product Flavors & BuildConfig

- Use flavors for environment configuration (dev, staging, production)
- Use BuildConfig for compile-time constants
- **Never** put secrets in BuildConfig — use `local.properties` or CI secrets

## Build Performance

- Enable configuration cache
- Use `--parallel` and `--build-cache`
- Prefer `api` over `implementation` only when necessary
- Use lazy task configuration

## R8 / ProGuard

- Enable for release builds
- Use `assets/proguard-rules.pro.template`
- Strip logs in release
- Preserve crash report source info
- Upload mapping files to crash reporter
