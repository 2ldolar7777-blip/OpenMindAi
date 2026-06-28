# 🧠 OpenMind AI Hub

**Your Universal AI Interface** — A modular, extensible Android application that provides a unified chat interface for cloud and local AI models through a flexible provider layer.

---

## ✨ Features

### 🤖 Multi-Provider AI Support
- **OpenAI-Compatible APIs** — OpenAI, Groq, Mistral, DeepSeek, and any OpenAI-compatible endpoint
- **Anthropic Claude** — Full support for Claude models with Anthropic's API format
- **Google Gemini** — Integration with Google's Gemini models
- **Ollama (Local)** — Run models locally with Ollama's NDJSON streaming
- **Mock Provider** — Built-in demo provider for testing without API keys
- **Generic HTTP** — Connect to any OpenAI-compatible endpoint with custom configuration

### 💬 Advanced Chat Experience
- **Token-by-token streaming** with animated cursor for real-time response display
- **Rich markdown rendering** — Headers, code blocks (with copy button), lists, blockquotes, inline code, bold, italic
- **Conversation management** — Pin, search, delete, and organize conversations
- **Auto-generated titles** — AI creates descriptive titles for your conversations
- **Voice input** — Speech-to-text using Android's built-in recognizer

### 🔒 Security & Privacy
- **Encrypted API key storage** — Uses EncryptedSharedPreferences with AES256-GCM encryption
- **No data collection** — All data stays on your device
- **Network security** — HTTPS enforced for cloud APIs, HTTP only for local Ollama
- **Secure backup** — API keys excluded from cloud backups

### 🎨 Modern UI
- **Material Design 3** with dynamic color support (Android 12+)
- **Jetpack Compose** — Fully declarative UI with smooth animations
- **Dark/Light/System** theme modes
- **Responsive design** — Works on phones and tablets
- **Edge-to-edge** display with splash screen

### ⚙️ Customization
- **Model parameters** — Temperature, max tokens, context length, top-p
- **Font size** — Small, Medium, Large
- **Streaming toggle** — Stream or batch responses
- **Send with Enter** — Configurable input behavior

### 📦 Data Management
- **Export/Import** — JSON and Markdown export formats
- **Search** — Full-text search across all conversations and messages
- **Token estimation** — Approximate token counting and cost calculation

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│                   UI Layer                      │
│  (Jetpack Compose + Material3 + Navigation)    │
├─────────────────────────────────────────────────┤
│               ViewModel Layer                    │
│  (ChatViewModel, ProvidersViewModel, etc.)      │
├─────────────────────────────────────────────────┤
│              Repository Layer                    │
│  (ChatRepository, ProviderRepository,           │
│   PreferencesRepository)                        │
├─────────────────────────────────────────────────┤
│                 Data Layer                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │   Room    │  │  OkHttp  │  │  DataStore   │  │
│  │   DB     │  │  +SSE    │  │  Prefs       │  │
│  └──────────┘  └──────────┘  └──────────────┘  │
├─────────────────────────────────────────────────┤
│             Provider Layer                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ Generic  │  │  Ollama  │  │  OpenAI Mock │  │
│  │  HTTP    │  │ Provider │  │   Provider   │  │
│  │Provider  │  │          │  │              │  │
│  └──────────┘  └──────────┘  └──────────────┘  │
│         Managed by ProviderManager               │
└─────────────────────────────────────────────────┘
```

### Key Design Patterns
- **MVVM** with Repository pattern
- **Dependency Injection** via Hilt
- **Observer pattern** via Kotlin Flows
- **Strategy pattern** via AIProvider interface
- **Factory pattern** via ProviderManager
- **Single source of truth** — Room DB for conversations, DataStore for preferences

---

## 📁 Project Structure

```
app/src/main/java/com/openmind/
├── OpenMindApplication.kt          # @HiltAndroidApp, lifecycle management
├── data/
│   ├── model/                      # Domain models
│   │   ├── AIResponse.kt           # Response + Part (streaming) + Usage
│   │   ├── Conversation.kt         # Conversation domain model
│   │   ├── Message.kt              # Message with Role enum
│   │   ├── ModelInfo.kt            # Model metadata
│   │   ├── ProviderConfig.kt       # Provider config with ProviderType enum
│   │   └── UserPreferences.kt      # Theme, font, model params
│   ├── db/                         # Room database
│   │   ├── OpenMindDatabase.kt      # Room DB definition
│   │   ├── ConversationEntity.kt   # Conversation table
│   │   ├── MessageEntity.kt        # Message table (FK → Conversation)
│   │   ├── ProviderConfigEntity.kt # Provider config table
│   │   ├── ConversationDao.kt      # Conversation queries
│   │   ├── MessageDao.kt           # Message queries
│   │   ├── ProviderConfigDao.kt    # Provider config queries
│   │   └── Converters.kt          # TypeConverters for Lists/Maps
│   ├── provider/                   # AI provider abstraction
│   │   ├── AIProvider.kt           # Core interface (stream, complete, models)
│   │   ├── ProviderManager.kt      # Factory + cache for providers
│   │   ├── GenericHttpProvider.kt  # OpenAI/Anthropic/Gemini/Groq/Mistral
│   │   ├── OllamaProvider.kt       # Local Ollama (NDJSON streaming)
│   │   └── OpenAIMockProvider.kt   # Demo provider with simulated streaming
│   ├── repository/                # Repository layer
│   │   ├── ChatRepository.kt      # Chat operations + streaming
│   │   ├── ProviderRepository.kt   # Provider CRUD + secure keys
│   │   └── PreferencesRepository.kt # DataStore preferences
│   ├── security/
│   │   └── SecureKeyStore.kt       # EncryptedSharedPreferences for API keys
│   ├── voice/
│   │   └── VoiceInputManager.kt    # Speech-to-text via Android SpeechRecognizer
│   ├── search/
│   │   └── SearchEngine.kt         # Full-text search across conversations
│   ├── backup/
│   │   └── OpenMindBackupAgent.kt  # Selective backup (excludes API keys)
│   └── util/
│       ├── FlowUtils.kt            # UiState wrapper, flow extensions
│       ├── DateTimeUtils.kt        # Relative time formatting
│       ├── TokenUtils.kt           # Token estimation + cost calculation
│       ├── MarkdownUtils.kt        # Structured markdown parsing
│       ├── ExportUtils.kt          # JSON/Markdown/PlainText export
│       └── NetworkUtils.kt         # Connectivity checks, URL normalization
├── di/                             # Hilt modules
│   ├── NetworkModule.kt            # OkHttp, Json, interceptors
│   ├── DatabaseModule.kt           # Room, DAOs
│   ├── ProviderModule.kt           # DataStore, repositories
│   └── SecurityModule.kt          # SecureKeyStore, VoiceInputManager
└── ui/                            # Compose UI
    ├── MainActivity.kt             # @AndroidEntryPoint, edge-to-edge
    ├── theme/
    │   ├── Theme.kt                # Material3 colors + ChatColors
    │   └── Typography.kt          # Complete typography
    ├── navigation/
    │   └── OpenMindNavigation.kt   # NavHost with animated routes
    ├── components/
    │   └── SharedComponents.kt     # Reusable composables
    ├── chat/
    │   ├── ChatScreen.kt           # Main chat UI
    │   ├── ChatItemComposable.kt   # Message rendering + markdown
    │   ├── ChatViewModel.kt        # Chat state management
    │   └── ChatUiState.kt          # Chat UI state
    ├── conversations/
    │   ├── ConversationsScreen.kt   # Conversation list
    │   └── ConversationsViewModel.kt
    ├── providers/
    │   ├── ProvidersScreen.kt       # Provider management UI
    │   └── ProvidersViewModel.kt
    └── settings/
        ├── SettingsScreen.kt       # Settings UI
        └── SettingsViewModel.kt
```

---

## 🚀 Getting Started

### Prerequisites
- **Android Studio** Hedgehog (2023.1.1) or later
- **JDK 17** 
- **Android SDK** with compileSdk 35
- **Kotlin 2.1.0** plugin support
- An AI provider API key (OpenAI, Anthropic, etc.) or [Ollama](https://ollama.ai) running locally

### Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-org/OpenMindAIHub.git
   cd OpenMindAIHub
   ```

2. **Open in Android Studio:**
   - File → Open → Select the `OpenMindAIHub` directory
   - Wait for Gradle sync to complete

3. **Configure a provider:**
   - Run the app on a device or emulator
   - Go to **Providers** tab → **Add Provider**
   - Select provider type (e.g., OpenAI Compatible)
   - Enter your base URL and API key
   - Test the connection

4. **Start chatting:**
   - Select your provider and model from the dropdowns
   - Type your message and send!

### Local Ollama Setup

1. Install Ollama: `curl -fsSL https://ollama.ai/install.sh | sh`
2. Pull a model: `ollama pull llama3.2`
3. Start the server: `ollama serve`
4. In OpenMind, add an **Ollama** provider with base URL `http://YOUR_IP:11434`

---

## 🔧 Technology Stack

| Category | Technology | Version |
|----------|-----------|---------|
| Language | Kotlin | 2.1.0 |
| UI | Jetpack Compose | BOM 2024.12.01 |
| Design | Material Design 3 | - |
| DI | Hilt | 2.53.1 |
| Database | Room | 2.6.1 |
| Network | OkHttp | 4.12.0 |
| REST | Retrofit | 2.11.0 |
| Serialization | kotlinx.serialization | 1.7.3 |
| Navigation | Navigation Compose | 2.8.5 |
| Preferences | DataStore | 1.1.1 |
| Security | EncryptedSharedPreferences | 1.1.0-alpha06 |
| Splash | SplashScreen | 1.0.1 |
| Build | AGP | 8.7.3 |
| Min SDK | Android 8.0 (API 26) | - |
| Target SDK | Android 15 (API 35) | - |

---

## 🧩 Adding a New Provider

OpenMind's extensible architecture makes it easy to add new AI providers:

### Method 1: Runtime Configuration (No Code Changes)

For any OpenAI-compatible API, simply add a new provider in the app:
1. Go to **Providers** → **Add Provider**
2. Select **Generic OpenAI-Compatible**
3. Enter the base URL, API key, and custom headers
4. The GenericHttpProvider handles all OpenAI-compatible endpoints

### Method 2: Custom Provider Implementation

1. Create a new class implementing `AIProvider`:
   ```kotlin
   class MyCustomProvider(
       private val config: ProviderConfig,
       private val client: OkHttpClient,
       private val json: Json
   ) : AIProvider {
       override val id = config.id
       override val displayName = config.name
       override val supportsStreaming = true
       
       override fun streamCompletion(
           messages: List<Message>,
           model: String,
           params: Map<String, Any>
       ): Flow<AIResponse.Part> {
           // Implement streaming logic
       }
       
       // ... implement other methods
   }
   ```

2. Register in `ProviderManager.createProvider()`:
   ```kotlin
   ProviderType.MY_CUSTOM -> MyCustomProvider(config, client, json)
   ```

3. Add the enum value to `ProviderConfig.ProviderType`

---

## 🔐 Security

- **API keys** are encrypted at rest using `EncryptedSharedPreferences` with AES256-GCM
- The master encryption key is stored in the **Android Keystore** (hardware-backed on supported devices)
- **Network security config** enforces HTTPS for all cloud API calls
- HTTP is allowed only for local Ollama connections (localhost, 192.168.x.x)
- API keys are **excluded from cloud backups** to prevent leakage
- The `SecureKeyStore.maskApiKey()` utility ensures keys are never displayed in full

---

## 📄 License

This project is licensed under the MIT License — see the LICENSE file for details.

---

## 🙏 Acknowledgments

- [Jetpack Compose](https://developer.android.com/compose) — Declarative UI toolkit
- [Material Design 3](https://m3.material.io/) — Design system
- [Hilt](https://dagger.dev/hilt/) — Dependency injection
- [OkHttp](https://square.github.io/okhttp/) — HTTP client
- [Room](https://developer.android.com/training/data-storage/room) — Database
- [Ollama](https://ollama.ai/) — Local LLM runtime
- [Kotlin](https://kotlinlang.org/) — Programming language

---

*Built with ❤️ using the highest professional tools and techniques*
