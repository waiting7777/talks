---
theme: seriph
title: '換到新模型後，給 AI 的規則檔要怎麼整理？'
info: |
  ## 換到新模型後，給 AI 的規則檔要怎麼整理？
  拆解 Anthropic 公開的 prompt-audit skill
author: Waiting
keywords: ai,prompt,skill,claude-code
exportFilename: prompt-audit
drawings:
  persist: false
transition: slide-left
mdc: true
---

# 換到新模型後<br/>給 AI 的規則檔要怎麼整理？

## 拆解一份公開的 prompt-audit skill

<div class="abs-br m-6 text-sm opacity-60">
依據 anthropics/skills 的 prompt-audit.md（Apache 2.0）
</div>

<!--
大家好。上週分享 Claude Code 團隊怎麼審 AI 寫的程式，今天算續集。
講的是一份公開的 markdown，誰都能拿去給自己的 AI 工具用。
-->

---

# 為什麼講這題

<div class="mt-8 text-xl leading-loose">

1. **最近換到新模型**
2. 聽到一個說法：**拿掉限制型的句子，AI 反而更強**
3. Claude Code 有指令可以掃規則檔，背後的方法是**公開的**

</div>

<div class="mt-10 p-6 bg-blue-900/10 rounded-lg border border-blue-900/30 text-lg">

"One of the patterns we saw is that we were over-constraining Claude."

<div class="text-right text-base opacity-60 mt-3">— Thariq，上週分享的訪談</div>

</div>

<!--
上週訪談裡 Thariq 說，他們以前把模型限制得太多。
剛好最近換了模型，我就去找有沒有方法可以檢查自己的規則檔。
-->

---

# 指令在哪裡

| 入口 | 掃什麼 |
| --- | --- |
| `/doctor prompt-audit` | CLAUDE.md、skills、agents、commands |
| `/claude-api prompt-audit` | 程式裡的 system prompt、tool 描述 |
| 直接讀 `prompt-audit.md` | 任何規則檔，包括 AGENTS.md |

<div class="mt-8 text-lg">

文件本身寫明：同時適用 API 應用和 coding agent 的設定檔。

**用什麼工具都能用**，把這份 markdown 給你的 AI 讀，請它照著檢查。

</div>

<!--
Claude Code 內建成 doctor 的子指令。
用其他工具的，例如 opencode，直接把這份文件給 AI 讀就好，它就是一份規則。
-->

---

# 一句話

<div class="mt-16 text-3xl text-center leading-relaxed">

找出**不再適合**的句子

<span class="opacity-60">不是把檔案變短</span>

</div>

<div class="mt-12 text-lg text-center">

不適合新模型 ・ 不適合現在的專案 ・ 彼此矛盾

</div>

<!--
文件反覆強調：目標不是變短。
有些句子是為舊模型加的，換了模型反而會讓它用過頭。這才是要找的。
-->

---

# 八步流程

<div class="grid grid-cols-2 gap-8 mt-6 text-lg leading-loose">
<div>

0. 定範圍和目標模型
1. 盤點所有送進模型的文字
2. **查來歷**：git blame
3. **刪除規則**：一題判斷

</div>
<div>

4. 掃四組反模式
5. 報告：位置、原文、信心
6. 提出修改，**不自己改**
7. **前後驗證**

</div>
</div>

<div class="mt-8 text-lg text-center opacity-80">

刪掉一句只是假設，不是結論

</div>

<!--
重點三個：查來歷、刪除規則、驗證。
查來歷是問每句強調的話當初在防什麼錯，新模型還會不會犯。
-->

---

# 刪除規則：只問一題

<div class="mt-12 text-3xl text-center">

模型自己會不會**已經知道**？

</div>

<div class="grid grid-cols-2 gap-8 mt-12 text-lg">
<div class="p-4 rounded bg-green-900/10 border border-green-900/30">

**留：只有作者知道的**

產品、環境、品質標準、限制的理由

</div>
<div class="p-4 rounded bg-orange-900/10 border border-orange-900/30">

**候選刪：模型本來就會的**

「要準確」「要仔細」、為舊毛病加的補丁

</div>
</div>

<!--
背景永遠不是贅字。會被刪的是模型本來就會做的事，還有為舊模型毛病加的補丁。
-->

---

# 會被抓出來的句子

| 反模式 | 舊寫法 | 新寫法 |
| --- | --- | --- |
| 施壓語氣 | `CRITICAL: You MUST...` | `Use this tool when...` |
| 被取代的鷹架 | think step by step | 刪掉，新模型會先想 |
| 規定過細 | 判斷題寫 STEP 1、2、3 | 寫目標和怎麼驗證 |
| 單一範例 | 一個標準答案 | 幾個不同範例 |
| 輸出編排 | 「最多五點」「120 字內」 | 一起拿掉 |
| 互相矛盾 | 兩個檔案講法不同 | 留一個地方講 |

<div class="mt-4 text-base opacity-80">

「盡量」「如果可以」也會被抓：新模型照字面讀，會當成可以不做。

</div>

<!--
講大聲會用過頭，講太客氣會被當成可以不做，兩邊都要調。
-->

---

# 不准刪的，跟要刪的一樣重要

<div class="mt-6 text-xl leading-loose">

- 背景永遠不是贅字
- 贅字 ≠ 長，不能拿字數當理由
- 刪除、權限、合規，**保留精確步驟**
- 新模型還會犯的錯，禁令留著
- 什麼都沒找到是**正常結果**

</div>

<div class="mt-8 text-lg text-center opacity-80">

只會叫人刪的檢查，最傷最認真照做的人

</div>

<!--
這一頁是我覺得這份文件寫得最好的地方。它不是叫你刪，是教你分辨。
-->

---

# 拿我自己的 skill 試

| 原句 | 判斷 |
| --- | --- |
| 「每個檔案都要看，一個都不能跳」 | 留，補理由：大 PR 會挑著看 |
| 「最多報 5 則」 | **待討論**：產品限制還是舊習慣？ |
| 「找不到就跳過，不要編造」 | 低信心，先留，之後測 |
| 「先跑腳本定範圍」 | 留：固定流程要精確步驟 |
| 「以下類別不刪，就算不確定」 | 改寫：補上為什麼寧可多報 |

<div class="mt-4 text-base opacity-70">

照文件規則手動判斷，不是工具跑出來的報告。

</div>

<!--
五句裡只有一句明確要改。大部分不是刪，是補上理由。
最多報 5 則這條最有意思：如果是因為太多意見沒人看，那是產品限制，可以留；如果只是怕舊模型囉嗦，就該拿掉。
-->

---

# opencode 自己也這樣做

opencode 會依模型選 system prompt，給 Claude 的那份這樣變：

| 時間 | 給 Claude 的 prompt | 全大寫強調（MUST、NEVER…） |
| --- | --- | --- |
| v1.19 | 專用的，105 行 | 8 個 |
| v2 初期 | 專用的，53 行 | 7 個 |
| 2026-09-05 | **整份刪掉**，改用共用的 15 行 | 0 個 |
| 2026-09-11 | 加回一段：註解密度跟著周圍的程式碼 | 0 個 |

<div class="mt-4 text-base opacity-80">

先刪到最短，看到哪裡會出問題，再用一段話加回去。跟 prompt-audit 第七步一樣。

</div>

<!--
這是 opencode 開源 repo 裡看得到的紀錄，PR 47447 和 48444。
不是說他們照 prompt-audit 做，是兩邊各自走到同一個做法。用 opencode 的人，背後的 prompt 已經是這樣了。
-->

---
layout: center
---

# 回去可以做什麼

<div class="mt-6 text-xl leading-loose">

- [ ] 找出全大寫的 MUST、NEVER，問：有沒有寫理由？
- [ ] 搜「think step by step」「盡量」「如果可以」
- [ ] 跑 `/doctor prompt-audit`，或把文件給你的 AI 讀
- [ ] 刪之前想一個測試，刪之後跑一次

</div>

<!--
最小的一步：打開自己的規則檔，搜全大寫的 MUST 和 NEVER，看每句有沒有寫為什麼。
-->

---

# 來源

<div class="mt-8 text-lg leading-loose">

- prompt-audit 全文：anthropics/skills（Apache 2.0）<br/>
  <span class="text-base opacity-70">github.com/anthropics/skills/blob/main/skills/claude-api/shared/prompt-audit.md</span>
- 指令說明：Claude Code CHANGELOG<br/>
  <span class="text-base opacity-70">github.com/anthropics/claude-code/blob/main/CHANGELOG.md</span>
- opencode 的 prompt 變化：anomalyco/opencode PR #47447、#48444<br/>
  <span class="text-base opacity-70">github.com/anomalyco/opencode/pull/47447</span>
- 「over-constraining」：Simon Willison 訪談 Cat 與 Thariq，2026-07-21<br/>
  <span class="text-base opacity-70">simonwillison.net/2026/Jul/21/cat-and-thariq/</span>

</div>

<!--
謝謝大家。全文 237 行，有興趣可以直接讀，寫得很清楚。
-->
