# Claude 用得對，也用得省

> **工程師帶你搞懂選模型、Token 優化與底層邏輯**
>
> 2026 iThome 鐵人賽 · Claude AI 主題競賽組 · 連續 30 天

這是我第三次參加 iThome 鐵人賽。前兩屆分別寫了 JavaScript 與 Vue 3，這次挑戰 **Claude AI 主題競賽組**。

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

---

## 這個系列在寫什麼？

市面上「Claude 怎麼用」的教學已經夠多了。這系列想補的是另一半——**為什麼這樣用**。

書名的兩個「用得」，就是這 30 天的兩條主軸：

**用得對**——同一個問題，選錯模型、設錯參數，答案的品質天差地遠。

- 四個模型放在你面前，你憑什麼判斷該用哪一個？
- `effort` 調低到底犧牲了什麼？什麼時候該調、什麼時候不該？
- 為什麼你不用再寫「think step by step」了？

**用得省**——同樣一段程式碼丟給 Claude，為什麼有人花 $0.3、有人花 $3？

- 你的 token 到底花在哪裡？帳單為什麼跟你的直覺對不上？
- 快取、批次、分流——三種省法各自適用什麼場景？
- 對話越長越貴，這件事有沒有解？

**主軸是思維，不是操作步驟。** 操作步驟會過期，判斷依據不會。

### 這系列適合誰

| 你是 | 你會拿到 |
| :--- | :--- |
| 前端／後端工程師 | API 串接、Claude Code 實戰、成本優化架構思維 |
| 剛接觸 AI 工具的開發者 | 模型怎麼選、token 怎麼算、prompt 怎麼寫才穩 |
| 被 API 帳單嚇到的人 | Prompt Caching、Batch API、模型分流的完整省錢路徑 |

每篇都**獨立自足**。你從 Google 直接落地在 Day 15，不需要回頭補前面 14 天。

---

## 30 天完整目錄

### 第一階段｜先把模型選對（Day 1–6）

| 天數 | 主題 | 描述 |
| :--- | :--- | :--- |
| [Day 1](day1/README.md) | Claude 模型怎麼選？2026 最新四階模型完整比較 | Fable 5 / Opus 5 / Sonnet 5 / Haiku 4.5 的定位、規格與適用場景 |
| [Day 2](day2/README.md) | Claude 的計價邏輯：搞懂 input / output 為什麼差 5 倍 | 不背數字，理解計價結構，建立可長期沿用的成本直覺 |
| [Day 3](day3/README.md) | 不知道用哪個模型？官方建議「從 Opus 5 開始」背後的思維 | 為什麼預設起手不是最便宜、也不是最強的那個 |
| [Day 4](day4/README.md) | Claude Haiku 4.5 適合做什麼？便宜模型的正確用法 | 便宜模型不是次等品，是專用工具 |
| [Day 5](day5/README.md) | Claude context window 是什麼？1M token 到底能塞多少東西 | 用實際檔案量換算，破除「塞越多越好」的迷思 |
| [Day 6](day6/README.md) | Claude 模型選擇決策表：一張圖判斷你該用哪一個 | 把前五天濃縮成一張可以貼在螢幕旁的決策流程 |

### 第二階段｜省 token 實戰（Day 7–13）

| 天數 | 主題 | 描述 |
| :--- | :--- | :--- |
| [Day 7](day7/README.md) | Token 是什麼？為什麼你的 Claude 帳單比想像中貴 | 從 tokenizer 原理理解中文為什麼特別燒錢 |
| [Day 8](day8/README.md) | Claude 省 token 的 5 個實用技巧（一般使用者也適用） | 不寫程式也能立刻套用的五個習慣 |
| [Day 9](day9/README.md) | Prompt Caching 是什麼？讓重複內容只算 10% 費用 | 快取寫入與命中的計價邏輯，以及什麼時候會虧 |
| [Day 10](day10/README.md) | Claude Batch API 教學：非即時任務直接省一半費用 | 用時間換金錢，非同步任務的正確打開方式 |
| [Day 11](day11/README.md) | 新世代 tokenizer：同樣的中文為什麼變貴了 | Claude 4.7 世代換了 tokenizer，這對中文使用者的實際影響 |
| [Day 12](day12/README.md) | 對話越長越燒錢？Claude 長對話的成本陷阱與解法 | 每一輪都重算全部歷史——以及三種切斷成本累積的做法 |
| [Day 13](day13/README.md) | Claude 用量怎麼監控？成本失控前的預警機制 | 從 usage 欄位到 Console 儀表板，把帳單變成可觀測系統 |

### 第三階段｜設定與調校（Day 14–20）

| 天數 | 主題 | 描述 |
| :--- | :--- | :--- |
| [Day 14](day14/README.md) | Claude effort 參數是什麼？五個檔位該怎麼設 | `low` / `medium` / `high` / `xhigh` / `max` 的取捨與實測建議 |
| [Day 15](day15/README.md) | Adaptive Thinking 是什麼？為什麼你不用再寫「think step by step」 | 模型自己決定何時思考，舊 prompt 技巧為何失效 |
| [Day 16](day16/README.md) | Claude 回答變淺了？檢查這兩個隱藏設定 | 排查思路：先看 effort，再看 thinking 設定 |
| [Day 17](day17/README.md) | Claude Prompt 寫法教學：官方最佳實踐的骨架 | 一個可以套用在 90% 情境的 prompt 結構 |
| [Day 18](day18/README.md) | 用 XML 標籤讓 Claude 輸出更穩定（結構化輸出教學） | 為什麼 Claude 特別吃 XML，以及怎麼設計標籤 |
| [Day 19](day19/README.md) | System Prompt 怎麼寫？角色設定的正確姿勢 | system 與 user 的分工，以及「你是一位專家」為什麼沒用 |
| [Day 20](day20/README.md) | Claude 幻覺怎麼防？降低錯誤輸出的實用做法 | 引用來源、允許說不知道、把驗證寫進流程 |

### 第四階段｜開發者實戰（Day 21–26）

| 天數 | 主題 | 描述 |
| :--- | :--- | :--- |
| [Day 21](day21/README.md) | Claude Code 是什麼？安裝與第一次使用完整教學 | 從安裝到跑完第一個任務，含常見卡關點 |
| [Day 22](day22/README.md) | Claude Code 省 token 設定：別讓它讀完整個專案 | `CLAUDE.md`、忽略規則與 context 控制的實戰配置 |
| [Day 23](day23/README.md) | MCP 是什麼？把外部工具接進 Claude 的原理與實作 | Model Context Protocol 的設計哲學與一個可跑的範例 |
| [Day 24](day24/README.md) | 前端如何呼叫 Claude API？Messages 端點入門 | 第一支 API 請求，以及為什麼不該在瀏覽器直接呼叫 |
| [Day 25](day25/README.md) | Claude 串流輸出（Streaming）：打造即時回應體驗 | SSE 事件流解析與前端逐字渲染 |
| [Day 26](day26/README.md) | Claude API 錯誤處理與重試：正式環境該注意什麼 | 429 / 529 的正確退避策略與冪等性設計 |

### 第五階段｜底層邏輯與架構思維（Day 27–30）

| 天數 | 主題 | 描述 |
| :--- | :--- | :--- |
| [Day 27](day27/README.md) | 模型分流（Model Routing）是什麼？別再一支模型用到底 | 依任務難度動態選模型的判斷邏輯 |
| [Day 28](day28/README.md) | LLM 成本優化架構：小模型前置分流 + 大模型收尾 | 一套可落地的分層架構與失敗處理 |
| [Day 29](day29/README.md) | 從「會用」到「用得對、用得省」：我 30 天的踩坑與心法 | 誠實記錄過程中判斷錯誤的地方 |
| [Day 30](day30/README.md) | Claude 使用總整理：模型、成本、設定一次看懂 | 全系列濃縮成一份可以收藏的速查表 |

---

## 專案結構

```
30Days/
├── README.md          ← 本文件（專案總覽）
├── CLAUDE.md          ← 本系列的協作規格書
├── day1/
│   └── README.md      ← 【Day 1】文章全文
├── day2/
│   └── README.md
│   ...
└── day30/
    └── README.md
```

點進任一 `dayN/` 資料夾，GitHub 會自動渲染該天的完整文章。

---

## 關於內容正確性

Claude 的模型迭代速度比寫文章的速度還快。這系列的每一篇在動筆前都會對照 [Claude 官方文件](https://platform.claude.com/docs) 查證，並遵守三個原則：

1. **只信一手來源**——第三方部落格僅供找線索，不作為文章依據。
2. **標註查證日期**——涉及定價、模型代號、參數的段落會加註查證時間點。
3. **不確定就留白**——查不到或官方未明說的，直接說「這裡我不確定」，絕不編造。

如果你在閱讀時發現任何過期或錯誤的資訊，非常歡迎透過上面的 Email 告訴我。

---

## 前兩屆系列

- [IThome-Javascript-30days](https://github.com/Chung-Chi-Lin/IThome-Javascript-30days) — JavaScript 30 天
- [IThome-Vue-30days](https://github.com/Chung-Chi-Lin/IThome-Vue-30days) — Vue 3 30 天

---

**《Claude 用得對，也用得省：工程師帶你搞懂選模型、Token 優化與底層邏輯》**
2026 iThome 鐵人賽 · Claude AI 主題競賽組 · [Chung-Chi-Lin](https://github.com/Chung-Chi-Lin)
