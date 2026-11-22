# 換到新模型後，給 AI 的規則檔要怎麼整理？ {#prompt-audit}

**找出「不再適合」的句子，不是把檔案變短。** Anthropic 把這套檢查寫成一份公開的 skill，Claude Code 也內建成 `/doctor prompt-audit`。

- 新模型更聽話、照字面做，為舊模型加的施壓語氣和步驟反而讓它用過頭。[](source:audit)
- 判斷刪不刪只問一題：模型自己會不會已經知道？只有作者知道的背景永遠留。[](source:audit)
- 不准刪的清單跟要刪的一樣重要。危險操作要留步驟，新模型還會犯錯的地方要留禁令，每條限制要留理由。[](source:audit)
- 刪掉一句只是假設，改前改後各跑一次，退步就用最短的說法加回去。[](source:audit)

@answer scope="依據 anthropics/skills 的 prompt-audit.md（Apache 2.0）與 Claude Code 更新紀錄 · 2026-11"
@meta locale=zh-TW style=clean
@source audit title="anthropics/skills：skills/claude-api/shared/prompt-audit.md" url=https://github.com/anthropics/skills/blob/main/skills/claude-api/shared/prompt-audit.md
@source changelog title="Claude Code CHANGELOG：/doctor prompt-audit" url=https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
@source oc-remove title="anomalyco/opencode PR #47447：remove legacy Anthropic prompt（2026-09-05）" url=https://github.com/anomalyco/opencode/pull/47447
@source oc-append title="anomalyco/opencode PR #48444：append comment guidance to Anthropic system prompt（2026-09-11）" url=https://github.com/anomalyco/opencode/pull/48444
@source simon title="Simon Willison：A Fireside Chat with Cat and Thariq from the Claude Code team（2026-07-21）" url=https://simonwillison.net/2026/Jul/21/cat-and-thariq/

## 為什麼講這題？ {#why-this-talk}

@steps id=why-steps
| w1 | 最近換到新模型 | 最近換到新模型，以前寫的規則檔要不要跟著調？ |
| w2 | 聽到一個說法 | 拿掉一堆限制型的句子，AI 反而更強。上週分享的訪談裡，Claude Code 團隊也說過他們以前把模型限制得太多。 |
| w3 | 找到一個指令 | Claude Code 有指令可以掃規則檔。方法寫在一份公開的 markdown 裡，誰都能拿去給自己的 AI 工具用。 |

所以今天拆開這份文件：它怎麼判斷哪句該刪、哪句不能刪。最後拿我自己寫的 skill 草稿試一次。

## 指令在哪裡？ {#where}

| 入口 | 掃什麼 | 適合誰 |
| --- | --- | --- |
| `/doctor prompt-audit`（也可打 `/checkup prompt-audit`） | CLAUDE.md、skills、agents、commands | 用 Claude Code 的人 |
| `/claude-api prompt-audit` | 程式裡的 system prompt、tool 描述、請求設定 | 自己寫 API 應用的人 |
| 直接讀 `prompt-audit.md` | 任何規則檔，包括 AGENTS.md | 用其他 coding agent 的人 {good:誰都能用} |

文件本身就寫明，它同時適用 API 應用和 coding agent 的設定檔（CLAUDE.md、AGENTS.md、skill、自訂指令）。[](source:audit) [](source:changelog)

## 它怎麼跑？ {#flow}

@workflow id=audit-flow title="八步：盤點、判斷、報告、驗證"
| scope | 定範圍和模型 | frontend |
| inventory | 盤點文字 | backend |
| blame | 查來歷 | backend |
| classify | 刪除規則 | security |
| scan | 掃四組反模式 | backend |
| report | 報告 | frontend |
| diff | 提出修改 | frontend |
| verify | 前後驗證 | database |
| scope > inventory | |
| inventory > blame | |
| blame > classify | |
| classify > scan | |
| scan > report | |
| report > diff | 只放中高信心 | emphasis |
| diff > verify | |

@steps id=flow-steps
| f1 | 定範圍和模型 | 不問使用者，從 repo 推斷。假設寫在報告最上面，錯了重跑就好。 | audit-flow | scope |
| f2 | 盤點文字 | 找出所有會送進模型的文字：規則檔、skill、tool 描述、範例。設定檔和金鑰不讀。 | audit-flow | inventory |
| f3 | 查來歷 | 對每句強調或禁止的話跑 git blame。問兩題：它當初在防什麼錯？新模型還會犯嗎？ | audit-flow | blame |
| f4 | 刪除規則 | 只問一題：模型自己會不會已經知道？ | audit-flow | classify |
| f5 | 報告 | 每條寫位置、原文、對到哪個模式、為什麼過時、信心高中低。 | audit-flow | report |
| f6 | 提出修改 | 一條一塊，讓人挑著收。不自己改檔案。 | audit-flow | diff |
| f7 | 前後驗證 | 刪除是假設不是結論。一次改一個，退步就用最短的說法加回去。 | audit-flow | verify |

## 哪些句子會被抓出來？ {#patterns}

| 反模式 | 舊寫法 | 新寫法 |
| --- | --- | --- |
| 施壓語氣 | `CRITICAL: You MUST use this tool when...` {caution:用過頭} | `Use this tool when...` |
| 被功能取代的鷹架 | 「think step by step」、要它在輸出裡寫推理 | 刪掉，新模型本來就會先想，深度用 effort 控制 |
| 規定過細 | 判斷題寫成 STEP 1、2、3，一長串「不要」 | 寫目標、限制和怎麼驗證 |
| 單一範例 | 只放一個標準答案 {caution:會照抄} | 幾個刻意不同的範例，標明僅供參考 |
| 輸出編排 | 「120 字以內」「最多五點」「每三步回報一次」 | 一起拿掉 |
| 化石 | 舊模型名字、為一次事故加的永久規則 | 寫現在的規則，不寫歷史 |
| 互相矛盾 | 兩個規則檔對同一件事講法不同 | 留一個地方講 |

「盡量」「如果可以」也會被抓：新模型照字面讀，會當成可以不做。[](source:audit)

@callout level=note title="tool 描述是反方向"
tool 的描述常常要加字，參數意思、限制、什麼情況會失敗都要寫清楚。拿掉的只有語氣和範例。

## 哪些句子不准刪？ {#keep}

- [ ] 背景永遠不是贅字：產品、環境、品質標準、限制的理由。太短的 prompt 只會得到平庸的輸出。
- [ ] 贅字不等於長，不能拿字數當刪的理由。
- [ ] 危險操作保留精確步驟：刪除、權限、合規流程。
- [ ] 新模型還會犯的錯，那條禁令留著。判斷標準是錯誤會不會重現，不是句子長得像禁令。
- [ ] 什麼都沒找到是正常結果，空的修改勝過硬湊的修改。

文件說，一份只會叫人刪的檢查，最傷最認真照做的人。[](source:audit)

## 拿自己的 skill 試一次 {#my-skill}

我照這份文件的規則，手動檢查自己寫的 code review skill 草稿：

| 原句 | 對到的模式 | 判斷 |
| --- | --- | --- |
| 「每一個檔案都要看過，一個都不能跳」 | 施壓語氣 | 留，但補理由：大 PR 會挑著看是現在還會發生的錯 {good:有理由} |
| 「最多報 5 則」 | 輸出編排 | 要討論：是產品限制（太多意見沒人看），還是舊模型的習慣？ {caution:待決定} |
| 「找不到就跳過，不要編造」 | 規定過細（do not hallucinate） | 低信心，先留著，之後用測試確認 |
| 第 1 節「先跑腳本定範圍」 | 規定過細 | 留：這是流程固定的操作，要精確步驟 {good:不准刪} |
| 「以下類別不刪，就算你覺得不太確定」 | 施壓語氣 | 改寫：補上為什麼權限和安全寧可多報 |

五句裡只有一句明確要改。大部分不是刪，是補上理由。

@callout level=warning title="Tips"
這張表是我照文件規則手動判斷的，不是工具實際跑出來的報告。真正的檢查還要做最後一步：改前改後各跑幾個舊 PR，看結果有沒有變差。

## opencode 也這樣做嗎？ {#opencode}

opencode 會依模型挑不同的 system prompt。開源 repo 記錄了給 Claude 那份怎麼變：

| 時間 | 給 Claude 的 prompt | 全大寫強調（MUST、NEVER…） |
| --- | --- | --- |
| v1.19 | 專用的，105 行 | 8 個 |
| v2 初期 | 專用的，53 行 | 7 個 |
| 2026-09-05 | 整份刪掉，改用所有模型共用的 15 行 {good:最短} | 0 個 |
| 2026-09-11 | 加回一段：註解密度跟著周圍的程式碼 | 0 個 |

先刪到最短，看到 Claude 每個函式都加註解，再用一段話加回去。這跟 prompt-audit 第七步一樣，[](source:oc-remove) [](source:oc-append) 但 opencode 沒說它是照這份文件做的。用 opencode 時，背後的 prompt 已經是這樣，自己寫的 AGENTS.md 也能照同一個方向檢查。

## 回去可以做什麼？ {#takeaway}

- [ ] 挑一份自己寫給 AI 的規則檔，找出全大寫的 MUST、NEVER、CRITICAL，每句問：有沒有寫理由？
- [ ] 搜「think step by step」「盡量」「如果可以」，換新模型後這些最先過時。
- [ ] 用 Claude Code 的跑 `/doctor prompt-audit`；用其他工具的，把 prompt-audit.md 給你的 AI 讀，請它照著檢查。
- [ ] 刪之前先想一個測試，刪之後跑一次。

方法的全文在 anthropics/skills，[](source:audit) 「以前把模型限制得太多」這句出自上週分享的訪談。[](source:simon)
