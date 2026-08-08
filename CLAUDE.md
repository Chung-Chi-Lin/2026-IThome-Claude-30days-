# 專案：2026 iThome 鐵人賽 — Claude AI 主題競賽組

> 這份文件是本專案的長期記憶。開始任何寫作前，請先完整讀過，並依「啟動流程」與我核對。

---

## 一、專案基本資訊

| 項目 | 內容 |
| :--- | :--- |
| 賽事 | 2026 iThome 鐵人賽 |
| 系列書名 | **Claude 用得對，也用得省：工程師帶你搞懂選模型、Token 優化與底層邏輯** |
| 賽道 | **Claude AI 主題競賽組**（會被評分，內容需嚴謹） |
| 天數 | 連續 30 天，每天一篇 |
| 作者 | Chung-Chi-Lin（前端工程師，已完賽兩屆：JavaScript、Vue3） |
| 產出格式 | Markdown，可直接貼進 iThome 編輯器 |
| 檔案結構 | `dayN/README.md`（沿用前兩屆慣例） |
| 讀者定位 | 混合讀者：前端工程師 + 剛接觸 AI 工具的一般開發者 |
| 內容主軸 | **思維導向**：選模型策略、省 Token、設定調校、底層邏輯 |
| 額外目標 | **標題要命中 SEO**，讓人 Google 搜得到；每篇可獨立閱讀 |

---

## 二、最重要的守則：查證優先，絕不杜撰

Claude 的模型迭代速度比寫文章快。本系列的品牌價值全押在「官方最正確」上。

**每次動筆前必做：**

1. **先查官方**：`platform.claude.com/docs` 與 `docs.claude.com`，確認該篇涉及的模型名稱、API ID、參數、定價。
2. **只信一手來源**：第三方部落格僅供找線索，**絕不作為文章依據**。
   （實例：查 effort 參數時，兩個第三方頁面對檔位命名的說法互相矛盾。）
3. **不確定就標記**：查不到或官方未明說的，直接告訴我「這段請發文前再確認」，
   **寧可留白，不可造假**。
4. **標註查證日期**：涉及定價、模型代號處加註「（截至 2026 年 X 月官方文件）」。
   這既是誠實，也是 SEO 加分（讀者看到日期會更信任）。

**已知的高風險變動點：**

- Sonnet 5 的 $2/$10 導入價於 **2026-08-31 到期**，9 月鐵人賽期間應為標準價 $3/$15。
  （已於 2026-08-08 對照官方 Pricing 頁確認）
- ~~effort 檔位命名有兩種說法~~ → **已於 2026-08-08 查證結案，見第三節。**
- Claude Code 的設定項會隨版本變動，寫之前需實測 + 對照官方。
- Day27「模型分流」：學界有 LLM Cascade、FrugalGPT 等研究，
  **但正確出處尚未確認**，不可自行編造論文來源。

**價格的處理原則（2026-08-08 作者指示）：**

價格**不是**本系列的重點。讀者來這裡是學「怎麼用得對、用得省」，不是背價目表。
而且模型與定價會持續變動，把數字寫死等於替文章埋過期地雷。

- 價格只給**大概量級**與**相對比例**（例如「output 大約是 input 的 5 倍」）。
- 重點放在**計價結構為什麼長這樣**，而不是當下的絕對數字。
- 需要寫具體數字時，一律加註查證日期，並說明這是會變動的資訊。

---

## 三、已查證的基準資訊（截至 2026-08，需重新驗證）

官方目前為**四階模型**（非舊的 Opus/Sonnet/Haiku 三階）：

| 模型 | API ID | Input / Output（每百萬 token） | 定位 |
| :--- | :--- | :--- | :--- |
| Claude Fable 5 | `claude-fable-5` | $10 / $50 | 最強、已廣泛釋出 |
| Claude Opus 5 | `claude-opus-5` | $5 / $25 | **官方預設建議起手** |
| Claude Sonnet 5 | `claude-sonnet-5` | $3 / $15（導入價 $2/$10 至 8/31） | 平衡 |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | $1 / $5 | 快速、低成本 |

- Fable 5 / Opus 5 / Sonnet 5：context 1M tokens、最大輸出 128k。1M 全程標準計價，**無長文本溢價**。
- Haiku 4.5：context 200k、輸出 64k。
- Fable 5 於 **2026-06-09** 正式 GA。
- Mythos 5（`claude-mythos-5`）與 Fable 5 同規格同價，屬 Project Glasswing，
  **僅限邀請，不可寫成教學**（讀者拿不到）。

### thinking 支援狀況（重要：方向與直覺相反）

| 模型 | Extended thinking<br>`thinking.type: "enabled"` | Adaptive thinking |
| :--- | :---: | :---: |
| Claude Fable 5 | ❌ | ✅（永遠開啟） |
| Claude Opus 5 | ❌ | ✅ |
| Claude Sonnet 5 | ❌ | ✅ |
| Claude Haiku 4.5 | ✅ | ❌ |

新世代模型（Fable 5 / Opus 5 / Sonnet 5）**已無 extended thinking 這條路**，
思考深度改由 `effort` 控制。Day15、Day16 直接踩在這個點上，不可寫錯。

### effort 參數（2026-08-08 查證結案）

**五個檔位**，由低到高：`low` → `medium` → `high` → `xhigh` → `max`。
**沒有 `minimal`**（先前第三方來源的說法是錯的）。

- 參數位置是 **`output_config.effort`**，不是 top-level：
  `{ "model": "claude-opus-5", "output_config": { "effort": "medium" } }`
- 預設 `high`；設 `high` 等同不傳該參數。
- `xhigh` 較新，部分支援 `max` 的模型不支援 `xhigh`。
- Opus 5 在 `xhigh` / `max` 下**無法關閉 thinking**（傳 `thinking: {"type":"disabled"}` 回 400）。
- effort 會使快取失效——同一段對話中途改 effort 會打掉 prompt cache（Day9、Day12 可用）。
- `adaptive` 是 thinking 模式，**不是** effort 值，不可混寫。
- **Haiku 4.5 不在 effort 支援清單內**（支援清單：Fable 5、Mythos 5/Preview、Opus 5、
  Opus 4.8/4.7/4.6、Sonnet 5、Sonnet 4.6、Opus 4.5）。Day14 不可寫成「所有模型都能設」。

### 官方 400 錯誤訊息原文（Day15、Day16 可直接引用）

Opus 5 / Sonnet 5 / Fable 5 收到 `thinking.type: "enabled"`：

```text
"thinking.type.enabled" is not supported for this model.
Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

各模型 thinking 預設與拒絕值：Fable 5 永遠開啟（拒絕 `enabled`、`disabled`）；
Opus 5 / Sonnet 5 預設開啟（拒絕 `enabled`；Opus 5 在 effort `xhigh`/`max` 下亦拒絕 `disabled`）；
Haiku 4.5 預設關閉、僅支援 extended（拒絕 `adaptive`）。

### tokenizer 換代（Day11 的事實基礎）

換 tokenizer 的分界是 **Claude 4.7 世代**（Opus 4.7 起），不是 Sonnet 5 才換。
官方明載 Sonnet 4.6 及更早使用舊 tokenizer。同樣文字約產生 **多 30%** tokens。

**Day11 可直接用的實測方法（官方建議）：** token counting 端點會依你傳入的 `model`
採用該模型的 tokenizer，所以拿同一段文字呼叫兩次、換不同 model，比較兩次的
`input_tokens`，就能量出自己內容的實際增幅。官方明講不要沿用舊模型量到的數字估算成本。

### Token counting API（Day5、Day11、Day13 可用）

- 端點 `/v1/messages/count_tokens`，官方明載 **free to use**（不收費）。
- 有獨立的 RPM 上限：Start 2,000 / Build 4,000 / Scale 8,000，
  且與 message creation 的額度**互不相干**。
- 回傳為**估計值**，實際可能有小幅差異；系統自動加入的 token 不計費。
- 不會觸發快取邏輯（傳 `cache_control` 也不會真的建立快取）。

**寫作本身建議使用：Opus 5 + effort high。**

---

## 四、30 天目錄（已定案，SEO 標題版）

### 第一階段｜先把模型選對
- Day1　Claude 模型怎麼選？2026 最新四階模型 Fable 5 / Opus 5 / Sonnet 5 / Haiku 4.5 完整比較
- Day2　Claude 的計價邏輯：搞懂 input / output 為什麼差 5 倍
- Day3　不知道用哪個模型？官方建議「從 Opus 5 開始」背後的思維
- Day4　Claude Haiku 4.5 適合做什麼？便宜模型的正確用法
- Day5　Claude context window 是什麼？1M token 到底能塞多少東西
- Day6　Claude 模型選擇決策表：一張圖判斷你該用哪一個

### 第二階段｜省 token 實戰
- Day7　Token 是什麼？為什麼你的 Claude 帳單比想像中貴
- Day8　Claude 省 token 的 5 個實用技巧（一般使用者也適用）
- Day9　Prompt Caching 是什麼？讓重複內容只算 10% 費用
- Day10　Claude Batch API 教學：非即時任務直接省一半費用
- Day11　新世代 tokenizer：同樣的中文為什麼變貴了
- Day12　對話越長越燒錢？Claude 長對話的成本陷阱與解法
- Day13　Claude 用量怎麼監控？成本失控前的預警機制

### 第三階段｜設定與調校
- Day14　Claude effort 參數是什麼？high / medium / low 該怎麼設
- Day15　Adaptive Thinking 是什麼？為什麼你不用再寫「think step by step」
- Day16　Claude 回答變淺了？檢查這兩個隱藏設定
- Day17　Claude Prompt 寫法教學：官方最佳實踐的骨架
- Day18　用 XML 標籤讓 Claude 輸出更穩定（結構化輸出教學）
- Day19　System Prompt 怎麼寫？角色設定的正確姿勢
- Day20　Claude 幻覺怎麼防？降低錯誤輸出的實用做法

### 第四階段｜開發者實戰
- Day21　Claude Code 是什麼？安裝與第一次使用完整教學
- Day22　Claude Code 省 token 設定：別讓它讀完整個專案
- Day23　MCP 是什麼？把外部工具接進 Claude 的原理與實作
- Day24　前端如何呼叫 Claude API？Messages 端點入門
- Day25　Claude 串流輸出（Streaming）：打造即時回應體驗
- Day26　Claude API 錯誤處理與重試：正式環境該注意什麼

### 第五階段｜底層邏輯與架構思維
- Day27　模型分流（Model Routing）是什麼？別再一支模型用到底
- Day28　LLM 成本優化架構：小模型前置分流 + 大模型收尾
- Day29　從「會用」到「用得對、用得省」：我 30 天的踩坑與心法
- Day30　Claude 使用總整理：模型、成本、設定一次看懂（2026 完整版）

---

## 五、資料夾結構（重要：必須嚴格遵守）

沿用作者前兩屆（JavaScript-30days、Vue-30days）的既有慣例：
**一天一個資料夾，文章一律命名為 `README.md`。**

```
D:\30Days\
├── CLAUDE.md          ← 本文件（需納入版控，不要 gitignore）
├── README.md          ← 專案總覽（含 30 天目錄表 + 聯繫我）
├── .gitignore
├── day1/
│   └── README.md      ← 【Day 1】文章全文
├── day2/
│   └── README.md      ← 【Day 2】文章全文
├── day3/
│   └── README.md
│   ...
└── day30/
    └── README.md
```

**規則說明：**

- 資料夾一律小寫 `day` + 數字，**不補零**（是 `day1` 不是 `day01`）。
- 每個資料夾內只放一個 `README.md`，這樣 GitHub 進入資料夾會自動渲染文章。
- 若該篇有搭配的程式碼範例檔，放在同資料夾內（例如 `day24/example.js`）。
- 圖片放在該天資料夾內的 `images/` 子目錄。
- **`CLAUDE.md` 需納入版控**，不要加進 `.gitignore`。
  本專案主題即為 Claude AI，這份文件本身就是「如何用 Claude 協作」的實作證據。

---

## 六、文章格式規格（沿用前兩屆風格）

每篇 `dayN/README.md` 必須包含以下區塊，順序固定：

```markdown
# 【Day N】主標題：副標題

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言
（第一人稱開場，連結今天要解決的痛點）

## 目錄
（完整 30 天表格，欄位：天數 | 主題 | 描述）

## 正文區塊（依主題自訂 ## 與 ### 標題）
（比喻先行 → 技術細節 → 程式碼範例 → `>` 引言區塊解說）

## 本篇自我挑戰
- **今日挑戰**：（可動手做的具體任務）
- **反思**：（開放式問題，邀請留言討論）

## 總結
（收束今日重點，3～5 句）

本日關鍵字回顧

- 關鍵字A: 一句話定義。
- 關鍵字B: 一句話定義。

（結尾一句預告明天內容，帶點期待感）
```

---

## 七、寫作語氣規格

**定位：有技術涵養的口語教學。** 像大學裡最受歡迎的那位教授——
不是照稿念的老學究，也不是只會講段子的搞笑咖。

### 要做到
- **第二人稱「你」**，直接對讀者說話。
- **設問句帶節奏**：「心累不？」「看到差別了嗎？」「注意到了嗎？」
- **比喻先行**：用生活化比喻當入口，再進技術細節。
  （範例：Vue 2 像家用車開上賽道／編譯器像偵探／Proxy 是懶漢模式）
- **程式碼用前後對照組**：Before / After，後面接 `>` 區塊解說差異。
- **刻意植入專有名詞**：讓讀者拿得到關鍵字自己去查（作者的既定策略）。
- **每篇獨立自足**：讀者可能從 Google 直接落地在 Day15，不預設他讀過前面。

### 不要做
- 不要太死板、教科書腔。
- 不要太嘻嘻哈哈：不玩梗過頭、不用網路流行語轟炸、不塞大量表情符號。
- 比喻是入口，**不可取代技術內容**——比喻之後一定要補上實際原理。

---

## 八、啟動流程（第一次對話請照做）

1. 讀完本文件。
2. **先與我核對**下列事項，取得確認後再開始寫作：
    - 30 天目錄是否需要調整？
    - 是否先寫一篇 Day1 當範本，滿意後再往下展開？
    - 檔案要放在哪個路徑（預設 `dayN/README.md`）？
3. 核對通過後，**每寫一篇前先查證官方文件**，再產出 Markdown。
4. 產出後主動告訴我：**哪些內容是查證過的、哪些需要我發文前自行確認。**

---

## 九、目前進度

> **接續工作前請先讀 [`PROGRESS.md`](PROGRESS.md)** ——
> 那裡有現況、待辦、已埋的伏筆，以及下一批要做什麼。

- [x] 賽道選定：Claude AI 組
- [x] 30 天目錄定案（SEO 標題版）
- [x] 寫作風格與格式規格確立
- [x] 系列書名定案（2026-08-08）
- [x] 官方文件查證第一輪：定價、模型規格、effort、thinking、tokenizer（2026-08-08）
- [x] 專案 README.md（含 30 天目錄表）
- [x] **第一階段 Day1～Day6 完成**（2026-08-08）
- [ ] 第二階段 Day7～Day13（省 token 實戰）← 下一批
- [ ] 第三階段 Day14～Day20（設定與調校）
- [ ] 第四階段 Day21～Day26（開發者實戰）
- [ ] 第五階段 Day27～Day30（底層邏輯與架構思維）

**已定案的規格補充（2026-08-08）：**

- 每篇篇幅：**2000～3000 字**。
- 每篇的「目錄」區塊：**放完整 30 天表格**（沿用前兩屆做法）。
- 查證方式：撰寫前**上網比對官方文件**，不倚賴記憶。
- 前兩屆 repo：
  [IThome-Javascript-30days](https://github.com/Chung-Chi-Lin/IThome-Javascript-30days) /
  [IThome-Vue-30days](https://github.com/Chung-Chi-Lin/IThome-Vue-30days)
- 本屆 repo：[2026-IThome-Claude-30days-](https://github.com/Chung-Chi-Lin/2026-IThome-Claude-30days-)