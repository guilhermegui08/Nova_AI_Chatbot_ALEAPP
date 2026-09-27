# Nova AI Chatbot — ALEAPP Modules

[![ALEAPP](https://img.shields.io/badge/ALEAPP-Merged-success)](https://github.com/abrignoni/ALEAPP)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Forensic analysis modules for the Android application **AI Chatbot – Nova** (`com.scaleup.chatai`), developed for [ALEAPP](https://github.com/abrignoni/ALEAPP) (Android Logs Events And Protobuf Parser).

These four modules were **submitted and are currently merged into the official ALEAPP repository**, automating the extraction, parsing, and reconstruction of forensic artifacts from the Nova AI Chatbot application.

---

## 📱 Target Application

| Characteristic | Value |
|---|---|
| **App Name** | AI Chatbot – Nova |
| **Package** | `com.scaleup.chatai` |
| **Version Analyzed** | 4.0.13 |
| **Downloads** | 100,000,000+ |
| **Publisher** | ScaleUp Yazılım Hizmetleri A.S. (HubX) |
| **Supported Models** | ChatGPT (GPT-4o, GPT-4o Mini, GPT-5.1), Gemini, DeepSeek, Claude, Grok, Mistral, Llama2 |

---

## 🧩 Modules Overview

The following modules were developed to parse artifacts from a rooted Android extraction of the Nova AI Chatbot app:

| Module | Source Artifacts | Reports Generated |
|---|---|---|
| **`AIChatbotNovaHistory`** | `chat-ai.db` (main SQLite database) | 5 distinct reports covering conversation history, messages, models used, timestamps, token counts, and attached documents |
| **`AIChatbotNovaSharedPrefs`** | `shared_prefs/` (`MOMO_PREF_FILE.xml`, `AdaptySDKPrefs.xml`) | 2 reports covering user authentication (JWT token), usage metrics, and payment/subscription data (Adapty SDK) |
| **`AIChatbotNovaMediastore`** | `chat-ai.db`, `external.db`, `sdcard/Android/media/com.scaleup.chatai/Nova/` | 1 report correlating files submitted by the user, files present in the app's media directory, and files indexed in the Android MediaStore |
| **`AIChatbotNovaConversations`** | `chat-ai.db`, `external.db` | 1 unified report reconstructing full conversation history with file paths recovered via MediaStore |

---

## 🔍 What Each Module Extracts

### `AIChatbotNovaHistory`
Parses the main application database (`chat-ai.db`) which contains the core forensic artifacts:

- **`History`** — All conversation sessions: UUID, title, creation/last modification timestamps, chatbot model used (`chatBotModel`), and assistant ID (`assistantID`) for personality-based chats (e.g., Math Teacher, SuperBot).
- **`HistoryDetail`** — Full message content (user prompts + chatbot responses), timestamps, token count per iteration, sync state, and message author (`type`: user or chatbot).
- **`HistoryDetailDocument`** — Attached files and documents, including the original filename and the direct Firebase Cloud Storage URL.
- Recovers deleted conversations despite the anti-forensic `softDeleted` field behavior.

### `AIChatbotNovaSharedPrefs`
Parses the application's XML shared preferences:

- **`MOMO_PREF_FILE.xml`** — User JWT token (decodable via JWT tools) containing email, username, Firebase UID, and Google `sign_in_provider`; Firebase tokens (`KEY_FCM_TOKEN`, `KEY_USER_AUTHENTICATION_ID`); and usage metrics (`KEY_SUCCESSFULL_CHAT_RESPONSE`, `KEY_SESSION_COUNT`, per-model usage counters).
- **`AdaptySDKPrefs.xml`** — Payment/subscription data (Adapty SDK): device model, Android version, app version, store country, timezone, test-user flag, previous app instance ID, total revenue in USD, paywall type, and last-update timestamp.

### `AIChatbotNovaMediastore`
Correlates three data sources to determine which files interacted with the app:

1. `HistoryDetailDocument` entries in `chat-ai.db` (attached documents)
2. Files physically present in `/sdcard/Android/media/com.scaleup.chatai/Nova/`
3. Files indexed in the Android `MediaStore` (`external.db`)

This module distinguishes between files still present locally, files uploaded to Firebase, and files synced from another device with the same account.

### `AIChatbotNovaConversations`
Provides a single, fully reconstructed conversation report by correlating the outputs of the other modules, including:

- Complete conversation threads (chronological)
- User and chatbot messages
- Attached file paths recovered via MediaStore
- Token usage and message-level metadata

---

## 🛠️ Forensic Context

This work was developed as part of a Master's degree in Cybersecurity and Digital Forensics at the **Polytechnic Institute of Leiria** and represents the **first known forensic study** of the AI Chatbot – Nova application for Android.

Key findings that motivated these modules include:

- Deleted conversations removed from the UI are **not actually removed** from the SQLite database (`softDeleted` flag remains `0`), enabling recovery.
- Complete conversation histories, timestamps, token counts, and Firebase Cloud Storage URLs remain accessible in `chat-ai.db`.
- User PII (email, username, Firebase UID) is recoverable from shared preferences via an unencrypted JWT.
- Image attachments (AI-generated and user-submitted) are cached locally in `cache/image_manager_disk_cache/` (Glide cache) — including thumbnails of videos and full-sized original images.
- All user-uploaded documents and media are confirmed to be retained server-side on Firebase Cloud Storage.

For full analysis details, refer to the accompanying report (`Team6-report.pdf`).
