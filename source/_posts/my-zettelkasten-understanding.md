---
title: 我所理解的 Zettelkasten 卡片式管理法
date: 2021-06-20 17:23:00
updated: 2021-06-20 17:23:00
categories: Others
tags: [zettelkasten]
---

> 介紹什麼是卡片式管理法, 解決過去我所遭遇到作筆記的疑惑

<!-- more -->

# 前言

年初的時候被同事推坑 [Obsidian](https://obsidian.md/) 這套工具, 一開始跟 Typora 做比較, 沒有感覺到一定要換工具的痛點, 但後來才發現被取代的是我作為筆記工具的 Notion

雖然一開始使用並沒有感受到太多優點, 但隨著 Community Plugin 的豐富化, 以及許多提及 Obsidian 適合搭配 Zettelkasten 卡片式管理法使用的文章, 趁著疫情長期在家的期間開始整理筆記跟學習怎麼紀錄

# Zettelkasten 與其他筆記方式的差異

參考 [How One German Scholar Was So Freakishly Productive](https://writingcooperative.com/zettelkasten-how-one-german-scholar-was-so-freakishly-productive-997e4e0ca125) 文章可以發現, 大家對於做筆記是不斷遇到痛點再隨著時間演進的, 作筆記的目的主要有兩點: (1) 查詢 (2) 延伸思考, 以下會透過這兩件事情來比較差異

**假設我們把所有學過的東西都記在同一份筆記上?**

![](https://miro.medium.com/max/2780/1*_b2Of3MLwLrPRETuyK8lbA.png)

這種方式沒有索引, 非常難找到我們需要的內容, 不管是對於查詢或是延伸思考都是充滿雜訊且缺乏管理

**假設把所有學到的知識透過資料夾分類管理?**

![](https://miro.medium.com/max/2780/1*nH7-yxNfV0In00hRUNOw6Q.png)

自然而然地, 大家開始習慣用資料夾幫筆記分類, 對於查詢來說我們可以透過選定資料夾來縮小搜尋範圍, 但是對於關聯到其他筆記這件事情卻仍然很難有效率的查看

另外, 實際在幫筆記分類時也經常會遇到一個問題是: 不知道筆記應該放在哪個資料夾?

假設我有兩個資料夾分別叫做雲端系統與資料工程, 當我今天的筆記是紀錄如何在雲端系統建立資料處理的架構時, 就會不知道應該怎麼分類? 於是就會經常性的調整資料夾結構

這邊的問題是我們嘗試對包含複雜關聯的知識事先分類, 大部分的知識都沒辦法輕易歸屬在一種類別內, 另外事先分類也是相當於對該知識定型, 很難後續進一步與其他筆記做跨領域關聯

**假設我們進一步針對資料夾搭配 Tag 管理?**

![](https://miro.medium.com/max/2780/1*MIsuUFw0E0JjbBMZgSNKnw.png)

Tag 的發明打破了關聯的隔閡, 我們可以透過工具在跨專案跨資料夾跨領域的方式做關聯

但是當我們的筆記累積了幾年後, Tag 隨著筆記有爆炸性成長, 最終 Tag 會失去一套管理的規則, 變得跟沒有分類是一樣狀態

## Zettelkasten - 針對筆記做反向連結

![](https://miro.medium.com/max/2780/1*gxyKEtyW6Ms_1v7Sm4N2Rw.png)

後來 Luhmann 接著引入反向連結的機制, 發明了 Zettelkasten 卡片式管理法, 實際上對筆記與筆記之間建立連結, 不再單純依賴 Tag 做筆記之間的關聯, 用這種方式建構出來的知識系統是 Graph, 就像現在的網際網路一樣, 每篇筆記相當於每個網頁資源, 透過一個特有的 Identity 作為索引, 然後在筆記內夠過反向連結彼此關聯

![我的知識圖譜現況 2021-06-20](20210620_KnowlegdeGraph.png)

> Note: Luhmann 當時使用這套方式的時候是非常傳統使用小卡片用多個盒子裝起來來實踐

# Zettelkasten 筆記的類型與使用的時機

Zettelkasten 的系統內, 其實更進一步把筆記的類型定義出來, 大多數人都參考自 **How to take smart notes** 這本書, 其中我根據我的 workflow 主要使用以下兩種

- Literature Note, 閱讀外部資源後所寫的筆記
- Permanent Note, 紀錄自己的想法或是從多個 Luterature Note 整理完的概念

其他還有幾項我實際上還沒使用過
- Fleeting Note, 紀錄突然想到的想法
- Biblioographical Note, 根據某種概念將多筆 Literature/Permanent Note 關聯在一起的筆記
- Project-Based Note, 專門為了解決某一個問題的筆記

## Zettelkasten 筆記的建立原則

同樣在 [How One German Scholar Was So Freakishly Productive](https://writingcooperative.com/zettelkasten-how-one-german-scholar-was-so-freakishly-productive-997e4e0ca125) 中有提到相關概念, 但我這邊列出一些我覺的特別重要的

- 筆記需要具有原子性: 也就是一篇筆記只包含一個主題, 太多主題的話會失焦, 導致知識沒有辦法被妥善組織
- 一篇筆記內的內容需要可以獨立解釋主題內容: 就算原文消失也要有足夠的資訊可以解釋想要表達的內容
- 筆記需要被連結到其他筆記: Zettelkasten 的觀念是假如沒有建立連結, 未來一定會遺忘, 筆記就相當於沒有價值, 只是浪費時間
- 筆記需要用自己的話重新解釋: 用自己說的話才代表自己理解的內容

這邊你可以看到一個觀念是我們只紀錄最小單位的筆記 (原子性), 但實際上這個定義還是有點模糊, 比如說我應該定義多大的 Scope 代表一個主題?

## Zettelkasten 筆記的原子性

![Anatomy of Zettel](https://zettelkasten.de/introduction/anatomy.png)

實際上一個具有原子性的筆記結構需要包含以下三點

- Unique Identifier, 提供該筆記一個唯一關聯的 ID, 就像網頁的 URL 一樣
- Body Content, 筆記內容
- Reference, 當紀錄內容包含外部資源時就可以紀錄資賴來源, 為了方便隨時回去看原文, 假設原文的內容豐富, 我們也可以進一步把原文紀錄成 Literature Note, 再跟筆記做反向連結

原作者 Luhmann 實際上有一套自己設計的 Index 方式, 但我們不一定要使用, 目前我則是把 timestamp + 主題作為 Unique Identifier

不過這邊又延伸出一個疑問是: 什麼筆記內容才能當作是「一個主題」？

這邊參考 [盧曼卡片盒筆記法介紹(Introduction to the Zettelkasten Method) • Zettelkasten Method](https://zettelkasten.de/introduction/zh/) 中所提到的, 基本上還是取決於你希望建立怎樣的知識系統? 假設你希望建立的是個人想法組合的知識系統, 那所謂的主題就代表你對一件事情的理解

# Reference

- [Zettelkasten - How One German Scholar Was So Freakishly Productive | by David B. Clear | The Writing Cooperative](https://writingcooperative.com/zettelkasten-how-one-german-scholar-was-so-freakishly-productive-997e4e0ca125)
- [基於 Obsidian 的卡片盒筆記法實踐 - 呂立青](https://blog.jimmylv.info/2020-06-03-zettelkasten-in-action/)
- [如何正確使用 Zettelkasten 筆記法連結 - 朱騏 | PM的生產力工具箱 | Medium](https://medium.com/pm%E7%9A%84%E7%94%9F%E7%94%A2%E5%8A%9B%E5%B7%A5%E5%85%B7%E7%AE%B1/%E5%A6%82%E4%BD%95%E6%AD%A3%E7%A2%BA%E4%BD%BF%E7%94%A8-zettelkasten-%E7%AD%86%E8%A8%98%E6%B3%95-4ff20303ec3e)
- [盧曼卡片盒筆記法介紹(Introduction to the Zettelkasten Method) • Zettelkasten Method](https://zettelkasten.de/introduction/zh/)