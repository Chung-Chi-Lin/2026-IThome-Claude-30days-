# 進度交接（PROGRESS）

> **最後更新：2026-08-19**
> 下次開新對話時，請先讀 `CLAUDE.md`（規格書），再讀本檔（現況與待辦）。
>
> **全系列 30 天草稿已全數完成，且已 commit（尚未 push）。** 下一步是作者審核，
> 審核通過後 push + 發文。這不是「繼續寫作」的交接文件了，是「審核與發文前確認」的清單。
>
> **2026-08-19 重要更正**：Sonnet 5 導入價「$2/$10 於 8/31 到期、9 月起恢復 $3/$15」
> **沒有照原訂計畫發生**——官方公告 $2/$10 直接轉為標準價，9/1 漲價計畫已取消。
> `CLAUDE.md`、`day1`、`day2`、`day10` 已訂正，`README.md`（GitHub 聯絡連結已移除，
> 見下方）內容不受影響。

---

## 一、現在做到哪裡

### 完成度

```
第一階段  Day 1-6    ██████████  100%  已完成並 commit
第二階段  Day 7-13   ██████████  100%  已完成，尚未 commit ← 待作者審核
第三階段  Day 14-20  ██████████  100%  已完成，尚未 commit ← 待作者審核
第四階段  Day 21-26  ██████████  100%  已完成，尚未 commit ← 待作者審核
第五階段  Day 27-30  ██████████  100%  已完成，尚未 commit ← 待作者審核

全系列 30/30 篇草稿完成。剩下的工作是：審核 → 視需要補字數 → commit → push → 發文。
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
| `day7/README.md` | Token 是什麼、tokenizer 原理與中文效率 | ~2037 字 |
| `day8/README.md` | 5 個省 token 實用技巧 | ~1859 字 |
| `day9/README.md` | Prompt Caching 完整運作機制 | ~1992 字 |
| `day10/README.md` | Batch API 教學與計價 | ~1841 字 |
| `day11/README.md` | 新舊 tokenizer 換代衝擊 | ~1788 字 |
| `day12/README.md` | Compaction 與 Context Editing | ~1821 字 |
| `day13/README.md` | 用量監控與三道防線 | ~1820 字 |
| `day14/README.md` | effort 五個檔位完整攻略 | ~1846 字 |
| `day15/README.md` | Adaptive Thinking 原理與逐模型設定表 | ~1799 字 |
| `day16/README.md` | 回答變淺排查清單（含案例） | ~1937 字 |
| `day17/README.md` | Prompt 骨架七區塊 | ~1804 字 |
| `day18/README.md` | XML 標籤最佳實踐 | ~1830 字 |
| `day19/README.md` | System Prompt 角色設定 | ~1921 字 |
| `day20/README.md` | 降低幻覺的組合技 | ~1992 字 |
| `day21/README.md` | Claude Code 安裝與第一次使用 | ~1747 字 |
| `day22/README.md` | Claude Code 省 token 實戰設定 | ~1799 字 |
| `day23/README.md` | MCP 原理與 Connector 實作 | ~1719 字 |
| `day24/README.md` | Messages 端點入門與瀏覽器風險 | ~1667 字 |
| `day25/README.md` | Streaming SSE 事件解析 | ~1655 字 |
| `day26/README.md` | 錯誤處理、重試與冪等性 | ~1946 字 |
| `day27/README.md` | 模型分流原理與 FrugalGPT 出處 | ~1755 字 |
| `day28/README.md` | 三層成本優化架構 | ~1698 字 |
| `day29/README.md` | Vibecoding 盲區與提問反轉 | ~1730 字 |
| `day30/README.md` | 全系列速查表總整理 | ~1396 字 |

### Git 狀態

- 分支 `main`，remote 已接上 `2026-IThome-Claude-30days-`
- **本地有多個 commit 尚未 push**（第一階段 3 個 + 本次 Day 7-30 尚未 commit）
- **Day 7-30（共 24 篇）尚未 commit**，等作者審核過再 commit（本次對話未經作者要求不 commit）
- 下次開工前先跑：`git status` 確認 Day 7-30 是否已被處理，再 `git push`

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
| Day 29 | 從「會用」到「用得對、用得省」：我 30 天的踩坑與心法 | Vibecoding 做出網站之後：AI 不會主動告訴你的那些事 | 原稿是寫作過程自述，改為對讀者更有用的 vibecoding 盲區主題 |

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
8. ~~定價與 Sonnet 5 導入價到期日（2026-08-31）皆與原規格書一致，已確認~~ →
   **2026-08-19 更新：這件事後來變了。** 官方公告 Sonnet 5 的 $2/$10 直接轉為標準價，
   原訂 9/1 起漲到 $3/$15 的計畫取消。詳見下方第四節「需要作者決定」的訂正記錄。

---

## 四、待處理事項

### 需要作者決定

- [ ] **Day 1、Day 2 的調性審核** — 這兩篇定了整個系列的框架，建議優先看。要改的話越早越省事。
- [ ] **多篇篇幅略低於 2000 字下限**（用 CLAUDE.md 的估算公式量測，已排除目錄表格）：
      Day 7-29 共 22 篇落在 1396～1946 字之間（完整清單見上方檔案表，逐篇字數已標註）。
      內容完整、查證足夠，但字數卡在下限邊緣——是否需要再補，還是接受目前篇幅，請作者定奪。
      **Day 30 是速查表格式，表格密度高、純文字量本來就會偏低（~1396 字），這篇建議不用硬湊字數**，
      跟其他天的「敘事型」文章性質不同，湊字數反而會稀釋速查表的可讀性。
- [ ] **iThome 支不支援 mermaid？** Day 6 同時放了 ASCII 決策樹與 mermaid 流程圖。
      若 iThome 不支援，發文時刪掉 mermaid 區塊即可，ASCII 版可獨立成立。
- [ ] Day 1 的三個提問尚未回覆：比喻密度是否足夠、「四個模型的個性」該節是否保留、
      反思段落要「只問不答」還是「先給答案再反問」。
- [ ] **Day 7-30（共 24 篇）尚未 commit**，請作者審過內容後再 commit（本次刻意不自動 commit）。
- [ ] **Day 19 第三節需要作者判斷**：「你是一位專家為什麼沒用」是唯一一段找不到官方
      逐字依據的推論內容（嘗試查證 `prompt-engineering/system-prompts` 專頁，該網址目前
      302 轉址回合併後的 `claude-prompting-best-practices` 頁面，轉址後內容裡沒有找到
      「system vs user 分工」「泛用角色 vs 具體角色」的官方原文），已在文中明確標記為推論，
      建議發文前確認是否要補充其他佐證或調整論述強度。
- [ ] **Day 24 第四節需要作者判斷**：`anthropic-dangerous-direct-browser-access` 這個標頭
      名稱與用途，是透過 WebSearch 交叉比對多個第三方來源（包含 SDK PR 討論、Simon Willison
      的技術部落格）確認的，**沒有在 platform.claude.com 官方文件頁面上直接找到專門說明這個
      標頭的段落**。標頭名稱本身、用途、安全疑慮的推論方向都合理，但嚴格來說不是「官方逐字
      文件查證」，屬於 CLAUDE.md 第二節「不確定就標記」的情況，發文前建議自己再測試一次
      （對 `https://api.anthropic.com/v1/messages` 從瀏覽器 fetch 帶上這個標頭，確認行為是否
      如文中描述）。

### 寫作時的已知風險

- [x] ~~**Day 27「模型分流」**：學界有 LLM Cascade、FrugalGPT 等研究，但正確出處尚未確認~~ →
      **已查證解決**。正確出處是 Lingjiao Chen、Matei Zaharia、James Zou（史丹佛，2023）發表的
      《FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance》，
      arXiv:2305.05176。作者、機構、arXiv 編號、核心主張（成本最高降 98%）皆經多方來源交叉比對確認。
      文中已明確區分「這是學術研究、不是 Anthropic 官方功能」，避免讀者誤以為 API 有內建分流開關。
- [x] ~~**Day 21、Day 22（Claude Code）**：設定項會隨版本變動，需實測 + 對照官方~~ →
      已對照 2026-08-17 當下的 `code.claude.com` 官方文件查證，惟 Claude Code 更新頻率高，
      發文前建議用 `claude --version` 確認指令與行為未變。
- [x] ~~**Day 11** 可用官方建議的實測法~~ → 已在 Day 11 寫入完整方法與程式碼範例。
- [x] ~~**Day 14-16 effort／thinking 查證**~~ → 已重新對照最新官方文件（含
      `thinking-steering-and-cost`、`thinking-troubleshooting` 兩篇），
      CLAUDE.md 第三節的初步結論全數覆核通過，且補上更完整的逐模型設定表與三句 400 錯誤原文。
- [x] ~~**Day 23 MCP**~~ → 已查證目前是 `mcp-client-2025-11-20` 版本（前一版
      `mcp-client-2025-04-04` 已棄用），文中有標註新舊版本差異，供發文前再次確認版本號未變。

### Day 7-30 內文中，明確標記「非官方逐字保證、屬合理推論」的段落（發文前可再次確認）

- **Day 7 第一、二節**：子詞切分機制與「中文容易被切碎」的原理，屬業界通用技術背景，
  非 Anthropic 官方逐字說明。
- **Day 11 第五節**：「output token 是否也適用約 30% 增幅」——官方文件只明確講 input，
  output 端的推論是基於 tokenizer 雙向編碼機制的合理推論，未經官方數字證實。
- **Day 19 第三節**：「你是一位專家為什麼沒用」與「功能性角色 vs 裝飾性角色」判準，
  是基於官方角色範例邏輯的推論，非官方逐字定義（詳見上方「需要作者決定」）。
- **Day 24 第四節**：`anthropic-dangerous-direct-browser-access` 標頭說明，來源為第三方
  交叉比對而非官方頁面逐字查證（詳見上方「需要作者決定」）。
- **Day 26 第八節**：「Messages API 沒有內建冪等鍵機制」是查證後的**否定性結論**（找不到
  官方文件記載此功能，且 Claude Code routine-fire 端點的文件明確說「沒有 idempotency
  key」），不是直接引用官方逐字聲明「絕對沒有」，發文前可留意官方是否新增此功能。
- 其餘數字性內容（快取倍率、最小可快取長度、批次折扣、rate limit 數字、compaction/context
  editing、effort 五檔位、thinking 逐模型設定表、SSE 事件格式、HTTP 錯誤碼表的參數與預設值）
  均為官方文件逐字查證，已於各篇文末標註查證日期（2026-08-14 或 2026-08-17）。

---

## 五、發文前最終檢查清單

**全系列 30 篇草稿都完成了，接下來不是「寫下一批」，是「審核 → 發文」。** 建議依這個順序處理：

1. **先看 Day 1、Day 2**（見上方「需要作者決定」）——這兩篇定調整個系列，要改越早改越省事。
2. **抽查標記為「推論／非官方逐字」的四段內容**（Day 7、Day 11、Day 19、Day 24，完整清單見下方），
   判斷要不要補強佐證、調整措辭強度，或接受目前的標記方式直接發文。
3. **決定篇幅政策**：22 篇落在 2000 字下限之下，是要逐篇補到門檻以上，還是接受「內容完整優先於
   字數門檻」，統一調整 CLAUDE.md 裡的篇幅規則？這個決定會影響後續類似系列的規格，值得認真想一次。
4. **跑一次全系列一致性檢查**：30 天目錄表每篇都要完全一致（已用同一份文字複製貼上到全部 30 篇，
   理論上一致，但若之後手動修改任何一天的標題，記得同步更新全部 30 篇的目錄表格，不能只改一處）；
   確認 Day 2 的三乘數框架、Day 3 的 20 個測試案例等貫穿全系列的伏筆前後說法沒有矛盾。
5. **Windows 開發環境部分**（Day 21、Day 22 的安裝指令、Day 24 的瀏覽器標頭）建議親自實測一次
   再發文——這兩塊查證時間點是 2026-08-17，是全系列裡最新查證的部分，但也是最容易被下一次
   官方更新影響的部分。
6. **確認 iThome 是否支援 mermaid**（Day 6），決定要不要刪除 mermaid 區塊。
7. **通過以上檢查後**：`git add` → commit（依內容分階段或一次 commit 皆可，取決於作者偏好）→
   `git push` → 依 iThome 鐵人賽時程排定每日發文。

### 已經埋好、且已在 Day 27-30 全數兌現的伏筆

- Day 2 的「三個乘數」框架 → 全系列貫穿使用，Day 28、Day 30 收尾。
- Day 3 的「20 個測試案例」→ Day 4、Day 5、**Day 27** 都重複使用，最後一次兌現已完成。
- Day 5、Day 7、Day 11 的 tokenizer 分工、Day 4/5 指向 Day 9/10、Day 6 指向 Day 13、
  Day 9 指向 Day 14、Day 12 指向 Day 15/16、Day 18 指向 Day 20 → **全數已兌現**。
- Day 12 提到 Compaction 摘要可能遺失細節 → Day 20 第六節已回頭連結這個風險。
- Day 13 提到的 Rate Limits API → Day 26 錯誤處理章節已接住 429/529 實際重試邏輯。
- Day 22 提到的 Agent teams（多個 Claude Code 實例協作）→ 未強行接進 Day 28，
  Day 28 選擇聚焦在單一請求流的三層架構，Agent teams 的多代理協作模式**留白，未展開**，
  如果之後要做系列外的延伸內容，這是一個現成的題目。

**如果之後要開新的系列或大幅改版本系列，每篇的固定流程可以直接沿用**：
讀 `CLAUDE.md` → 上網查證官方文件 → 依格式產出 → 估算篇幅 → 說明查證狀況。
這套流程跑過 30 次，沒有出現需要中途調整的情況。

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
