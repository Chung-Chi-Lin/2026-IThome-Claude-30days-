# 【Day 26】Claude API 錯誤處理與重試：正式環境該注意什麼

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

第四階段最後一天，把正式環境上線前最容易被輕忽的一塊補上：**錯誤處理與重試。** 開發階段，一個請求失敗了，你看一眼 log、重新跑一次就過去了。正式環境不是這樣——你的服務要在使用者流量、網路波動、Anthropic 端偶發過載的情況下，自己判斷「這個錯誤該不該重試、要等多久再試、重試會不會製造出新問題」。

今天把這些問題一次講完。

> 本篇錯誤碼、重試機制與驗證錯誤，於 **2026 年 8 月 17 日**對照 [Claude API errors 官方文件](https://platform.claude.com/docs/en/api/errors) 查證。

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

## 一、標準錯誤格式：一張表看懂所有 HTTP 碼

Claude API 用可預期的 HTTP 錯誤碼格式：

| 碼 | 錯誤類型 | 意義 |
| :--- | :--- | :--- |
| 400 | `invalid_request_error` | 請求格式或內容有問題 |
| 401 | `authentication_error` | API 金鑰有問題（格式錯誤、已撤銷、過期） |
| 402 | `billing_error` | 帳務或付款資訊有問題 |
| 403 | `permission_error` | 金鑰沒有權限使用指定資源 |
| 404 | `not_found_error` | 找不到請求的資源 |
| 409 | `conflict_error` | 請求與資源目前狀態衝突（例如唯一值已被使用） |
| 413 | `request_too_large` | 請求超過位元組上限 |
| 429 | `rate_limit_error` | 帳號撞到速率限制 |
| 500 | `api_error` | Anthropic 系統內部非預期錯誤 |
| 504 | `timeout_error` | 請求處理逾時 |
| 529 | `overloaded_error` | API 暫時過載 |

每個錯誤都用同一種 JSON 結構回傳：

```json
{
  "type": "error",
  "error": {
    "type": "not_found_error",
    "message": "The requested resource could not be found."
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

**官方 SDK 會把這些錯誤轉成各語言慣用的型別化例外**（Python 的 `anthropic.NotFoundError`、Ruby 的 `Anthropic::Errors::NotFoundError`、Go 的統一 `*anthropic.Error`），官方建議**攔截型別化的例外類別，而不是用字串比對錯誤訊息**——訊息內容依版本演進政策可能會變動，但例外類別是穩定的介面。

## 二、429 跟 529：一個是你的責任，一個不是

這兩個錯誤碼最容易被搞混，但成因完全相反：

**429 `rate_limit_error`**：你的組織撞到了 Day 13 講過的用量層級限制（RPM/ITPM/OTPM）。**這是你這邊可以預防的**——設計合理的請求節奏、善用 Day 9 的快取降低 ITPM 消耗，都能降低撞到它的機率。

**529 `overloaded_error`**：Anthropic 系統整體流量過大，暫時性過載。**這不是你的請求出了什麼問題**，純粹是服務端當下承載不了。

官方在 429 的說明裡還埋了一個容易被忽略的提醒：

> "In rare cases, if your organization has a sharp increase in usage, you might see 429 errors because of acceleration limits on the API. To avoid hitting acceleration limits, ramp up your traffic gradually and maintain consistent usage patterns."

**如果你的流量突然暴增（例如一次性批量上線某個新功能），就算沒有真的超過 Day 13 那張速率限制表上的數字，也可能因為「加速限制」被擋下 429。** 修法是**漸進式拉高流量**，而不是瞬間衝量——這對規劃新功能上線節奏是個實用的提醒。

## 三、SDK 內建的自動重試

好消息是你不用從零開始寫重試邏輯。官方 SDK 內建了自動重試機制：

> "The official SDKs automatically retry transient failures (such as connection errors, rate limits, and 5xx server errors) with exponential backoff, twice by default, honoring the `retry-after` header when present."

**連線錯誤、速率限制、5xx 伺服器錯誤，SDK 預設會用指數退避（exponential backoff）自動重試兩次，而且會尊重伺服器回傳的 `retry-after` 標頭**（Day 13 提過這個標頭，告訴你該等幾秒再試）。每個 SDK 都能設定最大重試次數，或完全關閉這個行為：

```python
# Python SDK：自訂重試次數
client = anthropic.Anthropic(max_retries=5)

# 單次請求覆寫
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "..."}],
).with_options(max_retries=0)  # 這次請求不自動重試
```

**多數情境下，SDK 的預設行為就夠用**——你不需要自己重新實作指數退避邏輯，除非你有特殊的重試策略需求。

## 四、請求大小上限

依端點而異的限制：

| 端點類型 | 最大請求大小 |
| :--- | :--- |
| Messages API | 32 MB |
| Token Counting API | 32 MB |
| Batch API | 256 MB |
| Files API | 500 MB |

超過上限會收到 413 `request_too_large`——**在直連 Claude API 的情境下，這個錯誤在請求送達 API 伺服器前，就會被 Cloudflare 攔下來**，官方特別註明這一點,代表如果你在除錯 413,不用往 API 應用層邏輯裡找,問題出在請求本身太大。

## 五、長請求：改用串流，或設置 TCP keep-alive

官方對長時間執行的請求給了明確建議：

> "Consider using the streaming Messages API or Message Batches API for long-running requests, especially those over 10 minutes."

**避免在不使用串流或批次的情況下設定很大的 `max_tokens`**——原因是有些網路會在連線閒置一段時間後直接斷線，導致請求失敗或逾時卻拿不到任何回應。Day 25 講的串流輸出，除了改善使用者體感，這裡是它的第二個實際用途：**持續有資料流動的連線，比較不會被判定為閒置而被中斷。**

如果你是直接串接 API（不透過 SDK），官方建議設定 **TCP socket keep-alive** 降低閒置連線逾時的影響；官方 SDK 已經預設幫非串流請求驗證「不會超過 10 分鐘逾時」，也內建設定了 TCP keep-alive。

## 六、Request ID：跟支援團隊溝通的關鍵

每個 API 回應都帶著一個獨一無二的 `request-id` 標頭，同樣的值也會出現在錯誤回應的 `request_id` 欄位裡。**聯繫官方支援時附上這個 ID，能大幅加快問題排查速度**——比起描述「大概幾點鐘、我送了一個什麼樣的請求」，一個精確的 request ID 能讓支援團隊直接定位到那一次請求的完整記錄。

## 七、常見的驗證錯誤：呼應前面幾天學過的規則

官方文件整理了幾個最常見的 400 驗證錯誤，剛好都是本系列前面提過的規則：

- **Prefill 不支援**：4.6 之後的模型不支援預填回應，錯誤訊息是「This model does not support assistant message prefill. The conversation must end with a user message.」——Day 17 提過，替代方案是 Structured Outputs 或 system prompt 指示。
- **Thinking 區塊被修改**：Day 15 提過，工具使用情境下，thinking 區塊要原封不動送回，錯誤訊息會標出問題區塊的位置（例如 `messages.1.content.0`）。
- **Extended/Adaptive thinking 不匹配**：Day 15、Day 16 完整講過的三句 400 錯誤原文，都收錄在這份文件裡，是同一組規則的權威來源。

**這是一個值得記住的巧合**：錯誤處理文件裡列出的常見錯誤，幾乎都能對應回本系列前面某一天已經講過的原理。**看到陌生的錯誤訊息時，先想這個錯誤在講的規則你是不是已經學過**，而不是急著上網搜尋——很可能答案就在你已經讀過的內容裡。

## 八、冪等性：Messages API 沒有內建的重試保護機制

這是正式環境設計裡最容易被忽略的一塊。**跟某些強調冪等性的 API（例如金流服務常見的 idempotency key 機制）不同，Claude 的 Messages API 目前沒有文件記載的冪等鍵參數**——每一次成功的請求都會被視為一次全新的、獨立的呼叫。

這件事在**代理式工作流（agentic loop）**裡特別需要小心：如果你的重試邏輯是「請求逾時就整段重送」，而這次請求裡 Claude 呼叫了工具、工具已經執行了有副作用的操作（例如寄出一封信、寫入一筆資料庫紀錄），**盲目重送整個請求可能導致同樣的副作用被觸發兩次**——因為 API 端不會替你記得「這個請求我是不是已經處理過一次了」。

實務上的因應原則：

- **只在確定沒有收到任何回應（純網路層失敗）時重試整個請求**，如果你已經收到部分串流內容或工具呼叫結果，先確認這些副作用是否已經發生，不要單純假設「沒收到最終回應就等於什麼都沒發生」。
- **在你自己的應用層設計去重機制**，例如在有副作用的工具函式裡自己加一層「這個操作是否已經執行過」的檢查，而不是依賴 API 本身的保護。
- **Day 10 的 Batch API 提供了另一種思路**——每筆請求都有你自己指定的 `custom_id`，天生比較適合需要追蹤「這筆到底有沒有處理過」的場景。

## Before / After：一個會重複寄信的重試邏輯

**❌ Before：逾時就整段重送，不管副作用**

```python
def call_with_retry(messages, tools):
    for attempt in range(3):
        try:
            return client.messages.create(
                model="claude-opus-5", max_tokens=1024,
                messages=messages, tools=tools,
            )
        except anthropic.APITimeoutError:
            continue  # 逾時直接整段重來，即使工具可能已經執行過寄信動作
```

**✅ After：工具函式自己做去重，而不是依賴請求層級的重試安全**

```python
def send_email_tool(to, subject, body, request_context_id):
    if already_sent(request_context_id):  # 應用層自己記錄，不依賴 API 冪等性
        return {"status": "already_sent"}
    result = actually_send_email(to, subject, body)
    mark_as_sent(request_context_id)
    return result

def call_with_retry(messages, tools):
    for attempt in range(3):
        try:
            return client.messages.create(
                model="claude-opus-5", max_tokens=1024,
                messages=messages, tools=tools,
            )
        except anthropic.APITimeoutError:
            continue
```

> Before 版本假設「重試永遠安全」，這個假設在沒有副作用的問答情境下成立，但在會觸發真實動作的工具呼叫情境下並不成立。After 版本把「這個操作到底有沒有真的發生過」的判斷責任，放在你自己控制的應用層——**因為 API 本身沒有替你做這件事。**

## 本篇自我挑戰

- **今日挑戰**：檢查你目前的重試邏輯，如果請求裡包含會觸發真實副作用的工具呼叫（寄信、寫資料庫、呼叫金流），確認重試時有沒有可能讓同樣的動作被觸發兩次。如果目前完全靠 SDK 預設的自動重試，確認你的工具函式是否已經對重複執行是安全的（idempotent）。

- **反思**：「重試」聽起來像是一個純技術問題，但今天講的內容顯示它其實牽涉到業務邏輯——**什麼樣的操作可以安全地重來一次，什麼樣的不行。** 這個判斷沒辦法完全交給框架或 SDK 處理，需要你對自己的業務有清楚的理解。你目前的系統裡，有沒有已經在用「自動重試」、但其實從沒認真想過重試安不安全的地方？

## 總結

第四階段收在這裡。今天講的是把 Claude API 真正放進正式環境需要的最後一塊拼圖：**標準化的錯誤格式跟型別化例外**、**429（你的責任）跟 529（不是你的責任）的區分**、**SDK 內建的指數退避重試（預設兩次，尊重 `retry-after`）**、**長請求該用串流或批次來避免閒置逾時**，以及最容易被忽略的一點——**Messages API 沒有內建冪等鍵機制，重試安全性要靠你自己的應用層設計**，尤其是在有工具呼叫、有真實副作用的代理式工作流裡。

**本日關鍵字回顧**

- **標準錯誤格式**：`{type, error: {type, message}, request_id}`，SDK 提供型別化例外，建議依類別攔截而非字串比對。
- **429 vs 529**：前者是你的組織撞到速率限制（可預防），後者是 Anthropic 端暫時過載（非你造成）。
- **加速限制（Acceleration limits）**：流量急遽增加可能觸發 429，即使未超過標準速率限制表，修法是漸進式拉高流量。
- **SDK 自動重試**：預設對暫時性錯誤指數退避重試兩次，尊重 `retry-after`，可透過 `max_retries` 調整。
- **無內建冪等機制**：Messages API 沒有官方冪等鍵參數，重試安全性需在應用層（尤其是工具呼叫的副作用）自行把關。

第四階段「開發者實戰」到此結束。從 Claude Code、MCP、Messages API、Streaming 到今天的錯誤處理，這六天把前面三個階段的知識，全部串進真實可以上線的系統裡。

明天進入最後一個階段——**底層邏輯與架構思維**。第一站是模型分流：別再一支模型從頭用到尾，怎麼依任務難度動態選模型。

**Day 27，模型分流是什麼？**
