# 【Day 12】對話越長越燒錢？Claude 長對話的成本陷阱與解法

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

Day 8 技巧三提過一句話：「每一輪 Claude 都要重新讀一次前面所有的對話歷史，才能生成這一輪的回答。」今天把這件事的成本結構、以及官方提供的兩種解法，完整攤開來講。

先建立一個直覺：**多輪對話不是「累加式」收費，是「重算式」收費。** 第 20 輪的請求，不是只算第 20 輪新增的內容，而是要把第 1 輪到第 19 輪**整段歷史**都當作 input 重新送一次，再加上第 20 輪的新內容。對話越長，這個「重付」的基數就越大。

Day 9 講的快取能緩解這個問題的「錢」的部分——重複的歷史用 0.1 倍價讀取。但快取解決不了另一個問題：**歷史本身還是佔著 context window 的空間**，而且還是計入 ITPM（每分鐘輸入 token）額度。今天要講的，正是快取管不到的那一塊。

> 本篇 compaction 與 context editing 的規格，於 **2026 年 8 月 14 日**對照 [Compaction](https://platform.claude.com/docs/en/build-with-claude/compaction) 與 [Context editing](https://platform.claude.com/docs/en/build-with-claude/context-editing) 官方文件查證，兩者皆為 beta 功能。

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

## 一、成本累積的形狀：不是線性，是三角形

想像一個對話進行了 20 輪。第 1 輪的 input 很小，第 20 輪的 input 要扛起前面 19 輪的全部內容。把每一輪的 input token 畫成長條圖，形狀是一路往上爬的樓梯——**總成本是這 20 個長條的總和，而不是最後一輪的大小。**

這對兩種情境特別致命：**長時間執行的代理任務**（不斷呼叫工具、產生工具結果，每個工具結果都疊進歷史）、以及**客服或陪伴類的長聊天**（使用者可能一路聊幾十輪不換話題）。兩者的共通點是：**歷史成長的速度，比你直覺以為的快很多。**

官方在 compaction 文件裡把這個問題講得很直接——長對話與代理任務會讓 context 持續逼近上限，**伺服器端摘要是目前建議的主要因應策略**。

## 二、解法一：Compaction，讓伺服器自動幫你摘要

Compaction 是**伺服器端**的功能：當對話的 input token 接近你設定的門檻，系統會自動把較早的內容摘要成一個精簡的 `compaction` 區塊，取代原本那一大段歷史，讓對話能繼續往下走而不會撞到 context 上限。

啟用方式是在請求裡加上 `context_management.edits`：

```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-opus-5",
    max_tokens=4096,
    messages=messages,
    context_management={
        "edits": [
            {
                "type": "compact_20260112",
                "trigger": {"type": "input_tokens", "value": 150000},
            }
        ]
    },
)
```

幾個實務上要注意的細節：

- **`trigger` 預設值是 150,000 input tokens**，可以自訂，但**最低只能設到 50,000**——不能設得比這個更靈敏。
- **支援的模型有限**：`claude-fable-5`、`claude-mythos-5` / `mythos-preview`、`claude-opus-5`、`opus-4.8`/`4.7`/`4.6`、`claude-sonnet-5`、`sonnet-4.6`。如果你還在用更早的模型，這個功能用不了。
- **目前是 beta**，要帶 `betas=["compact-2026-01-12"]`。
- 可以用 `instructions` 參數自訂摘要的重點（例如「優先保留程式碼片段與技術決策」），也可以設 `pause_after_compaction` 讓摘要完成後暫停，方便你在繼續前檢查或插入額外內容。

**計價上有個容易漏算的地方**：Compaction 本身需要**額外的一次取樣**（要先讓模型讀完舊內容、生成摘要），這筆成本會被記在 `usage.iterations` 陣列裡，**不包含在頂層的 `input_tokens` / `output_tokens` 裡**：

```json
{
  "usage": {
    "input_tokens": 23000,
    "output_tokens": 1000,
    "iterations": [
      { "type": "compaction", "input_tokens": 180000, "output_tokens": 3500 },
      { "type": "message", "input_tokens": 23000, "output_tokens": 1000 }
    ]
  }
}
```

官方原文提醒：**要算出這次請求實際被計費的總 token 數，必須把 `usage.iterations` 陣列裡的每一筆都加總**，只看頂層數字會低估。換句話說，compaction 省的是「之後每一輪」的重付成本，但**觸發摘要的那一輪，本身要多付一次摘要的費用**——這是用一次性成本，換長期的下降曲線。

## 三、解法二：Context Editing，更細的手動控制

如果你不想讓系統自動摘要（可能你需要更精準地控制哪些內容被留下），**Context Editing** 提供兩種更細緻的清除策略，一樣是伺服器端處理，客戶端仍保留完整、未修改的對話歷史。

**① Tool Result Clearing（`clear_tool_uses_20250919`）**：針對大量使用工具的代理工作流，自動清掉最舊的工具執行結果，換成佔位文字：

```python
context_management={
    "edits": [
        {
            "type": "clear_tool_uses_20250919",
            "trigger": {"type": "input_tokens", "value": 30000},
            "keep": {"type": "tool_uses", "value": 3},
            "exclude_tools": ["web_search"],
        }
    ]
}
```

預設觸發門檻是 **100,000 input tokens**，預設保留最近 **3 次**工具使用。可以指定某些工具（例如 `web_search`）不參與清除。

**② Thinking Block Clearing（`clear_thinking_20251015`）**：控制要保留幾輪先前的思考內容——保留太多會佔用 context 空間，清得太乾淨又可能影響快取與模型的連貫性：

```python
context_management={
    "edits": [
        {
            "type": "clear_thinking_20251015",
            "keep": {"type": "thinking_turns", "value": 2},
        }
    ]
}
```

不同模型的預設行為不同：**Opus 4.5 之後、Sonnet 4.6 之後的模型預設保留所有先前的思考；更早的模型與所有 Haiku 系列則只保留最後一輪。**（這跟 Day 5 提過的「thinking 區塊是否保留在 context 裡」是同一組規則。）

兩種策略可以合併使用，但**官方要求 `clear_thinking_20251015` 必須排在 `clear_tool_uses_20250919` 之前**。目前一樣是 beta，要帶 `anthropic-beta: context-management-2025-06-27`。

## 四、該用哪一個？官方給了明確的優先順序

兩份文件都寫了幾乎一樣的話：

> "For most use cases, server-side compaction is the primary strategy for managing context in long-running conversations and agentic workflows. [Context editing 的策略] are useful for specific scenarios where you need more fine-grained control over what content is cleared."

翻成白話：**先用 Compaction，這是官方的預設建議。** 只有當你需要更精準地控制「清什麼、留什麼」——例如你知道某些工具結果就是沒用、某些思考過程一定要留著——才切換到 Context Editing 的細粒度控制。

兩者也可以理解成互補而非互斥：Compaction 處理「整段歷史該不該摘要」的大方向，Context Editing 處理「特定類型的內容該不該被清掉」的細節。

## 五、跟自己土法煉鋼比，官方方案省在哪

在 Compaction 變成官方功能之前，很多團隊（包含我自己）的做法是**自己動手**：對話長到一個門檻，就另外呼叫一次 Claude，請它把前面的對話摘要成一段文字，再拿這段摘要取代原始歷史，塞回下一輪的 messages 裡。

這個土法煉鋼的做法能達到類似的效果，但有三個地方比官方方案吃虧：

**① 觸發時機要自己算。** 你得自己追蹤目前 input token 數，決定什麼時候該摘要——這正是 Day 7 教的 `count_tokens` 派得上用場的地方，但終究是額外的程式碼與額外的一次 API 呼叫（用來檢查，不是用來摘要）。

**② 摘要品質沒有跟對話上下文整合。** 自己另外呼叫一次「請幫我摘要」，跟官方在同一個請求脈絡裡生成的 compaction 區塊，兩者對「這段對話裡什麼重要」的判斷基礎不完全一樣——官方方案是在原生的對話狀態裡生成摘要，語境更完整。

**③ 快取會被打散。** 自己動手插入摘要，等於改動了 messages 陣列的內容，這會讓 Day 9 提過的快取前綴整個對不上，之後的每一輪都要重新寫入快取。官方的 compaction 機制則會在摘要區塊上套用快取設計，把這個副作用降到最低。

土法煉鋼不是不能用——如果你需要完全自訂的摘要邏輯（例如只保留特定格式的資訊），Context Editing 或自訂 `instructions` 的 Compaction 仍然是更貼近的選擇。但如果你只是想「讓長對話不要撞牆」，官方的伺服器端方案能省下不少自己維護的邊角案例。

## 六、Before / After：一個跑很久的代理任務

**❌ Before：什麼都不做，讓對話自然變長**

```python
messages = []
for step in agent_loop_steps:  # 可能跑幾十輪工具呼叫
    messages.append({"role": "user", "content": step.input})
    response = client.messages.create(
        model="claude-opus-5",
        max_tokens=4096,
        messages=messages,        # 每一輪都帶著完整歷史
        tools=my_tools,
    )
    messages.append({"role": "assistant", "content": response.content})
    # 到了第 30 輪，input 可能已經逼近甚至撞上 context 上限
```

**✅ After：加上 compaction，讓伺服器自動控制歷史大小**

```python
for step in agent_loop_steps:
    messages.append({"role": "user", "content": step.input})
    response = client.beta.messages.create(
        betas=["compact-2026-01-12"],
        model="claude-opus-5",
        max_tokens=4096,
        messages=messages,
        tools=my_tools,
        context_management={
            "edits": [{"type": "compact_20260112", "trigger": {"type": "input_tokens", "value": 100000}}]
        },
    )
    messages.append({"role": "assistant", "content": response.content})
    # 逼近 10 萬 token 時自動摘要，對話可以繼續跑下去而不會卡在上限
```

> Before 版本會在某一輪突然撞牆——不是報錯就是 `stop_reason` 變成 `model_context_window_exceeded`（Day 5 提過的那個靜默截斷）。After 版本用**主動觸發的摘要**取代「被動撞牆」，代價是觸發摘要那一輪要多付一次額外的取樣成本，換到的是整個代理任務能穩定跑更長。

## 本篇自我挑戰

- **今日挑戰**：如果你有任何會員跑超過 10 輪以上的對話或代理流程，去查一下它目前的 `input_tokens` 是怎麼隨輪數成長的——如果你手邊沒有現成的記錄，用今天的程式碼骨架跑幾輪，實際畫出那條成長曲線，感受一下它有多陡。

- **反思**：Compaction 的取捨是「犧牲一點精確度（摘要不等於原文），換取對話能繼續進行」。這跟人類開會很像——會議記錄從來不是逐字稿，而是摘要,但摘要漏掉的細節有時候很關鍵。你會怎麼判斷一個對話「適合被摘要」還是「必須保留完整原文」？

## 總結

長對話的成本陷阱來自一個簡單但容易被忽略的事實：**每一輪都要重付前面所有的歷史**，成本累積的形狀是三角形，不是線性。Day 9 的快取能讓重複的歷史用 0.1 倍價計費,但管不到 context 空間本身還是被佔用這件事。

官方給了明確的優先順序：**先用 Compaction**，讓伺服器自動在逼近門檻時摘要舊內容，這是大多數長對話與代理任務的預設解法；**需要更精細控制時再上 Context Editing**，用 Tool Result Clearing 和 Thinking Block Clearing 處理特定類型的內容清除。兩者都是 beta 功能，都是伺服器端處理，客戶端始終保留完整歷史。

記得 compaction 觸發時會多一筆摘要成本，要看 `usage.iterations` 才能算出真實總花費——這是唯一容易漏算的地方。

**本日關鍵字回顧**

- **成本三角形**：多輪對話每一輪都重付完整歷史，總成本是逐輪疊加，非線性成長。
- **Compaction（`compact_20260112`）**：伺服器端自動摘要，beta 功能，預設觸發門檻 150,000 input tokens，最低可設 50,000。
- **`usage.iterations`**：計算 compaction 請求真實總花費時必須加總的欄位，頂層 `input_tokens`/`output_tokens` 不含摘要成本。
- **Context Editing**：`clear_tool_uses_20250919`（清除舊工具結果）與 `clear_thinking_20251015`（控制思考區塊保留），beta 功能，用於更細粒度的控制場景。
- **官方優先順序**：多數情境優先用 server-side compaction，需要精細控制才切換到 context editing。

明天把成本這件事收尾——**光靠事後看帳單太晚了，怎麼在花超之前就先發現？** 從 usage 欄位到 Console 儀表板，把帳單變成一個可以主動監控的系統。

**Day 13，用量監控與預警機制。**
