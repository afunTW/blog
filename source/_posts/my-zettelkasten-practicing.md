---
title: 我所實踐的 Zettelkasten 卡片式管理法
date: 2021-07-10 17:50:12
updated: 2021-07-10 17:50:12
categories: Others
tags: [zettelkasten]
---
> 介紹我如何透過 Obsidian 與其他工具實踐與練習卡片式管理法
<!-- more -->

# 前言

根據[我所理解的 Zettelkasten 卡片式管理法](https://afun.tw/knowledge-graph/20210620-my-zettelkasten-understanding/) 想要進一步紀錄自己是如何實踐, 實際上只要工具可以支援反向連結 Backlink  跟視覺化的 Graph 都能滿足我實踐的工具, 目前是因為免費且 Community Plugin 的多樣性, 所以我才選擇 Obsidian

# 如何建立 Literature Note

Literature Note 是閱讀第三方資源後所作的讀書筆記, 同時基於 Zettelkasten 筆記的撰寫原則, 文章內應該包含以下內容

-   原文標題與連結: 確保隨時可以回朔原文
-   建立 Tags: 提供多樣性分類的功能性
-   我所認為的原文重點: 只擷取重要的資訊
-   以自己認知的內容重新敘述我所認為的原文重點: 將短期記憶轉變為長期記憶

同時推薦相關的工具可以讓我們快速建立 Literature Note

-   Obsidian Community Plugin
    -   [SilentVoid13/Templater: A template plugin for obsidian](https://github.com/SilentVoid13/Templater): 可以建立一個專門放 Template 的資料夾, 並透過各種變數讓你建立的時候自動帶入你需要的值, 對於建立文章結構非常有幫助
-   Chrome Add-On
    -   [TabCopy - Chrome 線上應用程式商店](https://chrome.google.com/webstore/detail/tabcopy/micdllihgoppmejpecmkilggmaagfdmb?hl=zh-TW): 可以客製化 URL 各種格式, e.g. Markdown Link, Obsidian Backlink, ...
    -   [Roam-highlighter - Chrome 線上應用程式商店](https://chrome.google.com/webstore/detail/roam-highlighter/mcoimieglmhdjdoplhpcmifgplkbfibp): 可以在瀏覽網路文章時畫重點, 並客製化成你需要的 Highlight Format

我們可以先建立 Literature Note Template
```
[原文標題與連結, 視情況稍微調整標題]

Tags: #LiteratureNote [其他 Tag]

## Highlight

[條列呈現原文重點]

## Perspective

[針對原文重點以自己的話重新敘述]
```

其中原文標題與連結可以透過上面提到的 TabCopy 快速產生, 然後我會根據以下狀況微調並作為筆記的 Title

-   簡體中文轉為繁體中文
-   如果有作者會在後面加上 `- [作者名稱]`
-   標題過長會擷取重點
-   標題太 General 會加上其他網頁提供的資訊

## 針對不同多媒體與格式調整

實際因為我們會從很多種渠道學習, 因此針對不同的多媒體其實都會對上述流程微調

- 針對 Local 端的檔案無法以 URL 形式對原文做連結
	- 將檔案放到 Obsidian Valult 中以資料夾管理
	- 透過反向連結取代 Markdown link 的功能
- 針對投影片類型的內容
	- Highlight 部份在 Prefix 加上頁碼 e.g. `p10` 代表 page 10
- 針對影片類型的內容
	- Highlight 部份在 Prefix 加上 Timestamp e.g. `00:01:35` 代表 1 分 35 秒
- 針對教學平台的課程內容
	- Highlight 根據課程大綱以 Section 做階層管理
- 針對電子書或是 Online Guide 等包含龐大內容的資訊
	- 針對小章節或是明確主題的內容拆為多個 Literature Note

# 如何建立 Permanent Note

Permanent Note 紀錄自己的想法, 同樣基於 Zettelkasten 筆記的撰寫原則, 文章設計會包含以下內容

- 建立日期 + 具有原子性的主題作為標題: 在 Zettelkasten 系統中作為 Unique Identifier 
- 紀錄學習目標: 幫助未來理解該主題內包含什麼內容與為什麼要建立這份筆記
- 建立筆記的日期: 未來如果更新的話會在更新處加上更新日期代表筆記的更動紀錄
- 建立 Tags: 提供多樣性分類的功能性
- 大綱: 根據思考邏輯架構建立 Outline
- 參考資料: 紀錄資料來源同時也方便回朔資訊

我們可以先建立 Permanent Note Template

```
[主題]

- Created: [[%Y-%m-%d]]
- Tags: #PermanentNote [其他 Tags]

## 學習目標

[為什麼紀錄這個主題與主題包含的內容]

## 大綱

## Reference

[如果有參考外部資訊, 條列呈現外部連結]

```

# 如何決定建立反向連結的策略

反向連結是 Zettelkasten 中的核心概念, 但根據不同的 Workflow 我會用不同的策略決定哪裡應該建立反向連結

- Bottom-Up: 不斷紀錄再不斷歸納
	- 建立 Literature Note 與 Permanent Note 的過程中, 我們可以直接針對有興趣進一步探索的內容**事先**建立反向連結
	- 當多個筆記都跟同樣的主題建立連結時就可以建立該主題作為新的筆記, 通常會是 Bibliographical Note 或是 Permanent Note
	- 通常這種反向連結的筆記標題都比較偏向關鍵字形式 e.g. `[[Zettelkasten]]`
- Top-Down: 不斷擴展再不斷紀錄
	- 有明確想要了解的主題, 以此建立 Permanent Note 為出發點, 不斷延伸產生新的 Literature Note 與 Permanent Note
	- 建立筆記的過程中透過關聯或是 Hashtag 發現其他筆記有想要紀錄的重點, 會直接針對**已經存在的筆記**做反向連結
	- 該反向連結的筆記標題明確指向一個主題
	- 大家以往都比較偏好 Top-Down 的作法

# Reference

- [用 Zettelkasten 筆記工具 Obsidian 打造「知識循環」](https://twgreatdaily.com/q840TnQBURTf-Dn5DpiL.html)