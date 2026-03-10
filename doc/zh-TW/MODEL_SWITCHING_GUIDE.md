# 🔀 模型切換指南 — 讓 Rokid 智能眼鏡使用你指定的 AI 模型

> 本指南說明如何透過 **RokidAIAssistant** 這個 Android App，切換智能眼鏡所使用的 AI 模型與服務商。

---

## 📐 系統架構概述

```
[Rokid 眼鏡] ──藍牙 SPP──> [手機 phone-app]
                                    │
                          SettingsRepository
                          (EncryptedSharedPreferences)
                                    │
                          AiServiceFactory
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
         GeminiService     AnthropicService    OpenAiCompatibleService
              │                                           │
       （以及其他服務商）                    (DeepSeek / Groq / xAI / Alibaba ...)
```

**眼鏡本身不直接呼叫 AI API。** 眼鏡透過藍牙將語音/照片傳到手機，手機的 `phone-app` 負責呼叫 AI 服務商 API，再把結果推回眼鏡顯示。

---

## ✅ 支援的 AI 模型一覽

### Google Gemini
| 模型 ID | 顯示名稱 | 備註 |
|---------|----------|------|
| `gemini-3.1-pro-preview` | Gemini 3.1 Pro (Preview) | 最強，Preview 版 |
| `gemini-3-flash-preview` | Gemini 3 Flash (Preview) | 快速低成本，Preview 版 |
| `gemini-2.5-pro` | Gemini 2.5 Pro | 穩定版，推薦首選 |
| `gemini-2.5-flash` | Gemini 2.5 Flash | 預設模型，速度/品質平衡 |
| `gemini-2.5-flash-lite` | Gemini 2.5 Flash-Lite | 最快最省，適合高頻查詢 |

### Anthropic Claude
| 模型 ID | 顯示名稱 | Context |
|---------|----------|---------|
| `claude-opus-4-6` | Claude Opus 4.6 | 200K（beta 可達 1M）|
| `claude-sonnet-4-6` | Claude Sonnet 4.6 | 200K（beta 可達 1M）|
| `claude-haiku-4-5-20251001` | Claude Haiku 4.5 | 200K，最快 |

### OpenAI
| 模型 ID | 顯示名稱 |
|---------|----------|
| `gpt-5.2` | GPT-5.2 |
| `gpt-5.1` | GPT-5.1（推薦通用）|
| `gpt-5-mini` | GPT-5 Mini（省錢版）|
| `o3` | o3（深度推理）|

### xAI Grok
| 模型 ID | 顯示名稱 |
|---------|----------|
| `grok-4.1-fast` | Grok 4.1 Fast（最大 2M context）|
| `grok-4` | Grok 4（純推理模型）|
| `grok-3` | Grok 3 |

### 其他支援服務商
- **DeepSeek**：`deepseek-chat`、`deepseek-reasoner`
- **Groq**：Llama 4 Scout / Maverick（超低延遲）
- **Alibaba**：Qwen 3 Max、Qwen 2.5 VL（視覺）
- **自訂（Ollama / LM Studio）**：本地模型

---

## 📱 切換模型的方法

### 方法一：在手機 App 的設定畫面切換（最簡單）

1. 打開手機上的 **RokidAIAssistant** App
2. 進入 **設定（Settings）** 頁面
3. 找到 **AI Provider** 區塊 → 選擇你要的服務商（如 Anthropic）
   - 切換服務商時，App 會自動選擇該服務商的第一個模型
4. 找到 **Model** 區塊 → 選擇具體模型（如 Claude Sonnet 4.6）
5. 確保對應服務商的 **API Key** 已填入
6. 返回主畫面，重新開始對話 → 眼鏡即使用新模型

> 💡 設定會即時儲存（EncryptedSharedPreferences），不需要重啟 App。

---

### 方法二：使用自訂端點（Ollama / LM Studio 本地模型）

如果你想讓眼鏡使用本機跑的開源模型（如 Llama 4、DeepSeek R1）：

1. 在電腦上安裝並啟動 **Ollama**：
   ```bash
   ollama run llama4
   # 預設開在 http://localhost:11434
   ```
2. 手機 App 設定 → **AI Provider** 選 `Custom`
3. 填入：
   - **Base URL**：`http://你電腦的IP:11434/v1/`（確保手機和電腦在同一 Wi-Fi）
   - **Model Name**：`llama4`（或你 ollama 有的模型名）
4. API Key 可留空（Ollama 不需要）

> ⚠️ 本機模型延遲較高，不建議用於需要即時回應的眼鏡場景。

---

### 方法三：修改預設模型（修改程式碼，永久生效）

如果你想讓 App 每次打開就預設使用特定模型，修改以下檔案：

**檔案**：`phone-app/src/main/java/com/example/rokidphone/data/ApiSettings.kt`

```kotlin
data class ApiSettings(
    val aiProvider: AiProvider = AiProvider.GEMINI,   // ← 改這裡
    val aiModelId: String = "gemini-2.5-flash",        // ← 改這裡
    ...
)
```

**範例：改成預設使用 Claude Sonnet**
```kotlin
data class ApiSettings(
    val aiProvider: AiProvider = AiProvider.ANTHROPIC,
    val aiModelId: String = "claude-sonnet-4-6",
    ...
)
```

修改後重新建置：
```bash
./gradlew assembleDebug
```

---

## 🔑 各服務商 API Key 設定位置

### 取得 API Key

| 服務商 | 取得網址 |
|--------|----------|
| Google Gemini | https://ai.google.dev/ |
| Anthropic | https://console.anthropic.com/ |
| OpenAI | https://platform.openai.com/ |
| Groq | https://console.groq.com/ |
| xAI | https://console.x.ai/ |
| DeepSeek | https://platform.deepseek.com/ |

### 填入方式

**方式 A：應用內設定（推薦，加密儲存）**
- 設定頁 → 對應服務商欄位 → 貼上 API Key

**方式 B：建置時注入（寫在 `local.properties`，適合開發）**

```properties
# RokidAIAssistant/local.properties（不要 commit 到 git！）
GEMINI_API_KEY=AIzaSy...
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
GROQ_API_KEY=gsk_...
XAI_API_KEY=xai-...
```

---

## ⚙️ 進階：調整 LLM 生成參數

切換模型後，你還可以微調回應品質。在設定頁找到 **LLM Parameters**：

| 參數 | 預設值 | 說明 |
|------|--------|------|
| Temperature | 0.7 | 越高越有創意，越低越精準 |
| Max Tokens | 2048 | 單次回應最大長度 |
| Top P | 1.0 | 核採樣，通常不需動 |
| Frequency Penalty | 0.0 | 減少重複用詞 |
| Presence Penalty | 0.0 | 鼓勵提到新話題 |

**眼鏡場景建議設定（簡短回答）：**
```
Temperature: 0.5
Max Tokens: 512
```

**需要深度分析時（如拍照問答）：**
```
Temperature: 0.7
Max Tokens: 2048
```

---

## 🎙️ 語音辨識（STT）也可以換

眼鏡收音後，語音辨識用的服務商和 AI 模型是分開的。設定 → **Speech Recognition Provider**：

| 服務商 | 特色 |
|--------|------|
| Gemini（預設）| 多語言，免費額度 |
| OpenAI Whisper | 準確度高 |
| Groq Whisper | 超低延遲 |
| Azure Speech | 企業級，支援即時串流 |
| Deepgram | 即時串流，延遲極低 |

---

## 🔄 推薦的模型組合

| 使用場景 | 推薦 AI 模型 | 推薦 STT |
|----------|-------------|----------|
| 日常問答（速度優先）| Gemini 2.5 Flash | Gemini |
| 複雜分析（品質優先）| Claude Sonnet 4.6 | Gemini |
| 看圖說話（拍照功能）| Gemini 2.5 Pro 或 Qwen 2.5 VL 72B | Gemini |
| 省錢模式 | Groq Llama 4 Scout | Groq Whisper |
| 隱私優先（本地）| Ollama llama4 | 使用裝置語音辨識 |

---

## ❓ 常見問題

**Q: 切換模型後眼鏡沒有反應？**
```
A: 確認手機 App 和眼鏡之間的藍牙連線正常。
   重新長按眼鏡的觸控板喚醒 AI 功能。
```

**Q: 切換到 Claude 但一直失敗？**
```
A: Claude 不使用 OpenAI 格式，確認設定中選的是 ANTHROPIC 而非 CUSTOM。
   API Key 需在 https://console.anthropic.com/ 建立。
```

**Q: 用 Ollama 本地模型，手機連不到電腦？**
```
A: Ollama 預設只聽 localhost，需要改讓它監聽所有介面：
   OLLAMA_HOST=0.0.0.0:11434 ollama serve
   手機填的 IP 要是電腦在區域網路的 IP（如 192.168.1.xxx）。
```

**Q: 我可以用語音直接告訴眼鏡切換模型嗎？**
```
A: 目前不支援語音切換模型。切換必須透過手機 App 設定頁。
   若有需求，可開發「tool call」讓 AI 自動觸發模型切換。
```

---

## 📁 相關原始碼位置

| 功能 | 檔案路徑 |
|------|----------|
| 模型清單 | `phone-app/src/.../data/AIModel.kt` |
| 服務商定義 | `phone-app/src/.../data/ApiSettings.kt` |
| 設定儲存 | `phone-app/src/.../data/SettingsRepository.kt` |
| AI 服務建立 | `phone-app/src/.../service/ai/AiServiceFactory.kt` |
| 設定頁 UI | `phone-app/src/.../ui/SettingsScreen.kt` |

---

*最後更新：2026-03-10*
