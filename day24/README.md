# 【Day 24】前端如何呼叫 Claude API？Messages 端點入門

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

前 23 天不管講什麼參數、什麼技巧，底層都繞著同一個端點打轉：`/v1/messages`。今天回到最基礎的一層——**第一次自己寫程式呼叫它**，以及一個幾乎每個前端工程師都會冒出來的念頭：「能不能直接在瀏覽器裡呼叫就好，省一層後端？」

先講結論：**能，但你不應該。** 今天會講清楚為什麼，以及正確的架構長什麼樣子。

> 本篇 Messages API 的參數與回應結構，於 **2026 年 8 月 17 日**對照 [Create a Message 官方 API 參考](https://platform.claude.com/docs/en/api/messages/create) 查證。

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

## 一、最小可行請求：三個必要參數

`/v1/messages` 是個 `POST` 端點，官方定義：「Send a structured list of input messages with text and/or image content, and the model will generate the next message in the conversation.」

一個能跑的請求，只需要三個必要參數——`model`、`max_tokens`、`messages`：

```bash
curl https://api.anthropic.com/v1/messages \
  -H 'Content-Type: application/json' \
  -H 'anthropic-version: 2023-06-01' \
  -H "X-Api-Key: $ANTHROPIC_API_KEY" \
  -d '{
    "model": "claude-opus-5",
    "max_tokens": 1024,
    "messages": [
      {"role": "user", "content": "Hello, world"}
    ]
  }'
```

三個必帶的標頭：**`Content-Type`**（固定 `application/json`）、**`anthropic-version`**（API 版本號，官方靠這個欄位管理版本演進，不是隨便填的裝飾）、**`X-Api-Key`**（你的金鑰，第五節會講為什麼這個絕對不能出現在前端程式碼裡）。

`messages` 陣列的規則值得記住：**Claude 是照 `user` / `assistant` 交替回合訓練的**——連續兩個 `user` 角色的訊息，API 會自動把它們合併成單一回合處理。

## 二、回應結構：拆解一個真實的回傳

```json
{
  "id": "msg_013Zva2CMHLNnXjNJJKqJ2EF",
  "type": "message",
  "role": "assistant",
  "model": "claude-opus-5",
  "content": [
    { "type": "text", "text": "Hi! My name is Claude." }
  ],
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": { "input_tokens": 10, "output_tokens": 8 }
}
```

幾個欄位值得對照前面幾天學到的東西：

- **`content` 是一個陣列，不是一段字串。** 每個元素是一個「內容區塊」（content block），型別可能是 `text`、`tool_use`、`thinking`（Day 15）等。實務上要組出完整回覆文字，得自己遍歷這個陣列，篩出 `type === "text"` 的區塊。
- **`stop_reason` 告訴你這次生成為什麼停下來。** 常見值：`end_turn`（自然講完）、`stop_sequence`（撞到你設定的自訂停止字串）、`max_tokens`（Day 5、Day 15 都提過的那個「被截斷」訊號）。**這個欄位不是可有可無的除錯資訊，是你的程式必須主動檢查的東西**——只讀 `content` 不看 `stop_reason`，你可能把一段被截斷的回答當成完整答案處理掉。
- **`usage` 就是 Day 2、Day 9 反覆提過的那組計費數字**，`input_tokens`、`output_tokens`，有快取時還會多出 `cache_creation_input_tokens`、`cache_read_input_tokens`。

## 三、API 是無狀態的：多輪對話要自己組歷史

這是初學者最常踩的一個坑：**Messages API 本身不記得你們聊過什麼。** 每一次請求都是獨立的，如果你要做多輪對話，得自己把先前的往來紀錄組進 `messages` 陣列，整包送出去：

```json
{
  "model": "claude-opus-5",
  "max_tokens": 1024,
  "messages": [
    {"role": "user", "content": "我叫小明"},
    {"role": "assistant", "content": "你好小明！有什麼我能幫你的嗎？"},
    {"role": "user", "content": "我剛剛說我叫什麼名字？"}
  ]
}
```

**這正是 Day 12 講的「每一輪都要重付整段歷史」的第一手來源**——現在你知道這個行為不是 Claude Code 或哪個框架的設計選擇，是 Messages API 本身無狀態（stateless）的天生結構。你在前端存的對話陣列，就是每次請求都要完整送出去的東西。

## 四、為什麼不該在瀏覽器裡直接呼叫

技術上，Anthropic 確實提供了讓瀏覽器直接呼叫 API 的方式——加上一個特殊標頭就能繞開瀏覽器的 CORS 限制。**但這個標頭的名字本身就是最誠實的警告：`anthropic-dangerous-direct-browser-access`。**

「dangerous」不是隨便選的字。原因很直接：**要在瀏覽器裡呼叫 API，你的 API 金鑰必須出現在前端程式碼裡**——不管是寫死在 JS 檔案裡，還是從某個設定檔載入，只要程式碼會送到使用者的瀏覽器執行，任何打開開發者工具、檢查網路請求的人，都能看到你的金鑰。**金鑰外洩之後，別人可以拿去用你的額度，帳單算在你頭上。**

這不代表這個標頭完全沒有正當用途——官方文件本身承認，內部工具（只有受信任的使用者會用）、或是讓使用者自己輸入自己金鑰的「bring your own key」場景，直接在瀏覽器呼叫是合理的取捨。但**面向一般公眾的產品，金鑰一旦跟著前端程式碼送出去，就已經是公開的了。**

## 五、正確架構：前端 → 你的後端 → Claude API

```text
瀏覽器（前端）  →  你的後端伺服器  →  Claude API
                    （金鑰只存在這裡）
```

金鑰只放在後端環境變數裡，前端呼叫的是你自己的後端端點，由後端代為呼叫 Claude API 再把結果轉傳回去：

```javascript
// 後端（Node.js 範例，金鑰只存在伺服器環境）
app.post("/api/chat", async (req, res) => {
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "anthropic-version": "2023-06-01",
      "X-Api-Key": process.env.ANTHROPIC_API_KEY,  // 只在伺服器端讀取
    },
    body: JSON.stringify({
      model: "claude-opus-5",
      max_tokens: 1024,
      messages: req.body.messages,
    }),
  });
  const data = await response.json();
  res.json(data);
});
```

```javascript
// 前端只呼叫自己的後端，完全不接觸 API 金鑰
const res = await fetch("/api/chat", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ messages: conversationHistory }),
});
```

這一層後端代理除了保護金鑰，還順便給了你幾個額外的控制點：**可以在這裡做 Day 13 提過的用量監控、可以做速率限制保護自己的服務、可以做輸入驗證跟內容過濾**——這些事情如果讓前端直接呼叫 API，你完全沒有插手的機會。

## 六、原生 fetch 還是官方 SDK？

今天的範例用了原生 `fetch`／`curl`，方便看清楚底層長什麼樣子。但實務開發，官方提供 Python、TypeScript 等多種語言的 SDK，通常比自己組 HTTP 請求更值得用：

```javascript
// 原生 fetch：自己處理 headers、JSON 序列化、錯誤格式解析
const res = await fetch("https://api.anthropic.com/v1/messages", { /* ... */ });

// 官方 SDK：型別提示、自動重試、串流輔助都內建
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic();
const message = await client.messages.create({
  model: "claude-opus-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, world" }],
});
```

SDK 幫你處理的東西，會在後面幾天逐一展開：**型別提示**能讓你在寫程式階段就抓到參數打錯的問題；**自動重試**（Day 26 的主題）在暫時性錯誤發生時省去你自己寫退避邏輯；**串流輔助**（Day 25 的主題）把原始 SSE 事件包成好用的介面。今天先動手用原生 `fetch` 理解底層機制，之後正式專案裡，直接用 SDK 通常是更務實的選擇——除非你有特殊的執行環境限制（例如某些邊緣運算平台不支援安裝額外套件），不然沒有必要重新造輪子。

## Before / After

**❌ Before：前端直接呼叫，金鑰寫在程式碼裡**

```javascript
// 任何打開瀏覽器開發者工具的人都能看到 API_KEY
const API_KEY = "sk-ant-xxxxxxxxxxxx";
fetch("https://api.anthropic.com/v1/messages", {
  headers: {
    "X-Api-Key": API_KEY,
    "anthropic-dangerous-direct-browser-access": "true",
  },
  // ...
});
```

**✅ After：前端呼叫自家後端，金鑰留在伺服器**

```javascript
fetch("/api/chat", {
  method: "POST",
  body: JSON.stringify({ messages }),
});
// 金鑰完全不會出現在任何送到瀏覽器的程式碼裡
```

> Before 版本能跑，甚至第一次測試時感覺很順——這正是它危險的地方：**它不會馬上出事，出事的時候通常是金鑰已經外洩一段時間之後。** After 版本多了一層後端，代價是多寫一支代理端點，換到的是金鑰永遠不會離開你控制的環境。

## 七、先用 curl 驗證，再寫進程式碼

一個實務上很好用的除錯習慣：**寫任何整合程式碼之前，先用 curl 手動送一次請求，確認格式沒問題。** Day 10 的 Batch API 那篇也提過類似的建議——先驗證單筆請求的形狀，再包進更複雜的邏輯。原因很單純：curl 請求出錯時，錯誤訊息直接顯示在終端機，不用在你自己的程式碼裡加一堆 `console.log` 才找得到問題出在請求本身還是你的程式邏輯。

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model": "claude-opus-5", "max_tokens": 100, "messages": [{"role": "user", "content": "test"}]}'
```

跑得通，再把同樣的參數結構搬進你的後端程式碼；跑不通，你已經排除了「是我的應用程式邏辑寫錯」這個可能性，可以直接鎖定請求本身的問題。

## 本篇自我挑戰

- **今日挑戰**：如果你手上有任何前端直接呼叫 Claude API 的程式碼（哪怕只是測試用的），打開瀏覽器開發者工具的 Network 分頁，確認金鑰有沒有出現在請求標頭裡。如果有，這週就把它改成本篇的後端代理架構。

- **反思**：「先求能動，架構之後再說」是很常見的開發心態，尤其在快速原型階段。但金鑰外洩這種風險有個特性——**能動的階段感覺不到任何代價，代價會在未來某個你想不到的時間點一次爆發。** 你在其他地方有沒有類似「圖一時方便，卻埋下延遲代價」的技術債？

## 總結

今天從最基礎的一層講起：**Messages API 只需要 `model`、`max_tokens`、`messages` 三個必要參數**；**回應的 `content` 是陣列，`stop_reason` 是你的程式必須主動檢查的欄位**；**API 本身無狀態，多輪對話要自己組完整歷史送出去**——這正是 Day 12 成本累積問題的源頭。

最重要的一條：**API 金鑰絕對不能出現在前端程式碼裡。** `anthropic-dangerous-direct-browser-access` 這個標頭的名字本身就是最直白的警告，正確做法永遠是透過你自己的後端代理呼叫，金鑰只存在於你能控制的伺服器環境。

**本日關鍵字回顧**

- **`/v1/messages`**：Claude API 的核心端點，三個必要參數是 `model`、`max_tokens`、`messages`。
- **`content` 陣列**：回應內容以區塊陣列呈現，需遍歷篩選 `type === "text"` 才能取得完整文字。
- **`stop_reason`**：說明生成停止原因的欄位，`max_tokens` 代表被截斷，是程式必須主動檢查的欄位。
- **無狀態 API**：Messages API 不保留對話記憶，多輪對話需自行組裝完整歷史一併送出。
- **`anthropic-dangerous-direct-browser-access`**：允許瀏覽器直接呼叫 API 的標頭，名稱本身即是安全警告，正式產品應透過後端代理，金鑰不落地前端。

明天講一個能大幅改善使用體驗、卻也帶來新複雜度的功能——**Streaming。讓回應像打字機一樣即時顯示，而不是等模型講完整段話才看到結果。**

**Day 25，串流輸出打造即時回應體驗。**
