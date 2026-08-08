# 進度交接（PROGRESS）

> **最後更新：2026-08-08**
> 下次開新對話時，請先讀 `CLAUDE.md`（規格書），再讀本檔（現況與待辦）。

---

## 一、現在做到哪裡

### 完成度

```
第一階段  Day 1-6    ██████████  100%  已完成並 commit
第二階段  Day 7-13   ░░░░░░░░░░    0%  ← 下一批從這裡開始
第三階段  Day 14-20  ░░░░░░░░░░    0%
第四階段  Day 21-26  ░░░░░░░░░░    0%
第五階段  Day 27-30  ░░░░░░░░░░    0%
```

### 已產出的檔案

| 檔案 | 內容 | 篇幅 |
| :--- | :--- | ---: |
| `README.md` | 專案總覽（書名、系列說明、30 天目錄表、查證聲明、前兩屆連結） | — |
| `CLAUDE.md` | 協作規格書（已納入版控，**不要 gitignore**） | — |
| `.gitignore` | 排除 `.idea/` 與 OS 檔案 | — |
| `day1/README.md` | 四階模型完整比較 | ~2324 字 |
| `day2/README.md` | 計價邏輯：1:5 與三個乘數 | ~2055 字 |
| `day3/README.md` | 為什麼官方預設是 Opus 5 | ~1934 字 |
| `day4/README.md` | Haiku 4.5 的正確用法 | ~2179 字 |
| `day5/README.md` | Context rot 與空間管理 | ~2159 字 |
| `day6/README.md` | 決策樹與四個反模式 | ~1938 字 |

### Git 狀態

- 分支 `main`，remote 已接上 `2026-IThome-Claude-30days-`
- **本地有 3 個 commit 尚未 push**（`6674dd1`、`a580396`、`a3825d1`）
- 下次開工前先跑：`git push`

---

## 二、已定案的決定（不要重新討論）

| 項目 | 決定 |
| :--- | :--- |
| **系列書名** | Claude 用得對，也用得省：工程師帶你搞懂選模型、Token 優化與底層邏輯 |
| **篇幅** | 每篇 2000–3000 字 |
| **目錄區塊** | 每篇都放**完整 30 天表格**（沿用前兩屆做法） |
| **價格處理** | 價格**不是**重點。只給量級與相對比例，重點放在計價結構。細節見 `CLAUDE.md` 第二節 |
| **查證方式** | 動筆前**上網比對官方文件**，不倚賴記憶 |
| **檔案結構** | `dayN/README.md`，資料夾小寫不補零 |

### 已修改過的目錄標題（三處）

| 天數 | 原標題 | 現標題 | 原因 |
| :--- | :--- | :--- | :--- |
| Day 2 | Claude 模型價格全解析 | Claude 的計價邏輯：搞懂 input / output 為什麼差 5 倍 | 價格不是重點，改講結構 |
| Day 11 | Sonnet 5 換了 tokenizer | 新世代 tokenizer | 事實錯誤：換代分界是 4.7 世代，非 Sonnet 5 |
| Day 29 | 從「會用」到「用得省」 | 從「會用」到「用得對、用得省」 | 對齊書名的雙軸 |

---

## 三、已查證的事實（2026-08-08）

完整內容在 `CLAUDE.md` 第三節。以下是**推翻了原本認知**的幾項，特別容易寫錯：

1. **effort 是五個檔位**：`low` / `medium` / `high` / `xhigh` / `max`。**沒有 `minimal`**。
   參數位置是 `output_config.effort`，不是 top-level。
2. **Haiku 4.5 不在 effort 支援清單內。** Day 14 不可寫成「所有模型都能設」。
3. **Extended / Adaptive thinking 是互斥的，方向反直覺**：
   Fable 5 / Opus 5 / Sonnet 5 只有 adaptive；**Haiku 4.5 只有 extended**。
4. **Opus 5 的知識截止日（2026-05）比 Fable 5（2026-01）更新。** 越貴不代表越新。
5. **tokenizer 換代分界是 Claude 4.7 世代**，同樣文字約多 30% token。
6. **快取不會省 context 空間**，只改變計價。官方原文：
   「changes what you pay for those tokens, not whether they count」
7. **Token counting API 免費**，且 RPM 額度與發訊息**分開計算**。
8. 定價與 Sonnet 5 導入價到期日（2026-08-31）皆與原規格書一致，已確認。

---

## 四、待處理事項

### 需要作者決定

- [ ] **Day 1、Day 2 的調性審核** — 這兩篇定了整個系列的框架，建議優先看。要改的話越早越省事。
- [ ] **Day 3、Day 6 篇幅**（~1934 / ~1938 字）略低於 2000，是否需要再補。
- [ ] **iThome 支不支援 mermaid？** Day 6 同時放了 ASCII 決策樹與 mermaid 流程圖。
      若 iThome 不支援，發文時刪掉 mermaid 區塊即可，ASCII 版可獨立成立。
- [ ] Day 1 的三個提問尚未回覆：比喻密度是否足夠、「四個模型的個性」該節是否保留、
      反思段落要「只問不答」還是「先給答案再反問」。

### 寫作時的已知風險

- [ ] **Day 27「模型分流」**：學界有 LLM Cascade、FrugalGPT 等研究，
      **但正確出處尚未確認**。寫之前務必查證，或直接不提論文來源。
- [ ] **Day 21、Day 22（Claude Code）**：設定項會隨版本變動，需實測 + 對照官方。
- [ ] **Day 11** 可用官方建議的實測法：同一段文字用不同 `model` 呼叫
      `count_tokens` 兩次，比較 `input_tokens` 差異。方法已記在 `CLAUDE.md`。

---

## 五、下一批要做什麼

**第二階段：Day 7–13（省 token 實戰），共 7 篇。**

| 天數 | 主題 | 動筆前需查證的官方頁面 |
| :--- | :--- | :--- |
| Day 7 | Token 是什麼？中文為什麼特別燒錢 | tokenizer 相關說明 |
| Day 8 | 省 token 的 5 個實用技巧 | （綜合，可沿用已查資料） |
| Day 9 | Prompt Caching 只算 10% 費用 | `/build-with-claude/prompt-caching` |
| Day 10 | Batch API 直接省一半 | `/build-with-claude/batch-processing` |
| Day 11 | 新世代 tokenizer 與中文成本 | pricing 頁的 tokenizer 註記 + token counting |
| Day 12 | 長對話的成本陷阱 | `/build-with-claude/compaction`、context-editing |
| Day 13 | 用量監控與預警 | `usage` 欄位、rate limits、Console |

### 已經埋好、後面必須兌現的伏筆

寫後續篇章時**務必回頭兌現**，否則前面的承諾會斷掉：

- Day 2 埋的「三個乘數」框架（單價 × 用量 × 次數）→ 第二階段應該扣著這個框架走
- Day 3 的「20 個測試案例」→ Day 4、Day 5 已重複使用，Day 27 分流條件還要用
- Day 5 說「Day 7 講 tokenizer 原理、Day 11 講換代衝擊」→ 兩篇的分工不要重疊
- Day 4、Day 5 都指向 Day 9（快取）與 Day 10（批次）的完整說明
- Day 6 說「帳單超出預期時先確認錢花在哪」→ Day 13 要接住

### 每篇的固定流程

1. 讀 `CLAUDE.md`（尤其第二節查證守則、第六節格式、第七節語氣）
2. 上網查該篇涉及的官方文件
3. 依格式產出 `dayN/README.md`
4. 用純中文字數 × 1.3 估算篇幅，確認落在 2000–3000
5. 產出後主動說明：**哪些查證過、哪些需作者發文前自行確認**

---

## 六、給下次的提醒

- **不確定就標記，不要編。** 這是整個系列的品牌價值所在。
  Day 2 有一段「output 為何較貴」的技術解釋，我已明確標註為「業界普遍理解、非官方說明」——
  這種處理方式請沿用。
- **每篇都要能獨立閱讀。** 讀者可能從 Google 直接落地在 Day 15。
- **比喻是入口，不能取代技術內容。** 比喻之後一定要補實際原理。
- 篇幅估算指令（PowerShell）：

  ```powershell
  $t = Get-Content dayN\README.md -Raw
  $all = ([regex]::Matches($t, '[一-鿿]')).Count
  $rows = ($t -split "`n" | Where-Object { $_ -match '^\| Day \d' }) -join ''
  $cjk = $all - ([regex]::Matches($rows, '[一-鿿]')).Count
  [math]::Round($cjk * 1.3)   # 估計字數
  ```
