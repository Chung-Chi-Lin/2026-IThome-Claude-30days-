# 【Day 19】System Prompt 怎麼寫？角色設定的正確姿勢

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

Day 17 骨架的第①塊，我只丟了一句「角色設定放 system prompt」就跳過了。今天把它補完整。

先破除一個常見的偷懶做法：很多人寫 system prompt 只寫一句「你是一位專家」，然後期待 Claude 從此變得專業。這句話不是沒用，但**它給模型的資訊量很低**——「專家」是誰、專精什麼、要對誰說話、用什麼標準判斷好壞，全部沒講。今天要講的是怎麼把這一塊寫得有實際作用，而不是流於形式的裝飾句。

> 本篇 system prompt 相關的驗證內容，於 **2026 年 8 月 14 日**對照 [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) 官方文件查證。部分段落屬於基於官方範例的合理推論，會在文中明確標記。

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

## 一、System 是獨立參數，不是對話的一部分

先確認一個結構上的事實：在 Messages API 裡，`system` 是**跟 `messages` 平行的獨立參數**，不是塞在對話陣列裡的一則訊息：

```python
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=1024,
    system="你是一位樂於助人的程式設計助理，專精 Python。",   # 獨立參數
    messages=[
        {"role": "user", "content": "如何依 key 排序一個字典串列？"}
    ],
)
```

這個結構本身就在暗示一種分工：**`system` 放的是這次對話期間大致穩定不變的設定，`messages` 放的是實際往來的對話內容。** 這跟 Day 9 快取策略提過的「穩定內容在前、變動內容在後」是同一種思路的延伸——system prompt 天生就適合放那些「這一整段對話都適用」的規則，而不是這一輪才出現的具體問題。

## 二、一句話就有感：角色設定的效果

官方對角色設定的效果講得很直接：

> "Setting a role in the system prompt focuses Claude's behavior and tone for your use case. Even a single sentence makes a difference."

**在 system prompt 裡設定角色，能聚焦 Claude 的行為與語氣，而且哪怕只有一句話都有感。** 官方給的範例：

```python
system="You are a helpful coding assistant specializing in Python."
```

這句話沒有長篇大論，但它同時做了三件事：**限定領域**（程式設計，而不是泛用助理）、**限定專精**（Python，不是所有語言都同等擅長）、**定調語氣**（helpful，友善協助的口吻）。

## 三、為什麼「你是一位專家」常常沒用

這是本篇最核心的問題，也是需要**誠實標記**的一段：以下的分析是基於「角色設定如何提供有效資訊」這個角度的推論，官方文件並沒有針對「泛用角色 vs 具體角色」給出正式的逐字比較說明。

回頭看官方那句範例——「specializing in Python」——它包含的是**可以被拿來做判斷依據的具體資訓**：領域是什麼、擅長什麼、語氣怎樣。而「你是一位專家」這句話**沒有提供任何可以被拿來做判斷依據的資訊**：專家於什麼？服務誰？用什麼標準判斷輸出好不好？

一個簡單的判準：**問自己「如果我拿掉這句角色設定，Claude 的行為會不會有任何具體改變？」** 如果答案是「不會」，這句角色設定大概率是裝飾性的，不是功能性的。「你是一位專家」拿掉前後，模型的行為通常沒有明顯差異；但「你是一位資深稅務顧問，服務對象是不熟悉稅法的一般民眾，回答時要避免專業術語」拿掉前後，行為差異會很明顯——因為後者給了三個可以直接影響輸出的維度：專業領域、目標讀者、表達限制。

**讓角色設定變得有效的做法，是把它從「頭銜」換成「頭銜 + 標準」**：

```text
❌ 你是一位專家
✅ 你是一位資深稅務顧問，服務對象是不熟悉稅法的一般民眾，
   回答時避免專業術語，遇到不確定的部分要明確告知使用者需要諮詢正式管道
```

## 四、System Prompt 不只設定身份，也設定行為傾向

官方在 Tool use 章節裡的一個範例，展示了 system prompt 除了「你是誰」之外的另一個用途——**設定 Claude 面對模糊指示時的行為傾向**。如果你希望 Claude 更主動、預設直接動手而不是只給建議：

```text
<default_to_action>
By default, implement changes rather than only suggesting them. If the user's intent is
unclear, infer the most useful likely action and proceed, using tools to discover any
missing details instead of guessing.
</default_to_action>
```

如果你反而希望它更保守，遇到不確定就先問而不是直接動手：

```text
<do_not_act_before_instructions>
Do not jump into implementation or change files unless clearly instructed to make
changes. When the user's intent is ambiguous, default to providing information, doing
research, and providing recommendations rather than taking action.
</do_not_act_before_instructions>
```

這兩段都放在 system prompt，而且都用了 Day 18 教的 XML 標籤把整段規則包起來——**這正是「角色設定」跟「行為規則」可以在同一個 system prompt 裡並存的示範**：角色定義「你是誰」，行為規則定義「你遇到模糊情境時該怎麼反應」。

## 五、用詞強度要拿捏，不是越強硬越有效

這是一個容易被忽略、但官方明確提出的重點。很多人寫 system prompt 習慣用力過猛——「CRITICAL」「你必須」「絕對不可以」，覺得語氣越強、模型越會照做。官方在講新世代模型時給了相反的提醒：

> "Claude Opus 4.5 and Claude Opus 4.6 are also more responsive to the system prompt than previous models. If your prompts were designed to reduce undertriggering on tools or skills, these models may now overtrigger. The fix is to dial back any aggressive language. Where you might have said 'CRITICAL: You MUST use this tool when...', you can use more normal prompting like 'Use this tool when...'."

**新世代模型對 system prompt 的反應比舊模型更敏感**，如果你的 system prompt 是照著舊模型的「要用力強調才有效」的習慣寫的，換到新模型上反而可能**過度觸發**——例如工具被呼叫得比預期更頻繁。修法是把「CRITICAL: 你必須在……時使用這個工具」這種語氣，調整回比較平常的「在……時使用這個工具」。

這跟 Day 15 提過的「system prompt 引導語句對措辭很敏感」是同一類提醒——**強硬程度不等於有效程度**，寫 system prompt 時值得先假設模型會認真照做，再依實際測試結果調整強度，而不是預設要用最強硬的語氣才安全。

## 六、System Prompt 也能用來取代舊的 prefill 技巧

Day 17 沒提到的一個實務場景：如果你以前用「預填 assistant 回應」的方式跳過開場白（例如強制回應以「Here is the summary:」開頭來避免模型講一堆客套話），新世代模型（4.6 之後）已經不支援這種預填方式了。官方給的替代方案就是寫進 system prompt：

```text
Respond directly without preamble. Do not start with phrases like
"Here is...", "Based on...", etc.
```

這也是 system prompt 的另一種用法——**不是設定身份，是設定輸出的起手式規則**，跟 Day 18 提過的「格式指示符」屬於同一類技巧，只是放的位置換成了 system 而不是每次 user 訊息裡都重複寫一次。

## 七、多輪對話裡，system prompt 該不該跟著變

一個實務上常見的問題：對話進行到第 10 輪，使用者的需求好像變了，該不該順勢改一下 system prompt？

回頭看 Day 9、Day 14 都提過的規則：**system prompt 是快取前綴的一部分，改動它會讓整段對話的快取失效**，效果跟 Day 14 提過的「跨請求改變 effort 會打斷快取」是同一類問題，只是這次動的是角色設定而不是 effort 值。

實務上的判斷原則：**如果角色設定本身需要因為對話內容而變（例如從「客服助理」變成「技術支援專員」），這通常代表你其實在處理兩個不同的任務，值得考慮開新對話（Day 8 技巧三），而不是在原本的對話裡硬改 system prompt。** 如果只是這一輪的具體要求不一樣（例如這次要簡短一點），該調整的是 Day 15 提過的「per-message steering」——把引導語句加在這一輪的使用者訊息裡，而不是動 system prompt 本身。**System prompt 的穩定性,本身就是它跟 user 分工的核心價值。**

## Before / After：從裝飾性角色到功能性角色

**❌ Before：裝飾性角色，拿掉沒差**

```python
system="你是一位專業的行銷顧問。"
```

**✅ After：功能性角色，附帶可執行的標準**

```python
system="""你是一位專精 B2B SaaS 產品的行銷顧問，服務對象是新創公司的行銷負責人。
撰寫文案時避免誇大不實的用詞（例如「業界唯一」「顛覆性」），
每個賣點都要能對應到具體的功能或數據，語氣專業但不生硬。"""
```

> Before 版本拿掉整句話，模型的行為八成不會有明顯變化——這就是它是裝飾性角色的證據。After 版本裡的每一句話都在限縮模型的行為空間：**領域**（B2B SaaS）、**讀者**（新創行銷負責人）、**用詞限制**（不誇大）、**論證標準**（每個賣點要有依據）、**語氣**（專業不生硬）。拿掉任何一句，輸出都會有可觀察的差異——這才是有效的角色設定。

## 本篇自我挑戰

- **今日挑戰**：檢查你目前所有的 system prompt，對每一句做本篇第三節的測試——「拿掉這句話，行為會不會有具體改變？」把答案是「不會」的句子，改寫成有明確標準或限制的版本。

- **反思**：「你是一位專家」這句話的問題，其實不是它錯，是它**沒有提供資訊**。這跟我們日常溝通很像——跟同事說「請你專業一點處理這件事」，跟說「請依照 A、B、C 三個標準處理，遇到不確定的部分先跟我確認」，後者才是真正能被執行的指示。你有沒有在其他場合也說過「聽起來很到位、實際上沒有資訊量」的指示？

## 總結

今天把 Day 17 骨架的第①塊補完整：**System prompt 是獨立於對話之外的參數**，適合放這次對話期間大致穩定的設定；**角色設定哪怕一句話都有感**，但前提是這句話要提供可以被拿來做判斷依據的具體資訊，而不是空泛的頭銜；**System prompt 也能設定行為傾向**（主動 vs 保守）跟輸出起手式，不只是身份；以及一個容易被忽略的提醒——**新世代模型對 system prompt 更敏感，用詞強度不用像過去那樣刻意加重。**

第三節的推論部分——「你是一位專家為什麼沒用」——是基於官方角色範例邏輯的合理延伸，建議你發文前再想一次是否需要補充更多第一手案例佐證。

**本日關鍵字回顧**

- **System 參數**：Messages API 裡獨立於 `messages` 的欄位，適合放對話期間穩定不變的設定。
- **一句話就有感**：官方對角色設定效果的原文結論，前提是這句話包含具體、可影響行為的資訊。
- **功能性角色 vs 裝飾性角色**：判準是「拿掉這句話，行為會不會有具體改變」（本節推論，非官方逐字定義）。
- **行為傾向設定**：`default_to_action` / `do_not_act_before_instructions` 展示 system prompt 除了身份，也能定義面對模糊情境的反應方式。
- **用詞強度拿捏**：新世代模型對 system prompt 更敏感，官方建議從強硬措辭「dial back」回正常語氣。

明天進入第三階段的收尾——**怎麼降低 Claude 的幻覺**。允許它說不知道、要求先引用再回答，把 Day 18 學到的 XML 標籤技巧用在防止瞎掰上。

**Day 20，降低錯誤輸出的實用做法。**
