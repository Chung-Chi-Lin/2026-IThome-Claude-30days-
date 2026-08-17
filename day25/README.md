# 【Day 25】Claude 串流輸出（Streaming）：打造即時回應體驗

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

Day 24 那支最基本的請求，有一個使用體驗上的缺點：**你得等模型把整段回答都生成完，才會拿到任何東西。** 如果這次的回答需要思考很久（Day 15 講過，thinking 會讓生成時間拉長），使用者盯著一片空白的畫面等上好幾秒甚至更久，體驗很差。

今天的主角是 **Streaming（串流輸出）**——讓回應像打字機一樣一段一段即時顯示，而不是全部生成完才一次給你。這不是讓 Claude 「回答得更快」，是讓你**更快看到它已經寫好的部分**。

> 本篇 SSE 事件格式與範例，於 **2026 年 8 月 17 日**對照 [Streaming messages 官方文件](https://platform.claude.com/docs/en/build-with-claude/streaming) 查證。

## 目錄

| 天數 | 主題 | 描述 |
| :--- | :--- | :--- |
| Day 1 | Claude 模型怎麼選？2026 最新四階模型完整比較 | Fable 5 / Opus 5 / Sonnet 5 / Haiku 4.5 的定位、規格與適用場景 |
| Day 2 | Claude 的計價邏輯：搞懂 input / output 為什麼差 5 倍 | 不背數字，理解計價結構，建立可長期沿用的成本直覺 |
| Day 3 | 不知道用哪個模型？官方建議「從 Opus 5 開始」背後的思維 | 為什麼預設起手不是最便宜、也不是最強的那個 |
| Day 4 | Claude Haiku 4.5 適合做什麼？便宜模型的正確用法 | 便宜模型不是次等品，是專用工具 |
| Day 5 | Claude context window 是什麼？1M token 到底能塞多少東西 | 用實際檔案量換算，破除「塞越多越好」的迷思 |
| Day 6 | Claude 模型選擇決策表：一張圖判斷你該用哪一個 | 把前五天濃縮成一張可以貼在螢幕旁的決策流程 |
| Day 7 | Token 是什麼？為什麼你的 Claude 帳單比想像中貴 | 從 tokenizer 原理理解中文為什麼特別燒錢 |
| Day 8 | Claude 省 token 的 5 個實用技巧（一般使用者也適用） | 不寫程式也能立刻套用的五個習慣 |
| Day 9 | Prompt Caching 是什麼？讓重複內容只算 10% 費用 | 快取寫入與命中的計價邏輯，以及什麼時候會虧 |
| Day 10 | Claude Batch API 教學：非即時任務直接省一半費用 | 用時間換金錢，非同步任務的正確打開方式 |
| Day 11 | 新世代 tokenizer：同樣的中文為什麼變貴了 | Claude 4.7 世代換了 tokenizer，這對中文使用者的實際影響 |
| Day 12 | 對話越長越燒錢？Claude 長對話的成本陷阱與解法 | 每一輪都重算全部歷史——以及三種切斷成本累積的做法 |
| Day 13 | Claude 用量怎麼監控？成本失控前的預警機制 | 從 usage 欄位到 Console 儀表板，把帳單變成可觀測系統 |
| Day 14 | Claude effort 參數是什麼？五個檔位該怎麼設 | `low` / `medium` / `high` / `xhigh` / `max` 的取捨與實測建議 |
| Day 15 | Adaptive Thinking 是什麼？為什麼你不用再寫「think step by step」 | 模型自己決定何時思考，舊 prompt 技巧為何失效 |
| Day 16 | Claude 回答變淺了？檢查這兩個隱藏設定 | 排查思路：先看 effort，再看 thinking 設定 |
| Day 17 | Claude Prompt 寫法教學：官方最佳實踐的骨架 | 一個可以套用在 90% 情境的 prompt 結構 |
| Day 18 | 用 XML 標籤讓 Claude 輸出更穩定（結構化輸出教學） | 為什麼 Claude 特別吃 XML，以及怎麼設計標籤 |
| Day 19 | System Prompt 怎麼寫？角色設定的正確姿勢 | system 與 user 的分工，以及「你是一位專家」為什麼沒用 |
| Day 20 | Claude 幻覺怎麼防？降低錯誤輸出的實用做法 | 引用來源、允許說不知道、把驗證寫進流程 |
| Day 21 | Claude Code 是什麼？安裝與第一次使用完整教學 | 從安裝到跑完第一個任務，含常見卡關點 |
| Day 22 | Claude Code 省 token 設定：別讓它讀完整個專案 | `CLAUDE.md`、忽略規則與 context 控制的實戰配置 |
| Day 23 | MCP 是什麼？把外部工具接進 Claude 的原理與實作 | Model Context Protocol 的設計哲學與一個可跑的範例 |
| Day 24 | 前端如何呼叫 Claude API？Messages 端點入門 | 第一支 API 請求，以及為什麼不該在瀏覽器直接呼叫 |
| Day 25 | Claude 串流輸出（Streaming）：打造即時回應體驗 | SSE 事件流解析與前端逐字渲染 |
| Day 26 | Claude API 錯誤處理與重試：正式環境該注意什麼 | 429 / 529 的正確退避策略與冪等性設計 |
| Day 27 | 模型分流（Model Routing）是什麼？別再一支模型用到底 | 依任務難度動態選模型的判斷邏輯 |
| Day 28 | LLM 成本優化架構：小模型前置分流 + 大模型收尾 | 一套可落地的分層架構與失敗處理 |
| Day 29 | 從「會用」到「用得對、用得省」：我 30 天的踩坑與心法 | 誠實記錄過程中判斷錯誤的地方 |
| Day 30 | Claude 使用總整理：模型、成本、設定一次看懂 | 全系列濃縮成一份可以收藏的速查表 |

## 一、啟用方式：一個參數的差別

把 Day 24 的請求加上 `"stream": true`，回應就會變成一連串 **Server-Sent Events（SSE）**，而不是一個完整的 JSON：

```python
client = anthropic.Anthropic()

with client.messages.stream(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
    model="claude-opus-5",
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

SDK 幫你把底層的事件解析都包好了，`stream.text_stream` 直接給你一段一段的文字。如果你要自己處理原始事件（例如自訂前端渲染邏輯），底層長這樣：

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model": "claude-opus-5", "max_tokens": 1024, "stream": true, "messages": [{"role": "user", "content": "Hello"}]}'
```

## 二、事件流程：固定的四個階段

官方定義了每次串流的標準骨架：

1. **`message_start`**：一個內容為空的 `Message` 物件，代表這次回應開始了。
2. **一系列內容區塊**：每個內容區塊依序有 `content_block_start` → 一或多個 `content_block_delta` → `content_block_stop`。**`index` 欄位對應到最終 `Message.content` 陣列裡的位置**——這代表你可以邊收邊組出最終結果的結構。
3. **一或多個 `message_delta`**：回報整個 `Message` 物件頂層的變化（例如 `stop_reason` 確定下來）。
4. **`message_stop`**：這次回應結束。

實際跑起來長這樣（節錄）：

```text
event: message_start
data: {"type":"message_start","message":{"id":"msg_...","content":[],"usage":{"input_tokens":25,"output_tokens":1}}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: ping
data: {"type":"ping"}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"!"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn"},"usage":{"output_tokens":15}}

event: message_stop
data: {"type":"message_stop"}
```

**`ping` 事件會穿插在整個流程中出現**，沒有實質內容，純粹是保持連線用的心跳，你的解析邏輯應該直接忽略它。

## 三、三種 Delta 類型，對應三種內容區塊

`content_block_delta` 事件的 `delta` 欄位，依內容區塊的型別而不同：

**文字（`text_delta`）**：最常見的類型，逐字或逐詞把文字補上。

**工具輸入（`input_json_delta`）**：Claude 呼叫工具時，工具的輸入參數會以**局部 JSON 片段**的方式串流出來：

```text
event: content_block_delta
data: {"type":"content_block_delta","index":1,"delta":{"type":"input_json_delta","partial_json": "{\"location\": \"San Fra"}}}
```

官方提醒：**目前的模型一次只會完整串出一個 key-value 屬性**，所以工具呼叫的串流事件之間可能會有延遲。你需要累積這些片段、直到收到 `content_block_stop` 才把完整 JSON 解析出來——官方建議用支援局部解析的函式庫（例如 Pydantic），或直接用 SDK 提供的輔助工具。

**思考內容（`thinking_delta` + `signature_delta`）**：Day 15 講的 thinking 區塊在串流時也會逐段送出，而且在 `content_block_stop` 之前，會多送一個特殊的 `signature_delta` 事件——**這個簽章用來驗證思考區塊的完整性**，如果你的應用會把 thinking 區塊原封不動送回下一輪請求（Day 15 提過這是必要的），這個簽章要一併保留。

## 四、`message_delta` 裡的 usage 是累積值，別自己再加總

這是一個容易踩的坑，官方特別用警告框標出來：

> "The token counts shown in the `usage` field of the `message_delta` event are **cumulative**."

如果同一次串流過程收到好幾個 `message_delta` 事件，**每一個裡面的 `usage.output_tokens` 都是目前為止的累積總數**，不是這次事件新增的量。如果你的程式邏輯是「每收到一個 `message_delta` 就把 usage 加總一次」，你會得到一個嚴重灌水的錯誤數字——正確做法是**只採用最後一個 `message_delta` 事件裡的 usage 當作這次請求的最終值**。

## 五、串流中途出錯：跟平常的錯誤處理不一樣

Day 26 明天要完整講 API 錯誤處理，這裡先埋一個跟串流有關的重點。官方提醒：

> "When receiving a streaming response over server-sent events (SSE), an error can occur after the API returns a 200 response. In that case, error handling doesn't follow these standard mechanisms."

**串流請求一開始通常會拿到 HTTP 200**（代表連線建立成功），但**真正的錯誤可能在串流過程中才發生**，以一個獨立的 `error` 事件送出：

```text
event: error
data: {"type": "error", "error": {"type": "overloaded_error", "message": "Overloaded"}}
```

這代表你不能只靠檢查 HTTP 狀態碼判斷這次請求有沒有出錯——**串流情境下，你必須在事件迴圈裡額外監聽 `error` 事件類型**，才能正確捕捉到中途發生的問題（例如 Day 26 會講的 529 過載錯誤，串流時是以這種方式呈現，而不是直接回傳 HTTP 529）。

官方也提醒：依照版本演進政策，未來可能會新增新的事件類型，**你的解析邏輯應該對未知的事件類型優雅地忽略，而不是直接拋錯中斷**——這是為了向前相容，避免哪天官方加了一個新事件類型，你的程式就整個爆掉。

## 六、不想自己處理事件？直接拿最終結果

如果你的場景不需要逐字顯示效果，只是想避開 Day 26 會講的長請求逾時風險（官方建議超過 10 分鐘的請求改用串流或批次），SDK 提供了「串流接收、但一次性拿完整結果」的寫法：

```python
with client.messages.stream(
    max_tokens=128000,
    messages=[{"role": "user", "content": "Write a detailed analysis..."}],
    model="claude-sonnet-5",
) as stream:
    message = stream.get_final_message()

print(next(block.text for block in message.content if block.type == "text"))
```

這種寫法拿到的 `message` 物件，跟非串流請求回傳的格式完全一樣——**你用串流的方式建立連線（避開長時間閒置連線被中斷的風險），但程式邏輯可以維持跟 Day 24 一樣的處理方式**，不用自己組裝事件。

## 七、串流也解決了一個網路層的問題

除了改善使用者的等待感受，串流還有一個比較少人注意到的實際用途——**降低長時間請求被網路中斷的風險**。Day 26 會完整講錯誤處理，這裡先講跟串流相關的那部分：某些網路環境會在連線閒置一段時間後自動斷開，如果你的請求需要生成很長的內容（例如 Day 15 提過的高 effort 深度思考，或要求輸出一篇長文件），一個沒有任何資料流動的長時間連線特別容易被判定為閒置而中斷。

串流連線因為持續有事件送出，被判定為閒置的機率低得多。這也是為什麼官方建議**超過 10 分鐘的請求優先用串流或批次**，而不是拉長一個非串流的單次請求硬撐過去——與其事後處理連線中斷的重試邏輯，不如一開始就選一種不容易被中斷的傳輸方式。

## Before / After：一次性等待 vs 逐字顯示

**❌ Before：等模型講完整段話，畫面空白幾秒**

```javascript
const response = await fetch("/api/chat", {
  method: "POST",
  body: JSON.stringify({ messages }),
});
const data = await response.json();
renderMessage(data.content[0].text);  // 使用者盯著空白畫面，直到這裡才看到任何東西
```

**✅ After：邊生成邊顯示**

```javascript
const response = await fetch("/api/chat-stream", { method: "POST", body: JSON.stringify({ messages }) });
const reader = response.body.getReader();
const decoder = new TextDecoder();
let buffer = "";

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  buffer += decoder.decode(value, { stream: true });

  const events = buffer.split("\n\n");
  buffer = events.pop();
  for (const raw of events) {
    const dataLine = raw.split("\n").find((l) => l.startsWith("data:"));
    if (!dataLine) continue;
    const event = JSON.parse(dataLine.slice(5));
    if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
      appendToMessage(event.delta.text);  // 每收到一小段就立刻渲染
    }
  }
}
```

> Before 版本的等待時間，取決於整段回答生成完的總時間——如果 Claude 想得比較久（例如 Day 15 提到的高 effort 深度推理），使用者可能要盯著空白畫面好幾秒。After 版本讓使用者在第一個 token 生成出來的當下就開始看到內容，**總生成時間不會變快，但「感覺等了多久」大幅縮短**——這正是串流輸出真正解決的問題：不是縮短生成時間，是縮短使用者感受到的等待時間。

## 本篇自我挑戰

- **今日挑戰**：如果你目前的應用是非串流呼叫，挑一個對話式的介面，改成串流版本，實際比較使用者「感覺到第一個字出現」的時間差異。如果你已經在用串流，檢查你的 usage 統計邏輯有沒有誤把 `message_delta` 的累積值重複加總。

- **反思**：串流輸出解決的是「感受到的等待時間」，不是「實際運算時間」——這跟很多 UX 設計原則是同一種思路：**進度條、骨架畫面（skeleton screen）都是類似的邏輯，讓使用者感覺快，而不是真的讓系統變快。** 你在其他系統設計上，有沒有類似「處理感受，而不是處理實際數字」的經驗？

## 總結

串流輸出把一次性的完整回應，拆成一連串 SSE 事件：**`message_start` → 內容區塊的 start/delta/stop 循環 → `message_delta` → `message_stop`**，中間穿插心跳用的 `ping` 事件。三種 delta 類型——文字、工具輸入的局部 JSON、思考內容——分別對應不同的內容區塊型別。

三個容易踩的坑今天都講到了：**`message_delta` 裡的 usage 是累積值，不要重複加總**；**中途錯誤是以獨立的 `error` 事件送出，不是靠 HTTP 狀態碼判斷**；**未知事件類型要優雅忽略，為未來的新事件類型留相容空間。**

**本日關鍵字回顧**

- **`stream: true`**：啟用 SSE 串流輸出的參數，回應變成一連串事件而非單一 JSON。
- **事件流程**：`message_start` → `content_block_start/delta/stop` → `message_delta` → `message_stop`，`ping` 穿插其中。
- **三種 delta 類型**：`text_delta`（文字）、`input_json_delta`（工具輸入的局部 JSON）、`thinking_delta`/`signature_delta`（思考內容與簽章）。
- **累積 usage**：`message_delta` 事件裡的 token 數是累積值，只取最後一個事件的數字即可，不可加總。
- **串流中途錯誤**：以獨立的 `error` 事件送出，HTTP 200 之後仍可能出錯，需在事件迴圈裡額外監聽。

明天完整處理錯誤處理與重試——**429 跟 529 有什麼不同、官方建議的退避策略，以及正式環境上線前該檢查的冪等性設計。**

**Day 26，API 錯誤處理與重試實戰。**
