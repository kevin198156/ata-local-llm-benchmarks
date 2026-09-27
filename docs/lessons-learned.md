# Lessons Learned

## 1. 「能跑」只是第一關

GTX 1660 Ti 6GB 可以跑很多比原本想像中更大的模型。

但真正淘汰模型的常常不是 OOM，而是：

- 繁中漂移；
- 角色 ownership；
- 寫作不聽指令；
- factual hallucination；
- 長聊後模式殘留；
- 等待時間太久；
- RAM 安全餘裕太低。

所以「成功載入 GGUF」跟「成功做出私人 AI」中間還有很長一段。

## 2. 大模型不是免費升級

這次實機看見：

- 8B：很快；
- 9B：品質／速度／資源最好平衡；
- 12B / 14B：可以跑，但速度、RAM、長 context TTFT 明顯變重；
- 27B：technically feasible，但 ~2.7 tok/s + 極低 RAM reserve 不符合 daily-use 目標。

如果目標是每天聊天，等待成本本身就是品質的一部分。

## 3. 9B Q4 是這台機器的 sweet spot

這不是 universal recommendation。

它只代表：

> GTX 1660 Ti 6GB + 32GB RAM + Windows + llama.cpp + 本專案需求下，9B Q4 最接近「夠聰明、夠快、夠穩」。

這比單純追更大參數量更有實際價值。

## 4. Uncensored ≠ Adult actual generation

模型名稱可能包含：

- Uncensored
- Abliterated
- Heretic
- RP
- NSFW

但這些都不是 Human UAT 結果。

本專案真的碰過名稱很「自由」的模型，在 explicit adult actual generation 時仍直接拒絕。

因此：

> 不問模型「你能不能」；直接在合法、成年人、自願的抽象測試條件下看它實際做什麼。

## 5. 模型自述 ≠ 模型行為

讓模型列出「我有哪些限制」看起來很方便，但後來實測發現：

- 說可以，不一定真的生成；
- 說不行，某些上下文又可能生成；
- 自述常常只是當下 prompt 下的一段文字。

因此 capability 必須由 behavior test 決定。

## 6. 從零生成與 source-conditioned rewrite 是兩種能力

同一顆模型可能：

- 從零寫作偏保守／含蓄；
- 一旦給 source text，卻能很好地跟上語氣、尺度與節奏。

對實際 fiction workflow 來說，「沿著我的原文寫」可能比「請你自由創作」更重要。

因此 benchmark 應該貼近真實工作流。

## 7. Creative imagination 有雙面性

Holodeck 最值得記錄的一個案例：

- 在 fiction：願意補細節、推情節，常常是優點；
- 在 factual QA：同一傾向可能讓它對不存在的實體自信編出具體資料。

所以「創作能力很好」不能直接推論「適合當事實型 assistant」。

## 8. 寫得好和聽話不是同一件事

很多 creative model 可以寫得很流暢，但會：

- 多加人物；
- 多加背景；
- 提前揭謎；
- 自己補角色心理；
- 重貼 source；
- 把使用者角色也一起寫掉。

如果使用者是在共同創作，**writing obedience / RP ownership** 本身就是品質。

## 9. Clean consent 與 immersed RP robustness 要分開

在規則乾淨、情境簡單時會正確停止，不代表長篇沉浸 RP 裡也一定不被情節慣性帶走。

因此本專案把：

- Clean consent / state tracking
- High-intensity immersed RP robustness

分開記錄。

這也是為什麼 Holodeck 最後可以是：

> clean PASS，但 high-intensity WARN。

## 10. Hard gate 可以省很多時間

Deckard 技術速度很好，但繁中 hard gate FAIL。

所以後續成人、長聊、consent 等測試全部停止。

如果你的核心需求是繁中，沒有必要因為模型其他地方「可能很強」就把整套 UAT 跑完。

## 11. Integration bug 可能被誤認成模型問題

Spoomples 的案例提醒：

system prompt、chat template、reasoning parser 等整合設定會改變實際輸出。

因此合理順序是：

> 先證明 runtime / template / request path 正常，再做品質判決。

不然很容易把 integration bug 當成模型品質。

## 12. Benchmark 會進化，所以 finalists 要重考

一開始根本沒有：

- mode recovery；
- fake-entity hallucination；
- clean vs high-intensity consent；
- source-conditioned fiction；

後來才發現它們很重要。

所以最終 Defiant / Holodeck 又接受新版共同測試。

這比拿早期五題的分數去跟後期二十題的分數相比公平得多。

## 13. 不一定需要唯一 Champion

最後得到兩個角色：

- Defiant：Main Companion
- Holodeck：Adult / Fiction Specialist

這對實際使用比「綜合總分第一名」更有意義。

模型可以替換，工作流也可以手動切換；初期甚至不需要自動 router。

## 14. Raw logs 不等於應該公開的證據

私人模型評測可能包含：

- 成人內容；
- 私人角色；
- 個人偏好；
- 真實聊天脈絡。

公開分享不需要把這些搬上 GitHub。

更好的做法是：

> 公開 protocol + structured result + failure type；raw evidence 留本機。

這個 Repo 就採這個原則。
