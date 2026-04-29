# Architecture

Clean architecture with privacy-aware data flows. See [privacy-architecture.md](privacy-architecture.md) for the privacy-specific layer on top.

## Layers

```
UI (Compose) → ViewModel → UseCase → Repository (interface) → DataSource (Room/API)
```

### Rules

- UI depends on ViewModel only
- ViewModel depends on UseCases/Repository interfaces (domain)
- Data layer implements domain interfaces
- **Privacy layer** (consent, classification) wraps data access

### ViewModel Pattern

```kotlin
@HiltViewModel
class ProfileViewModel @Inject constructor(
    private val getUserProfile: GetUserProfileUseCase,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    private val _uiState = MutableStateFlow<ProfileUiState>(ProfileUiState.Loading)
    val uiState: StateFlow<ProfileUiState> = _uiState.asStateFlow()

    init { loadProfile() }

    private fun loadProfile() {
        viewModelScope.launch {
            getUserProfile().fold(
                onSuccess = { _uiState.value = ProfileUiState.Success(it) },
                onFailure = { _uiState.value = ProfileUiState.Error(it.message ?: "") }
            )
        }
    }
}
```

### Repository Pattern (Privacy-Aware)

```kotlin
// Domain layer — interface
interface UserRepository {
    suspend fun getUserProfile(userId: String): Result<UserProfile>
    suspend fun updateProfile(profile: UserProfile): Result<Unit>
    suspend fun deleteProfile(userId: String): Result<Unit>
    suspend fun exportProfileData(userId: String): Result<UserDataExport>
}

// Data layer — implementation with encryption
class UserRepositoryImpl @Inject constructor(
    private val userDao: UserDao,
    private val api: UserApi,
    private val encryption: AesGcmEncryption,
    private val consentManager: ConsentManager
) : UserRepository {
    override suspend fun getUserProfile(userId: String): Result<UserProfile> {
        // Decrypt stored data, return domain model
    }
}
```

### Network Layer

```kotlin
@Module @InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides @Singleton
    fun provideOkHttpClient(): OkHttpClient = OkHttpClient.Builder()
        .connectTimeout(30, TimeUnit.SECONDS)
        .connectionSpecs(listOf(ConnectionSpec.MODERN_TLS))
        .build()
}
```

### Cross-cutting Anti-patterns

- ❌ Context in ViewModel
- ❌ Business logic in UI layer
- ❌ Direct database access from ViewModel
- ❌ API calls without error handling
- ❌ Unencrypted PII in data layer
