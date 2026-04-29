# Convention Plugin Quick Reference

## Setup

1. Copy all files from `assets/convention/` to `build-logic/convention/src/main/kotlin/`
2. Create `build-logic/settings.gradle.kts`:
   ```kotlin
   dependencyResolutionManagement {
       repositories {
           google()
           mavenCentral()
       }
       versionCatalogs {
           create("libs") {
               from(files("../gradle/libs.versions.toml"))
           }
       }
   }
   rootProject.name = "build-logic"
   include(":convention")
   ```
3. Add to root `settings.gradle.kts`:
   ```kotlin
   pluginManagement {
       includeBuild("build-logic")
   }
   ```
4. Add plugin entries to `gradle/libs.versions.toml`

## Available Convention Plugins

| Plugin ID | Purpose |
|-----------|---------|
| `app.android.application` | App module setup |
| `app.android.library` | Library module setup |
| `app.android.compose` | Compose configuration |
| `app.android.hilt` | Hilt DI setup |
| `app.android.room` | Room 3 + KSP setup |
| `app.android.feature` | Feature module (Compose + Hilt) |
| `app.android.detekt` | Detekt code quality |

## Usage in Module build.gradle.kts

```kotlin
plugins {
    alias(libs.plugins.app.android.feature)
}

dependencies {
    implementation(projects.core.domain)
    implementation(projects.core.privacy) // Always include privacy module
}
```
