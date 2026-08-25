# 【Day 23】MCP 是什麼？把外部工具接進 Claude 的原理與實作

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

Day 22 講 Claude Code 怎麼管理 MCP 伺服器的 token 開銷，但沒解釋 MCP 本身是什麼。今天把它補上——這是一個你會在 Claude Code、claude.ai，甚至自己寫的 API 整合裡反覆遇到的名詞。

先講白話版定義：**MCP（Model Context Protocol）是一套讓 AI 存取外部工具與資料的通用協定。** 沒有 MCP 之前，你想讓 Claude 查資料庫、串 Slack、讀 GitHub issue，每一種服務都要自己寫一套客製化的工具定義（Day 9 提過，工具定義本身就要算 token，還要自己維護）。MCP 的價值在於：**服務提供者只要照著這套協定實作一次伺服器，任何支援 MCP 的 AI 應用都能直接接上，不用重新發明輪子。**

> 本篇 MCP connector 的規格與範例，於 **2026 年 8 月 17 日**對照 [MCP connector 官方文件](https://platform.claude.com/docs/en/agents-and-tools/mcp-connector) 查證。

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
| Day 29 | Vibecoding 做出網站之後：AI 不會主動告訴你的那些事 | 門檻降低的是「做出來」，不是「做對」——怎麼問出你不知道要問的問題 |
| Day 30 | Claude 使用總整理：模型、成本、設定一次看懂 | 全系列濃縮成一份可以收藏的速查表 |

## 一、兩種接法：API 端的 MCP Connector vs 自己管理的 MCP Client

官方文件開宗明義點出兩條路，選哪一條取決於你的伺服器在哪裡：

**① MCP Connector（今天的主角）**：直接在 Messages API 請求裡宣告要連的 MCP 伺服器，**不需要你自己實作一套 MCP client**。適用條件很明確——伺服器必須是**遠端、可透過 URL 存取**的。

**② Client-side 輔助工具**：如果你要接的是**本機執行的 stdio 伺服器**，或需要用到 MCP 的 prompts、resources 功能，或想對連線有更細的控制，才需要自己管理 MCP client，官方 SDK 提供轉換函式（例如 Python 的 `async_mcp_tool`、TypeScript 的 `mcpTools`）省去手動轉換的工夫。

今天聚焦在第一種——多數應用場景都是這一種。

## 二、什麼時候 Claude 會真的呼叫 MCP 工具

這個判準很重要，直接影響 Day 9 提過的 token 消耗：

> "Claude does **not** call an MCP tool for general knowledge questions about a connected service. Asking 'how do Notion databases work?' with a Notion server attached is answered directly; asking 'what's in my Projects database?' triggers the tool."

**問「Notion 資料庫怎麼運作」這種一般性知識問題，Claude 不會呼叫工具，直接用自己的知識回答；問「我的 Projects 資料庫裡有什麼」這種需要實際查詢的問題，才會觸發工具呼叫。** 這代表接了 MCP 伺服器，不代表每次對話都會產生工具呼叫的額外成本——**Claude 會判斷這次的請求是不是真的需要外部資料**。如果你發現它呼叫工具的頻率不如預期，Day 19 提過的 system prompt 技巧可以派上用場——用系統提示引導它更積極或更保守地觸發工具。

## 三、基本設定：兩個陣列各司其職

MCP Connector 用兩個獨立的參數協同運作：

```python
response = client.beta.messages.create(
    model="claude-opus-5",
    max_tokens=1000,
    messages=[{"role": "user", "content": "What tools do you have available?"}],
    mcp_servers=[
        {
            "type": "url",
            "url": "https://example-server.modelcontextprotocol.io/sse",
            "name": "example-mcp",
            "authorization_token": "YOUR_TOKEN",
        }
    ],
    tools=[{"type": "mcp_toolset", "mcp_server_name": "example-mcp"}],
    betas=["mcp-client-2025-11-20"],
)
```

**`mcp_servers`** 定義連線細節——URL、名稱、認證權杖；**`tools` 裡的 `mcp_toolset`** 則定義「這個伺服器裡的哪些工具要開放、怎麼配置」。這個設計把「連到哪裡」跟「能用什麼」分開，對應到官方三條驗證規則：每個 `mcp_server_name` 必須有對應的 server 定義、每個 server 都必須被剛好一個 toolset 引用、一個 server 不能被兩個 toolset 重複引用。

**這是目前的正式版本（`mcp-client-2025-11-20`）**——如果你看到教學文章裡把工具設定直接寫在 `mcp_servers` 裡面的 `tool_configuration` 欄位，那是已棄用的舊版（`mcp-client-2025-04-04`），設定方式已經搬到 `tools` 陣列裡的 `mcp_toolset` 結構，發文前如果要示範程式碼，記得用新版格式。

## 四、常見配置模式：允許清單、拒絕清單、延遲載入

**允許清單（Allowlist）**：預設全部關閉，只開放你明確列出的工具：

```json
{
  "type": "mcp_toolset",
  "mcp_server_name": "google-calendar-mcp",
  "default_config": { "enabled": false },
  "configs": {
    "search_events": { "enabled": true },
    "create_event": { "enabled": true }
  }
}
```

**拒絕清單（Denylist）**：預設全部開放，只關閉危險或不想要的操作。官方特別建議這個模式適合用在**唯讀助理**，或**需要人工確認才能執行的變更型操作**：

```json
{
  "type": "mcp_toolset",
  "mcp_server_name": "google-calendar-mcp",
  "configs": {
    "delete_all_events": { "enabled": false },
    "share_calendar_publicly": { "enabled": false }
  }
}
```

**延遲載入（`defer_loading`）**：Day 22 提過 Claude Code 裡 MCP 工具定義預設延遲載入，這個機制在 API 層級也存在——設成 `true` 的工具，一開始只有名稱進 context，完整描述要等真正被搜尋到才載入，搭配 Tool search tool 用在工具數量很多的情境。**這正是 Day 22 那句「MCP 工具定義預設延遲載入」的 API 端原始出處。**

## 五、目前的限制：只支援工具呼叫，伺服器必須公開

官方明確列出兩條限制：

- **MCP 規格裡只有「工具呼叫（tool calls）」這部分目前受 Connector 支援**，其他 MCP 功能（prompts、resources）要用 client-side 輔助工具。
- **伺服器必須透過 HTTP 公開曝露**（支援 Streamable HTTP 與 SSE 兩種傳輸方式）。**本機執行的 stdio 伺服器沒辦法直接用 Connector 連上**——這是第一節提到「兩條路」分岔的根本原因。

## 六、認證：OAuth Bearer Token

需要認證的 MCP 伺服器，透過 `authorization_token` 欄位傳入 OAuth 存取權杖：

```json
{
  "mcp_servers": [{
    "type": "url",
    "url": "https://example-server.modelcontextprotocol.io/sse",
    "name": "authenticated-server",
    "authorization_token": "YOUR_ACCESS_TOKEN_HERE"
  }]
}
```

**取得與更新這個權杖的責任在你這邊**——官方明講：「API consumers are expected to handle the OAuth flow and obtain the access token prior to making the API call, and to refresh the token as needed.」如果只是想先測試，官方推薦用 MCP Inspector（`npx @modelcontextprotocol/inspector`）跑一次 OAuth 流程，拿到測試用的權杖。

## 七、計價與資料保留：兩件容易被忽略的事

**計價**：MCP 工具呼叫**跟一般 Messages API 請求用同一套計價方式**，而且 Batch API（Day 10）也支援 `mcp_servers`——批次請求裡的 MCP 工具呼叫，計費方式跟即時請求一致，沒有額外的價差。

**資料保留**：官方明確標記 **MCP connector 不符合零資料保留（ZDR）資格**——跟 MCP 伺服器交換的資料，包含工具定義與執行結果，會依照 Anthropic 標準資料保留政策處理。如果你的應用場景對資料保留有嚴格要求，這是設計架構前就該確認的一項限制，不是事後才發現的意外。

## 八、什麼時候該接 MCP，什麼時候不需要

Day 22 提過一個實務建議：「優先用 CLI 工具而非 MCP」。今天知道了 MCP 的完整機制之後，可以把這個判斷原則講得更完整。

**適合用 MCP 的情境**：你要接的服務**沒有現成的指令列工具**，或是你需要**跨多個服務統一管理權限**（今天第四節的允許清單／拒絕清單機制，能在同一套設定邏輯下管理不同服務的存取範圍）。企業內部系統、SaaS 服務的官方或社群 MCP 伺服器，通常就是為了填補「沒有 CLI、只有網頁介面或私有 API」的空缺。

**不一定需要 MCP 的情境**：如果目標服務**已經有維護良好的 CLI 工具**（例如 `gh`、`aws`、`gcloud`），直接讓 Claude 呼叫這些工具，通常比接一個 MCP 伺服器更省事——不用管連線、認證權杖更新、伺服器是否公開暴露這些額外的維運負擔，而且 Day 22 提過 CLI 工具不會有任何「工具清單」的 context 開銷。

一個簡單的判斷句：**先問「這個服務有沒有好用的 CLI？」有,優先用 CLI；沒有,才考慮接 MCP。** 兩者不是互斥的——同一個工作流裡，同時用 CLI 工具處理一部分、MCP 伺服器處理另一部分（例如今天第七節的 Before/After 範例接的 Jira，多數團隊不會有 Jira 的官方 CLI），是很常見的組合。

## Before / After：接一個外部工具的兩種寫法

**❌ Before：自己寫一套客製化工具定義，手動維護**

```python
tools = [{
    "name": "search_jira_issues",
    "description": "手動維護的 Jira 搜尋工具描述……",
    "input_schema": {...},  # 手動對照 Jira API 文件自己寫
}]
# 每次 Jira API 改版，這裡都要跟著手動更新
```

**✅ After：接上官方或第三方提供的 MCP 伺服器**

```python
response = client.beta.messages.create(
    model="claude-opus-5",
    max_tokens=1000,
    messages=[{"role": "user", "content": "有哪些未解決的 Jira bug？"}],
    mcp_servers=[{
        "type": "url",
        "url": "https://your-jira-mcp-server.example.com/sse",
        "name": "jira-mcp",
        "authorization_token": JIRA_TOKEN,
    }],
    tools=[{"type": "mcp_toolset", "mcp_server_name": "jira-mcp"}],
    betas=["mcp-client-2025-11-20"],
)
```

> Before 版本每接一個新服務就要重新設計一套工具 schema，服務改版你也要跟著改。After 版本把「工具長什麼樣子」的維護責任交給 MCP 伺服器本身——**你只負責決定要不要接、開放哪些工具**，這正是 MCP 作為「通用協定」的價值所在。

## 本篇自我挑戰

- **今日挑戰**：如果你的團隊有內部系統（例如工單系統、內部知識庫），評估一下有沒有現成的 MCP 伺服器可以接，或者接一個公開的 MCP 伺服器（例如官方範例的 `example-mcp`）試跑一次今天的基本範例，觀察回應裡的 `mcp_tool_use` 與 `mcp_tool_result` 區塊長什麼樣子。

- **反思**：MCP 的核心價值是「一次實作、到處能用」的協定思維，這跟 Day 18 講的 XML 標籤、Day 17 的 prompt 骨架其實是同一種精神——**用標準化的結構，換取跨場景的可重複使用性。** 你在自己的工作裡，有沒有類似「每次都重新客製化，卻其實可以標準化一次就好」的地方？

## 總結

MCP 是一套讓 Claude 存取外部工具的通用協定，今天講的 **MCP Connector** 是 API 層級最直接的接法——**不用自己管理 MCP client，只要伺服器公開暴露在 HTTP 上**。`mcp_servers` 定義連線、`tools` 裡的 `mcp_toolset` 定義權限，允許清單、拒絕清單、延遲載入三種模式可以組合使用。

記住兩個容易被忽略的限制：**目前只支援工具呼叫，其他 MCP 功能要靠 client-side SDK**；**不符合 ZDR 資格**，資料保留採標準政策。計價上是好消息——跟一般請求同一套費率，批次 API 也能用。

**本日關鍵字回顧**

- **MCP（Model Context Protocol）**：讓 AI 存取外部工具與資料的通用協定，服務提供者實作一次，任何支援 MCP 的應用都能接上。
- **MCP Connector**：API 層級直連遠端 MCP 伺服器的方式，透過 `mcp_servers` 與 `mcp_toolset` 兩個參數設定。
- **智慧觸發**：Claude 只在真正需要外部資料時才呼叫 MCP 工具，一般性知識問題直接回答，不產生額外呼叫成本。
- **允許清單 / 拒絕清單 / 延遲載入**：三種常見的工具配置模式，拒絕清單適合唯讀或需人工確認的場景。
- **限制**：僅支援工具呼叫（不含 prompts/resources）、伺服器須公開於 HTTP、不符合零資料保留資格。

明天從 Claude Code 這種現成工具，回到最基礎的一層——**自己寫程式呼叫 Claude API**。第一支 Messages API 請求，以及為什麼你不該在瀏覽器裡直接呼叫它。

**Day 24，Messages 端點入門。**
