# 【Day 28】LLM 成本優化架構：小模型前置分流 + 大模型收尾

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

Day 27 給了模型分流的學術根據跟一個簡化範例。今天把它組裝成一套完整的架構——不是一段程式碼片段，是一個可以真正放進正式環境的三層系統。

這篇是全系列的整合篇，前 27 天教過的工具，今天會一個個回來報到。與其說今天在學新東西，不如說今天在**驗收**：把散落各處的知識點，接成一條完整的請求處理流水線。

> 本篇架構設計綜合前 27 天已查證的內容；`service_tier`（服務層級）相關規格於 **2026 年 8 月 17 日**對照 [Service tiers 官方文件](https://platform.claude.com/docs/en/api/service-tiers) 額外查證。

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

## 一、三層架構總覽

```
使用者請求
    │
    ▼
┌─────────────────┐
│ 第一層：前置分流層 │  Haiku 4.5，快、便宜，只做一件事：判斷難度
└────────┬─────────┘
         │
    ┌────┴────┐
    ▼         ▼
  簡單任務    複雜任務
    │         │
    ▼         ▼
┌─────────────────┐
│ 第二層：主要處理層 │  依難度動態選模型 + effort，Prompt 骨架化
└────────┬─────────┘
         │
         ▼
┌─────────────────┐
│ 第三層：品質驗證層 │  Day 20 技巧當保險絲，信心不足就升級重跑
└────────┬─────────┘
         │
         ▼
      最終回應
```

三層各自對應一個明確的職責，也各自對應本系列前面已經講透的知識點。

## 二、第一層：前置分流——用最便宜的模型做判斷

這一層唯一的工作是**分類**，不是解決問題本身。呼應 Day 4 講過的 Haiku 4.5 定位——「快速、低成本」正是分類任務的天生歸宿：

```python
def classify(request):
    result = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=10,
        messages=[{"role": "user", "content": f"分類這個請求的難度，只回答 simple/medium/complex：{request}"}],
    )
    return result.content[0].text.strip()
```

這一層值得疊加兩個 Day 9、Day 10 教過的技巧：**如果分類規則本身（例如系統提示裡描述「什麼算 simple、什麼算 complex」的說明）是固定不變的，用 Prompt Caching 讓它只算一次全額**；**如果你的應用可以接受非即時處理（例如夜間批次跑完隔天的客服工單分類），改用 Batch API 直接五折**。

## 三、第二層：主要處理——難度決定給誰做、給多少力氣

分類結果出來之後，這一層決定**用哪個模型、開多少 effort**：

```python
ROUTING_TABLE = {
    "simple":  {"model": "claude-haiku-4-5-20251001", "effort": None},
    "medium":  {"model": "claude-sonnet-5", "effort": "medium"},
    "complex": {"model": "claude-opus-5", "effort": "high"},
}

def process(request, difficulty):
    config = ROUTING_TABLE[difficulty]
    params = {
        "model": config["model"],
        "max_tokens": 2048,
        "system": SYSTEM_PROMPT,  # Day 19：功能性角色設定，附具體標準
        "messages": [{"role": "user", "content": build_prompt(request)}],  # Day 17/18：骨架化 + XML
    }
    if config["effort"]:
        params["output_config"] = {"effort": config["effort"]}
    return client.messages.create(**params)
```

**這一層是 Day 3 到 Day 19 全部知識的匯集點**：模型選型的判斷（Day 3、Day 4、Day 6）、effort 的分級（Day 14）、thinking 的自然觸發（Day 15）、prompt 骨架與 XML 結構（Day 17、Day 18）、角色設定（Day 19）——這些不是各自獨立的技巧，是同一套系統裡不同的旋鈕。

**注意 `simple` 那一列刻意把 `effort` 設成 `None`**——Day 14 查證過 Haiku 4.5 不在 effort 支援清單內，路由表設計時要記得這個邊界，不要對不支援的模型傳這個參數。

## 四、第三層：品質驗證——不是每題都要驗，但要有觸發機制

如果每一題都跑一次 Day 20 的完整驗證流程（先引用再推理、Best-of-N），那整套架構的成本又回到跟「每題都用最貴模型」差不多的量級——**驗證本身也要分級**。

實用的做法是把「觸發驗證」的條件跟第二層的路由結果掛鉤：

```python
def process_with_verification(request, difficulty):
    response = process(request, difficulty)

    # 只有中高難度、或模型自己承認不確定時，才觸發驗證與升級
    if difficulty in ("medium", "complex") and shows_uncertainty(response):
        response = escalate(request, current_tier=difficulty)

    return response

def shows_uncertainty(response):
    # Day 20 的「允許說不知道」技巧，這裡變成可被程式判斷的訊號
    text = response.content[0].text
    return "沒有足夠的資訊" in text or "需要人工複核" in text

def escalate(request, current_tier):
    next_tier = {"simple": "medium", "medium": "complex", "complex": "complex"}[current_tier]
    return process(request, next_tier)  # 升級到更高一級重跑
```

**這正是 Day 27 講的 Cascade 精神在架構裡的具體實作**——不是每題都驗證，是讓 Day 20 教的「允許說不知道」變成一個機器可以偵測的訊號，偵測到才觸發升級，把驗證成本集中花在真正需要的地方。

## 五、三層共用的兩件事：監控與容錯

**監控要貫穿全部三層**，不能只看整體帳單。Day 13 教的 `/usage` 分析、Console 的 rate limit 圖表，這裡建議依層級拆開看——如果你發現「前置分流層」的花費佔比異常高，代表分類器本身可能設計得太重（例如不小心把 `max_tokens` 設太大，或分類 prompt 太長沒有妥善利用快取）；如果「品質驗證層」觸發率異常高，代表第二層的路由規則可能判斷得不夠準，太多任務被低估難度。

**容錯要遵守 Day 26 的原則，尤其是分類步驟本身失敗的情況。** 如果第一層的分類請求逾時或出錯，**保守的預設值應該是「當作複雜任務處理」，而不是「當作簡單任務處理」**——分類失敗時寧可多花一點錢送到更強的模型，也不要讓一個沒分類成功的請求意外落到能力不足的模型手上、產生一個看起來合理但其實有問題的答案。這是 Day 26 「冪等性與副作用要謹慎」原則的延伸：**分流邏輯本身的失敗模式，也要往安全的方向倒，而不是往省錢的方向倒。**

## 六、一個容易被忽略的旁支：Priority Tier

官方還提供一種跟「模型分流」不同軸線的成本／延遲控制——**Service Tier（服務層級）**，用 `service_tier` 參數指定：

```python
message = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude!"}],
    service_tier="auto",  # Priority Tier 有餘額就優先用，沒有就自動退回標準層
)
```

Priority Tier 的設計目的是**在尖峰時段降低撞到「伺服器過載」（Day 26 提過的 529）的機率**，付出的代價是需要事先跟官方簽訂容量承諾。

**但這裡有兩個重要的現況查證，發文前務必留意**：**第一，官方明確標示 Priority Tier 容量承諾目前已經停止開放新購買**，只有原本就有承諾的組織能繼續使用到合約到期；**第二，Priority Tier 不支援本系列從頭到尾在用的 Opus 5 與 Sonnet 5**（也不支援 Fable 5、Mythos 5/Preview）。這代表對多數讀者來說，這個功能目前**不是一個可以規劃進架構裡的選項**——放在這裡當作背景知識，提醒你這個維度存在，但不建議把它當成今天架構設計的核心依據。

## Before / After：一個客服系統的完整改造

**❌ Before：所有工單都送 Opus 5，固定 high effort**

```python
def handle_ticket(ticket):
    return client.messages.create(
        model="claude-opus-5",
        max_tokens=2048,
        messages=[{"role": "user", "content": ticket}],
    )
# 一萬張工單，不管是「怎麼重設密碼」還是「合約條款爭議」，一律最貴規格處理
```

**✅ After：三層架構分工**

```python
def handle_ticket(ticket):
    difficulty = classify(ticket)               # 第一層：Haiku 4.5 分類
    response = process(ticket, difficulty)       # 第二層：依難度選模型與 effort
    return process_with_verification(ticket, difficulty)  # 第三層：不確定時才升級驗證
```

> Before 版本裡，「怎麼重設密碼」這種佔工單大宗的簡單問題，付的是跟「合約條款爭議」這種真正需要深度分析的問題一樣的價錢。After 版本讓簡單問題在第一層分類後，直接由 Haiku 4.5 用最低成本解決；複雜問題才逐層升級，用到 Opus 5 跟高 effort。**三層架構不是三倍的複雜度，是把 Day 2 那個「總成本 = 單價 × 用量 × 次數」的框架，實際套進系統設計裡——三個乘數,現在分別由三層架構的三個環節分開管理。**

## 本篇自我挑戰

- **今日挑戰**：畫出你目前系統的請求處理流程圖，標出「這一步用了哪個模型、什麼 effort」。如果流程圖只有一條線、一路到底都是同一個模型，考慮從最容易分類的一小塊任務開始，試著切出第一層分流。

- **反思**：三層架構的核心精神是「不要用同一套標準對待所有請求」。這句話放到系統設計之外，其實也是很多管理決策的核心——你在資源分配、時間管理上，有沒有類似「所有事情都用同一種認真程度處理」而其實可以分級的地方？

## 總結

今天把前 27 天的知識組裝成一套三層架構：**前置分流層用 Haiku 4.5 做便宜的難度判斷**；**主要處理層依難度動態選模型與 effort，套用骨架化的 prompt 與角色設定**；**品質驗證層不是每題都驗，而是在偵測到不確定訊號時才觸發升級重跑**。監控要拆層級看，容錯要記得「分類失敗時該往保守方向倒，而不是省錢方向」。

Priority Tier 是另一個值得知道、但目前不適合多數讀者規劃進架構的維度——已停止新購買，且不支援 Opus 5 / Sonnet 5。

**本日關鍵字回顧**

- **三層架構**：前置分流層（Haiku 分類）→ 主要處理層（依難度選模型/effort）→ 品質驗證層（不確定才升級）。
- **分類即服務**：第一層的唯一職責是判斷難度，本身應該盡可能便宜、快速。
- **升級觸發訊號**：把 Day 20「允許說不知道」的技巧變成程式可判斷的訊號，決定是否觸發第三層驗證。
- **保守容錯原則**：分流邏輯本身失敗時，應預設當作複雜任務處理，而非簡單任務，避免品質不足的答案被誤放行。
- **Service Tier / Priority Tier**：另一個成本／延遲控制維度，目前已停止新購買且不支援 Opus 5、Sonnet 5，僅供背景知識參考。

明天收尾前的最後一次盤點——**這 30 天走下來，哪些判斷是我一開始就想錯的，哪些是查證之後才發現的認知落差。**

**Day 29，30 天的踩坑與心法。**
