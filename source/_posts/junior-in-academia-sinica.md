---
title: 中研院研發替代役的新手紀錄
date: 2016-11-18 00:00:00
categories: Life
tags: []
---

>  紀錄剛開始在中研院當研發替代役的心得

<!-- more -->

說來有點小丟臉，都畢業了還在新手村。

現在在中研院當研發替代役，從開始工作到現在兩個多月以來，其實常常很後悔大學時期沒學好，不務正業到處做其他事情，所以乾脆藉這個機會記錄一下，在外面工作可能會遇到的雷（當然不是指遇到慣老闆的那種）（那種的私下聊）。

先稍微紀錄一下目前碰過的東西。

## 第一個月 - MIS

一個實驗室想當然爾會有很多 server，想當然而會有很多 user，想當然爾會有人用 Windows / macOS / Linux，想當然爾會有專門跑運算的跟專門存資料的 server，想當然爾會有 public / private network 的問題，想當然爾還會有很多開發環境的問題。

所以我花了一個月稍微研究了：

- NIS（集中控管使用者帳號的系統）
- Samba（跨系統溝通的軟體）
- LAMP（Linux + Apache 2 + MySQL + PHP）
- R studio server（透過網頁管理 R 開發環境）
- mount/sshfs
- 其他疑難雜症（？！）

當然這些我原本都沒操作過，甚至有些根本沒聽過，甚至有些找資料還找不太到。

## 第二個月 - Website Maintainer

是的，突然就變成一堆跟網站有關的工作。原本還想試試看 Cacti，Nagios，等等之類的。

然後網站的多樣性應該不用多提了，但我至少從零經驗開始做了：

- 架設 Wordpress
- R（我目前主要是用來整理 raw data）
- phpMyAdmin（跟資料庫的互動介面）
- SQL（操作 MySQL）
- PHP（拿來塞資料到資料庫跟後端程式）
- Apache 2 相關設定（包含 `.htaccess`）
- HTML 5 + Bootstrap + JavaScript + jQuery 寫簡單版 dashboard
- Vue 2.0 做 <abbr title="Single Page Application">SPA</abbr>（這邊嘗試過但沒時間完成，算失敗）

當然還有比較偏向雜事的部份：

- PHPMailer（透過 PHP 寄送大量客製化信件）
- 透過 PHP 產生連結，配合客製化寄信，取得 client 資料 （絕對不是黑黑）（雖然有翻 [DEVCORE Blog](http://devco.re/blog/)）
- 進機房

以上

---

真的做過這些，才覺得不管你原本想要做甚麼類型的職位，前端也好，後端也好，甚至是 Devops，RD之類的都好，這些都是多少要懂一點的程度，所以才會說**不是從頭到尾操作過 Apache(server), MySQL(DB), PHP(back-end), JavaScript(front-end) 就能稱做 full-stack engineer**

其實這兩個月最累的是沒有前輩帶，只能私訊問離職前輩，還有私訊纏著 MOLi 的各位大大，尤其是我認識的前端工程師應該都被我騷擾過...所以是時候回饋點東西了（？

如果對上面我列的東西有比較感興趣的，我有空可以想想看怎麼分享出來。
 