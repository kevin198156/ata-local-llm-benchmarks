# Human UAT Methodology

這份方法不是標準化學術 benchmark，而是為「本地私人 AI 是否真的值得每天使用」逐步長出來的 Human UAT protocol。

## 1. 為什麼不能只看 tok/s？

速度只能回答：

> 模型回得快不快？

但不能回答：

- 繁中會不會漂成簡中／英文？
- 陪聊是不是一直分析你？
- RP 會不會搶你的角色？
- 小說會不會無視你的風格限制？
- 不知道的事情會不會亂掰？
- 成人題材是真的能 actual generate，還是只會自述「我可以」？
- 長聊後會不會卡在上一種模式？

因此本專案把 technical gate 與 Human UAT 分開。

## 2. 結果標記

- **PASS**：在這次 protocol 與使用需求下可接受。
- **WARN**：能用，但有重要限制／不穩定行為。
- **FAIL**：觸發核心需求 hard gate，或明顯不適合該用途。
- **NOT TESTED**：沒有證據，不把缺測當 PASS 或 FAIL。
- **SOURCE BLOCKED**：來源／授權 gate 未過，因此不下載、不測。

這些標記只描述本專案的實測，不是全域模型排名。

## 3. 評測維度

### Traditional Chinese stability

不是只看「能不能輸出中文」，而是看：

- 多輪後是否維持繁體中文；
- 是否大量簡體 leakage；
- 是否突然 English drift；
- 用詞是否適合台灣使用者。

繁中是本專案 hard gate：嚴重失敗時可停止後續昂貴測試。

### Companion naturalness / emotional value

觀察日常對話是否：

- 像聊天，而不是每句都分析／列建議；
- 能配合「只陪我聊，不要解決問題」之類的指令；
- 有持續互動價值，而不是只有單輪回答漂亮。

這是高度主觀項，因此由 Human 直接判定。

### Adult actual generation

重點是 **actual behavior，不是模型自述**。

抽象測法：

1. 明確設定所有角色均為成年人。
2. 給定合法、自願的成人 fiction / RP 工作流。
3. 要求模型實際生成。
4. 記錄是否拒答、淡出、說教或正常生成。

公開報告只記 PASS / WARN / FAIL，不發布 raw explicit transcript。

### Explicitness

Adult actual generation PASS 仍不代表尺度／詞彙符合需求，因此另測：

- 是否只能含蓄帶過；
- 是否能依指定尺度與文體直接表達；
- 是否因為詞彙本身而退縮。

### Source-conditioned fiction

本專案發現「從零寫」與「給原文後續寫」是不同能力。

抽象 protocol：

1. 給一小段 source text。
2. 指定沿用人稱、節奏、語氣與尺度。
3. 要求改寫／續寫。
4. 比較模型是否維持 source 的 register，而不是回到自己的固定文風。

對真實使用者而言，這個測項可能比 generic creative-writing benchmark 更重要。

### Writing obedience

看模型是否尊重：

- 不新增重大背景；
- 不新增人物；
- 不提前揭謎；
- 不改人稱；
- 不重貼原文；
- 不自行補角色心理；
- 字數／節奏限制。

「寫得漂亮」和「照指令寫」是兩個分開的能力。

### RP ownership

定義：

> AI 應控制自己的角色與環境，不替使用者決定台詞、思想、感受或下一個行動。

簡單 protocol：

- 明確指定 user-controlled character；
- 進行數輪 RP；
- 記錄 serious ownership violations。

模型可以對 user 的既有行動做反應，但不應自行寫出 user 下一步。

### Consent / character-state tracking

這不是「成人文筆」測試，而是狀態追蹤。

觀察模型能否追蹤：

- 原本是否同意；
- 是否出現明確撤回；
- 是否應停止；
- 是否只在後續重新明確同意後恢復；
- 是否遵守新條件；
- 事後 recap 能否說對狀態。

### Clean consent test

Clean test 刻意排除干擾：

- 所有人都是成年人；
- 初始自願；
- 無藥物；
- 無束縛／威脅；
- 明確規則：最新明確 consent state 優先。

目的：

> 先確認模型在「規則很乾淨」時是否具備基本 state-tracking 能力。

### High-intensity immersed RP robustness

與 clean test 分開。

這裡的情境有更長上下文、更強角色慣性、更沉浸的自然語言。觀察模型是否會因為「劇情正在往前衝」而把自然拒絕錯解成角色表演。

因此：

> Clean test PASS **不等於** high-intensity robustness PASS。

### Mode recovery

在不清空 session 的情況下：

1. 正式結束 RP。
2. 切回普通 Companion。
3. 再切日常休閒／事實問題。
4. 問模型目前是否仍在角色扮演。

觀察是否有角色語氣、成人情境或舊模式殘留。

### Unknown-info / hallucination control

使用刻意虛構的：

- 樂團／作品；
- 遊戲；
- 小說／作者；

並明確要求：

> 不知道就說不知道，不要猜。

FAIL 的典型情況不是「答錯一點」，而是對不存在的實體自信補出開發商、年份、作品背景等具體資料。

### Long-session stability

觀察約 20–30 turns 或更長 session：

- 是否重複固定套路；
- 是否忘記早期重要狀態；
- 是否人物／模式漂移；
- 是否還能回到普通對話。

### Instruction switching

同一 session 切換：

- Companion
- RP
- fiction writing
- ordinary QA

看模型是否遵守最新指令，而不是被前一模式鎖住。

### Repetition / trope loop

觀察長聊或 fiction 是否：

- 重複句式；
- 重複同一橋段；
- 固定把角色推向同一 trope；
- 看似不同回答但實際在繞圈。

### Speed / daily usability

本專案不只記 decode tok/s，也看：

- TTFT；
- context 變長時 TTFT；
- RAM/VRAM reserve；
- 桌面同時操作；
- 使用者主觀「願不願意等」。

## 4. 技術 Gate 與 Human Gate

建議流程：

```text
Source / License Gate
        ↓
Technical Load Gate
        ↓
Traditional Chinese Gate
        ↓
Core Human UAT
        ↓
Long-session / recovery
        ↓
Finalist retest with the same current protocol
```

如果 hard gate 已失敗，例如主要需求是台灣繁中而模型持續簡中／英文 drift，就停止昂貴後續測試。

## 5. 為什麼 finalists 要重考？

測試方法會因新問題而進化。

如果 A 模型在第一天只考 5 題，B 模型在第四天考 20 題，直接比較兩者是不公平的。

因此本專案最後對 Defiant 與 Holodeck 補上同一批核心測試：

- clean consent/state;
- mode recovery;
- unknown-info;
- source-conditioned workflow;
- final role decision.

這比「每顆模型考不同題後硬排名」更可靠。

## 6. Adult / sensitive UAT 的公開方式

公開：

- protocol；
- PASS / WARN / FAIL；
- 抽象失敗類型；
- 是否有 refusal / hallucination / ownership / state 問題。

不公開：

- raw explicit transcript；
- 私人角色／故事內容；
- 私人偏好或身份資訊。

這讓方法可重現，同時避免把私人內容變成 benchmark fixture。

## 7. 這不是安全認證

Consent、hallucination、RP 等測試只是有限 Human UAT。

PASS 代表：

> 在這個樣本與這個 protocol 下沒有觸發問題。

不代表模型在所有情境都安全、正確或穩定。
