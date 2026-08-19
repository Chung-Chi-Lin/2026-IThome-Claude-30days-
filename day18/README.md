# 【Day 18】用 XML 標籤讓 Claude 輸出更穩定（結構化輸出教學）

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

Day 17 骨架裡的第⑤塊，我只帶過一句「把輸入資料用 XML 包起來」。今天把它單獨拉出來講——因為這是少數幾個「花小力氣、換大穩定性」的技巧，尤其你的 prompt 一旦混雜了指令、背景資料、範例、變數輸入，XML 標籤幾乎是必備的。

先講清楚一件事：**XML 標籤的作用是幫 Claude 消除歧義，不是裝飾。** 官方的說法很直接：「XML tags help Claude parse complex prompts unambiguously, especially when your prompt mixes instructions, context, examples, and variable inputs.」——當你的 prompt 只有一句話，XML 幫不上什麼忙；但當你的 prompt 開始有好幾個部分混在一起，XML 標籤能明確告訴模型「這一段是指令」「這一段是背景」「這一段是你要處理的內容」，減少模型自己猜的空間。

> 本篇 XML 標籤最佳實踐與範例，於 **2026 年 8 月 14 日**對照 [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) 官方文件查證。

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

## 一、兩條最佳實踐：一致命名、依層級巢狀

官方給的最佳實踐濃縮成兩句話：

**① 全程使用一致、描述性的標籤名稱。** 不要這次用 `<context>`、下次用 `<background>` 指同一種東西——標籤名稱本身就是給模型（以及未來的你）的說明文件，命名要讓人一看就懂裡面裝的是什麼。

**② 有層級關係的內容要巢狀。** 官方範例：多份文件包在 `<documents>` 裡，每一份個別文件再包在 `<document index="n">` 裡——巢狀結構直接反映了資料本身的層級關係，不需要額外的文字說明「這是第幾份文件」。

## 二、常見標籤與它們的用途

官方文件裡反覆出現的標籤組合，可以當作起手式直接套用：

| 標籤 | 用途 |
| :--- | :--- |
| `<instructions>` | 明確的任務指令 |
| `<context>` | 背景資訊、動機說明 |
| `<example>` / `<examples>` | 單一範例／多個範例的容器，讓 Claude 分得清「這是範例」跟「這是要處理的內容」 |
| `<document>` / `<document_content>` / `<source>` | 文件本體、文件內文、文件出處，用於多文件情境 |
| `<quotes>` | 從長文件裡摘錄的原文引句（Day 20 會延伸這個用法到降低幻覺） |
| `<input>` | 使用者這次要處理的實際輸入內容 |

這份清單不是規定死的語彙表，你可以依任務自訂標籤名稱——重點是**一致**跟**望文生義**，不是一定要用官方範例裡出現過的名字。

## 三、多文件情境的完整結構

當 prompt 裡有不只一份文件時，官方建議的結構長這樣：

```xml
<documents>
  <document index="1">
    <source>annual_report_2023.pdf</source>
    <document_content>
      {{ANNUAL_REPORT}}
    </document_content>
  </document>
  <document index="2">
    <source>competitor_analysis_q2.xlsx</source>
    <document_content>
      {{COMPETITOR_ANALYSIS}}
    </document_content>
  </document>
</documents>

分析年報與競爭對手分析，找出策略優勢並建議第三季重點方向。
```

這個結構做了三件事：**用 `index` 屬性明確排序**、**用 `source` 標記每份文件的出處**（讓 Claude 在回答時能明確引用是哪一份文件說的）、**用 `document_content` 把實際內文跟中繼資料分開**。對照 Day 17 提過的「長文件放最前面、問題放最後」，這個結構完美搭配——文件本體整段包在標籤裡放前面，最後一句才是真正的問題。

## 四、進階用法：用 XML 標籤強制模型先引用再回答

這個技巧把 XML 標籤從「結構化輸入」延伸到「結構化推理過程」，官方在醫療情境的範例特別清楚：

```xml
你是一位 AI 醫師助理，任務是協助醫生診斷病人可能的疾病。

<documents>
  <document index="1">
    <source>patient_symptoms.txt</source>
    <document_content>{{PATIENT_SYMPTOMS}}</document_content>
  </document>
  <document index="2">
    <source>patient_records.txt</source>
    <document_content>{{PATIENT_RECORDS}}</document_content>
  </document>
</documents>

先從病歷與回診紀錄中找出與病人症狀診斷相關的引句，放進 <quotes> 標籤。
接著根據這些引句，列出所有有助於醫生診斷的資訊，放進 <info> 標籤。
```

這裡的關鍵是**先要求 Claude 把引用的原文放進 `<quotes>`，再根據引用內容產生結論放進 `<info>`**——這個「先摘錄、再推理」的順序，讓最終答案有跡可循，你可以直接檢查 `<quotes>` 裡的內容是不是真的存在於原文，而不是憑空冒出來的。這正是 Day 20 要完整展開的降低幻覺技巧的雛型，XML 標籤在這裡扮演的是「把推理過程攤開來檢查」的角色。

## 五、控制輸出格式：正向指示 + XML 格式指示符

Day 17 提過一個原則——告訴 Claude 該做什麼，而非不該做什麼。XML 標籤是落實這個原則最直接的工具：

```text
❌ 「不要用條列式」
✅ 「請把正文寫在 <smoothly_flowing_prose_paragraphs> 標籤裡」
```

用一個具體、望文生義的標籤名稱去**定義**你要的格式，比條列一串「不要做什麼」更容易讓模型抓到重點。如果你需要更嚴格的格式控制（例如要求正文完全不使用 markdown），官方建議把整套規則包進一個標籤裡集中說明，而不是散落在 prompt 各處：

```xml
<avoid_excessive_markdown_and_bullet_points>
撰寫報告、文件、技術說明、分析或任何長篇內容時，請用完整的段落與句子撰寫，
使用一般段落換行來組織內容，markdown 僅保留給行內程式碼、程式碼區塊與簡單標題。
除非真的在列舉離散項目、或使用者明確要求列表，否則不要使用條列式或編號清單。
</avoid_excessive_markdown_and_bullet_points>
```

把規則集中包在一個標籤裡的好處是：**這段規則可以被重複使用**——存成一個固定模板，套用在任何「不想要條列式轟炸」的情境，而且因為內容穩定不變，也符合 Day 9 提過的快取友善結構。

## 六、XML 標籤 vs Structured Outputs：需要保證有效的 JSON 時,換工具

這是一個容易被混淆的地方：**XML 標籤是「提示」，不是「強制」。** 官方原文區分得很清楚——XML 標籤能大幅提高 Claude 遵守格式的機率，但終究是自然語言層級的引導，模型仍然有可能不完全照做。

如果你的場景是**程式要直接解析輸出**（例如丟進 `JSON.parse()`），而任何格式偏差都會讓程式出錯，這時候該用的是官方另一個功能——**Structured Outputs**（`output_config.format`），它透過**約束解碼**在生成過程中就強制輸出符合你給定的 JSON schema，不會產生格式錯誤：

```python
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "從這段文字擷取聯絡資訊"}],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "email": {"type": "string"},
                },
                "required": ["name", "email"],
            },
        }
    },
)
```

**分工原則很清楚**：XML 標籤適合用在「引導模型的思考結構與可讀性輸出」（例如今天前五節的用法），Structured Outputs 適合用在「程式必須拿到保證有效的 JSON，不能有任何例外」。兩者甚至可以搭配工具使用一起用——工具參數用 `strict` 驗證，回應本身也套用 schema，讓整個流程的輸出都有格式保證。

## Before / After：從一段話變成有結構的請求

**❌ Before：所有東西攪在一起**

```text
你是客服助理，這是我們的退換貨政策：{{POLICY}}，
還有這是客戶的訊息：{{MESSAGE}}，範例像是這樣：
如果客戶說想退貨超過30天，要說明超過期限但可以協助其他方案，
幫我回覆這位客戶，不要太生硬，語氣客氣一點，用條列會比較清楚
```

**✅ After：用 XML 拆開，各司其職**

```xml
system: 你是一位語氣親切、專業的客服助理。

<policy>
{{POLICY}}
</policy>

<example>
輸入：客戶想退貨，但已超過 30 天期限
輸出：說明已超過退換貨期限，同時主動提供其他可行方案（如換貨或折扣券）
</example>

<customer_message>
{{MESSAGE}}
</customer_message>

請依政策回覆客戶，語氣親切但專業，用條列列出可行方案。
```

> Before 版本把政策、範例、客戶訊息、格式要求全部黏在一句話裡，模型得自己猜哪一段是在講規則、哪一段是要處理的內容。After 版本用標籤把每一種內容各自隔開，模型不需要用猜的——**這不是讓 prompt 變長的裝飾，是把原本混在一起、容易被誤解的四種資訊，分成四個明確的區塊。**

## 本篇自我挑戰

- **今日挑戰**：找一個你目前寫的、內容混雜（指令 + 背景 + 資料）的長 prompt，套用今天的標籤系統重新拆分一次，比較拆分前後 Claude 遵循格式要求的穩定度。

- **反思**：XML 標籤有效的根本原因，是它把「模型要用猜的」變成「模型不用猜」。這個原則其實超越了跟 AI 溝通這件事——你在寫技術文件、寫需求規格、跟團隊交接工作時，有沒有類似「把不同性質的資訊清楚分區」就能大幅減少誤解的經驗？

## 總結

XML 標籤不是特殊咒語，它做的事情很單純：**把 prompt 裡混在一起的不同性質內容，用標籤明確分開**，減少模型需要自己猜測的空間。今天的兩條最佳實踐——**一致命名、依層級巢狀**——加上一份可以直接套用的常見標籤清單，應該足夠應付大多數情境。

記得一個分工原則：**XML 標籤是強力的引導，但不是保證。** 如果你的場景需要程式能直接、可靠地解析輸出，改用 Structured Outputs 的 schema 約束，而不是繼續加強 XML 提示的措辭。

**本日關鍵字回顧**

- **XML 標籤的作用**：消除 prompt 裡混合內容的歧義，不是裝飾性寫法。
- **一致命名、依層級巢狀**：官方兩條核心最佳實踐，标籤名稱望文生義、有層級關係就巢狀包裹。
- **先引用再推理**：用 `<quotes>` 搭配 `<info>` 的順序，讓模型的推理過程可被檢查，是降低幻覺的雛型技巧。
- **XML 格式指示符**：用具體標籤名稱定義輸出格式，比條列「不要做什麼」更容易被準確執行。
- **Structured Outputs**：`output_config.format` 透過約束解碼強制輸出符合 JSON schema，是需要保證有效輸出時，XML 標籤之外的正確工具。

明天回到 Day 17 骨架的第①塊——**System Prompt**。角色設定為什麼「一句話就有感」，以及為什麼單純寫「你是一位專家」常常沒什麼用。

**Day 19，System Prompt 的正確寫法。**
