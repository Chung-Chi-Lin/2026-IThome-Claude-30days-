# 【Day 17】Claude Prompt 寫法教學：官方最佳實踐的骨架

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

過去三天講了 `effort`、adaptive thinking、還有怎麼排查回答變淺。這些都是「調參數」層級的技巧。但參數調得再對，如果 prompt 本身寫得含糊，一樣拿不到好答案——**thinking 再深，也沒辦法猜中你心裡沒說出來的需求。**

今天要往回退一步，講一個更根本、也更長期有效的東西：**一份好 prompt 的骨架長什麼樣子。** 這套骨架不會因為模型換代而失效，因為它解決的是溝通問題，不是參數問題。

官方對「什麼時候該做 prompt engineering」有個很實際的判準：如果你的問題可以透過換模型解決（例如純粹是延遲或成本問題），那該做的是選型，不是死磕 prompt。但如果問題出在「模型没有理解我要什麼」，今天這套骨架就是你要的。

> 本篇的技巧與範例，於 **2026 年 8 月 14 日**對照 [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) 官方文件查證。

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

## 一、動筆前先做一次測試：黃金法則

官方給了一個很好用的自我檢查法，我稱它「黃金法則」：

> "Show your prompt to a colleague with minimal context on the task and ask them to follow it. If they'd be confused, Claude will be too."

**把你的 prompt 拿給一個對這件任務沒什麼背景的同事看，請他照著做。如果他會困惑，Claude 也會。**

官方的比喻是：把 Claude 想成一位**聰明但剛到職、還不熟悉你們公司規範和工作流程的新人**。你解釋得越精確，結果越好。這句話值得貼在螢幕旁——它比任何具體技巧都更根本，因為多數「Claude 答錯了」的案例，回頭一查都是「我以為講清楚了，其實沒有」。

## 二、骨架：七個區塊，由外而內排列

把官方分散在各處的技巧，組成一個可以套用在多數情境的骨架。順序是有意義的——由「穩定不變」排到「每次都變」，這同時也呼應 Day 9 提過的快取策略。

**① 角色設定（system prompt）**

```text
system: "You are a helpful coding assistant specializing in Python."
```

官方原文：「Setting a role in the system prompt focuses Claude's behavior and tone for your use case. Even a single sentence makes a difference.」——**一句話就有感**，這是 Day 19 的完整主題，這裡先當骨架的第一塊。

**② 情境與動機**

不要只給指令，順便說明「為什麼」：

```text
你的回答會被文字轉語音引擎朗讀出來，所以絕對不要使用刪節號（…），
因為 TTS 引擎不知道怎麼唸這個符號。
```

官方的觀察是：「Claude is smart enough to generalize from the explanation.」——**給了原因，Claude 能舉一反三**，比單純下禁止指令更有效，也更能應付你沒預想到的邊界情況。

**③ 明確直接的指令**

具體到「講清楚輸出格式與限制」的程度，並且用**條列步驟**表達有順序性的要求：

```text
❌ 建一個分析儀表板
✅ 建一個分析儀表板。盡可能包含相關的功能與互動，
   不要只做基本款，做一個功能完整的版本。
```

**④ 範例（few-shot / multishot）**

官方稱這是最可靠的技巧之一：「Examples are one of the most reliable ways to steer Claude's output format, tone, and structure.」建議放 **3 到 5 個**，而且要：

- **貼近真實情境**（Relevant）
- **涵蓋邊界案例、彼此有差異**（Diverse，避免模型學到不該學的規律）
- **用 `<example>` 標籤包起來**（多個範例包在 `<examples>` 裡，讓 Claude 分得清「這是範例」還是「這是指令」）

**⑤ 輸入資料（XML 結構化）**

把「指令」「情境」「輸入」分別包在對應的標籤裡（Day 18 會完整展開），例如 `<instructions>`、`<context>`、`<input>`。

**⑥ 輸出格式規範**

官方給了一個很實用的原則：**告訴 Claude 該做什麼，而不是不該做什麼**：

```text
❌ 「不要用 markdown」
✅ 「你的回答請用流暢的文章段落組成」
```

如果還是控制不好格式，可以進一步用 **XML 格式指示符**：

```text
請把文章正文寫在 <smoothly_flowing_prose_paragraphs> 標籤裡。
```

**⑦（適用長文件時）把長內容放最前面**

如果你的 prompt 涉及 20k token 以上的長文件，官方測試發現一個具體數字：

> "Queries at the end can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs."

**把長文件放在 prompt 最前面，問題放在最後**，在複雜的多文件情境下，回答品質最多可以提升到 30%。這跟 Day 9 快取策略的「穩定內容在前」剛好是同一個排列方向，一次改動兩邊都受益。

## 三、把七塊拼起來：一個完整範例

```python
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=2048,
    system="你是一位資深的資料隱私顧問，專精 GDPR 與 CCPA 合規審查。",  # ①角色
    messages=[{
        "role": "user",
        "content": """
你的審查結果會直接交給法務團隊做決策，所以每個結論都需要有明確的條文依據，
不能只憑印象判斷。  <!-- ②情境與動機 -->

<instructions>
1. 先從文件中摘錄與 GDPR、CCPA 合規性最相關的原文引句
2. 依據摘錄的引句，逐條分析合規風險
3. 用條列式輸出，每點不超過兩句話
</instructions>  <!-- ③明確指令，含順序步驟 -->

<examples>
<example>
輸入：「我們會在使用者同意後保留資料 90 天」
輸出：符合 GDPR 第 6 條同意原則，但建議明確標示 90 天的計算起點。
</example>
</examples>  <!-- ④範例 -->

<policy_document>
{{POLICY_TEXT}}
</policy_document>  <!-- ⑤輸入資料，XML包起來 -->

請用條列式輸出分析結果，每點附上引用的原文段落。  <!-- ⑥輸出格式規範 -->
"""
    }],
)
```

這個例子看起來比隨口一問長很多，但**每一段都有明確的功能**，不是堆砌——這正是骨架的價值：讓你知道「多寫的每一句話在幫什麼忙」，而不是憑感覺塞字數。

## 四、不是每個任務都需要整套骨架

官方的態度很務實：這套骨架是**工具箱，不是每次都要用滿**。簡單、一次性的問題，直接問就好；骨架的價值在**會被重複使用、或是輸出格式要求嚴格**的 prompt 上——例如你的應用程式會反覆呼叫同一個 prompt 模板處理不同輸入，這時候骨架前面幾塊（角色、指令、範例）值得花時間打磨，因為打磨一次，之後每一次呼叫都受益。

## 五、任務太複雜，一次 prompt 打不完怎麼辦：拆成鏈

Day 15 提過，adaptive thinking 加上代理式協作，已經能讓 Claude 內部處理大部分的多步驟推理，不一定需要你手動拆解。但官方也提到一種情境仍然值得手動拆——**當你需要檢視中間產出、或是要強制走一套特定流程時**：

> "The most common chaining pattern is self-correction: generate a draft → have Claude review it against criteria → have Claude refine based on the review. Each step is a separate API call so you can log, evaluate, or branch at any point."

**草稿 → 依標準審查 → 依審查結果修正**，拆成三次獨立的 API 呼叫，你就能在每個階段記錄、評估，甚至視情況分支處理——這對品質要求高、需要留審計軌跡的任務特別有用，但代價是三次呼叫的成本，權衡點跟 Day 10 批次處理的邏輯類似：**該不該拆，看你需不需要中間可視性，而不是預設拆比較保險。**

## 六、反模式：把骨架當成必填表單

骨架最常被誤用的方式，是把七個區塊當成**每次都要填滿的表單**，即使任務很簡單也硬湊。這會帶來兩個問題：**多餘的內容本身要花 token**（呼應 Day 2 的計價結構），以及**過度結構化反而可能讓簡單任務變得不自然**——一個「幫我把這句話翻成英文」的請求，套上完整的角色設定、範例、XML 標籤，不會讓翻譯更準，只是把 prompt 變長。

判斷該不該用滿骨架的簡單原則：**這個 prompt 會不會被重複使用？輸出格式的一致性重不重要？** 兩個問題只要有一個答案是「是」，骨架就值得投資；兩個都是「否」，直接問通常就夠了。

## Before / After：從隨口問到骨架化

**❌ Before：一句話打發**

```text
幫我看看這份隱私權政策符不符合規定
```

**✅ After：套用骨架**

```text
system: 你是一位資深的資料隱私顧問，專精 GDPR 與 CCPA 合規審查。

user:
你的審查結果會交給法務團隊做決策，需要明確條文依據。

<instructions>
1. 先摘錄與合規性最相關的原文引句
2. 依摘錄逐條分析風險
3. 條列輸出，每點不超過兩句
</instructions>

<policy_document>
{{POLICY_TEXT}}
</policy_document>
```

> Before 版本得到的答案品質完全取決於模型自己猜你要多細、要什麼格式、法務團隊看不看得懂。After 版本把「新人需要知道的背景」一次講清楚——這不是多此一舉，是把原本要靠好幾輪來回修正才能對齊的期望，一次寫進第一個請求裡，省下的是後面反覆調整的時間與 token。

## 本篇自我挑戰

- **今日挑戰**：挑一個你目前用得還算順手、但偶爾會出錯的 prompt，套上今天的黃金法則——找一位不熟悉這個任務的朋友或同事念一遍，看他會不會卡在某個地方。那個卡住的地方，往往就是 Claude 也會誤解的地方。

- **反思**：官方把 Claude 比喻成「聰明但剛到職的新人」，這個比喻的重點其實是在提醒我們——**問題通常不在 Claude 不夠聰明，是我們沒把「新人該知道的事」講清楚。** 你在跟真正的新同事交接工作時，會不會也犯同樣的毛病：以為自己講清楚了，其實漏了很多預設的背景知識？

## 總結

今天的骨架不是新發明的技巧，是把 Day 1 到 Day 16 分散提過的原則，收攏成一個可以重複套用的順序：**角色 → 情境動機 → 明確指令 → 範例 → 輸入資料 → 輸出格式 →（長文件時）內容前置。** 每一塊都對應官方一句具體的實測建議，不是憑空排的順序。

黃金法則值得抄下來反覆用：**拿給不懂背景的人看，他會不會困惑。** 這個檢查比任何單一技巧都更能抓出 prompt 裡藏著的模糊地帶。

**本日關鍵字回顧**

- **黃金法則**：把 prompt 給沒有背景的同事看，他會困惑，Claude 也會困惑。
- **多告知「為什麼」**：解釋指令背後的動機，能讓 Claude 舉一反三處理你沒預想到的情況。
- **告訴它該做什麼，而非不該做什麼**：正向指令比負向禁止更容易被準確執行。
- **長文件前置**：20k token 以上的文件放在 prompt 最前面、問題放最後，實測最多可提升 30% 回答品質。
- **Prompt 鏈（Chaining）**：草稿→審查→修正的三段式拆解，用於需要中間可視性或強制流程的複雜任務。

明天把骨架裡的第⑤塊——XML 標籤——單獨拉出來講透：為什麼 Claude 對 XML 特別敏感、怎麼設計一致的標籤系統。

**Day 18，用 XML 標籤讓輸出更穩定。**
