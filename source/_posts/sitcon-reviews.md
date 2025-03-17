---
title: 2017 SITCON reviews
date: 2017-05-18 00:00:00
categories: Life
tags: [conference, sitcon]
---

> 側寫 2017 SITCON 的紀錄

<!-- more -->

![](2017-sitcon.jpg)

## 攤位

### MOLi

今年的 SITCON 我反而沒有花那麼多時間去聽演講, 因為 **NCNU** X **MOLi** X **OSSPlanet** 今年變成贊助商攤販了！ 身為潛水成員, 平常都只在 Telegram 出現, 這種時候總該要出來露個臉

![NCNU X MOLi X OSS Planet 攤位照](2017-sitcon-moli.jpg)


> MOLi (Maker's Open Lab for Innovation) 創新自造者開放實驗室 ([詳細介紹請點此進入](https://moli.rocks/))

老屁股們好久不見, 有人從前端跨資安跟 Blockchain, 有人專案還沒討論就可以自己把網站做完, 有人準備要投論文結果是我們實驗室協辦的 conference, 有人是持續跳坑, 每次聊天除了屁話其實都還是有種我在跟大大們對話的感覺

> OSS Planet ([詳細介紹請點此進入](http://ossplanet.net/))

順便在這邊幫忙推廣一下, 這是一個可以幫 Open Source 做 Free hosting 的一個服務, 目前已經有蠻多比較大的服務在上面做 hosting 或是 mirror

### Ethereum Taipei meetup

這是今年有特別注意到的攤位, 因為區塊練跟虛擬貨幣的東西我一直沒有甚麼接觸, 僅僅知道這個東西的存在以及一堆人在玩　Bitcoin 跟挖礦而已, 這方面的 pickup 應該是今天我收穫最多的東西, 詳細的內容應該會再後面演講的部份一並提到

順便紀錄一下 @focaaby 大大給我的文章, 用白話文解釋區塊鍊並提及一些區塊練的問題, 我以一個完全沒接觸過區塊鍊資訊的人看完文章, 覺得蠻平易近人而且簡潔扼要, 希望給一些想要了解的人參考看看

> reference: [facebook page](https://www.facebook.com/notes/shi-cho-cha/%E6%AF%8F%E6%97%A5%E9%96%B1%E8%AE%80%E5%BF%83%E5%BE%97-20170317/10155112963374483/)

## 演講

剛剛上面也有提到, 今年沒有參加太多場演講, 也不打算講太多演講的技術細節

### 十分鐘認識 MMR (PineApple Kao)

MMR (Montgomery Modular Reduction),　主要是用來解決大數在做 mod (也就是算除法的餘數) 造成的龐大運算資源問題, 透過一些數學證明, 省略除法步驟達到 mod 的目的

可能大家一樣都有意會到這是實際上非常需要注意的問題, 首先不可能讀一個很大的數字再來做運算, 大部分的人都是做 mod 讓數字變小, 然後再對這個數字做運算, 之後再重複這個過程, 因此實際上耗費的運算資源是相對龐大的, 但至少讓大數運算變成一個可以實作的解法之一

數學細節這邊暫且不提, 但是我覺得內容以十分中的時間來說是相對有邏輯並且實際, 唯一可惜的就是 R1 當時人數多到有點誇張

### 所以我說那個 ML 入門其實真的沒想像中那麼難 (Terry Cheong)

可能因為自己有稍微接觸 Machine Learning 的東西, 這場演講反而是抱持著不知道十分鐘可以說些甚麼的心態去的

雖然有稍微介紹到何謂 Machine Learning, 但是我覺得解釋起來好像還是不夠清楚, 到目前為止我覺得直接參考 Tom M.Mitchell 的比較適合

> A computer program is said to learn from experience E with respect to some class of tasks T and performance measure P if its performance at tasks in T, as measured by P, improves with experience E - [wikipedia](https://en.wikipedia.org/wiki/Machine_learning)

然後講者說明了 Machine Learning 的過程, 以及一些常見的演算法

* decision tree
* random forest
* gradient descent

最後講解實作技術使用的是 scikit-learn, 概略描述建 model 的過程, 只是這邊發現講者頭影片上用的 gradient descent 是 stochastic gradient descient 但是卻沒有解釋這是甚麼東西, 果然以十分鐘的長度連 stochastic / batch / mini-batch 的概念都沒辦法完整描述完畢

紀錄一下講者補充的好玩套件: [r2d3](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/), 這是一個 R package, 主要是做視覺化的套件

感想就是果然十分鐘講這麼大的主題還是太短

### 和我簽訂契約成為區塊鍊魔法少女吧 (飛魚)

先不討論魔法少女跟庫洛魔法使大家有沒有聽過, 這場演講是我整天聽下來最好的演講, 從投影片到演講方式到內容深度完全適合當概論

這場主要是在說 Ethereum 跟智慧合約的部份, 前面稍微比較了 Bitcoin 跟 Ethereum 的差別, Ethereum 感覺更容易用現在人類的交易模式解釋, 透過挖礦或是購買得到足夠的手續費 (Gas), 然後才可以透過區域鍊做交易的動作, 透過這個機制的實作來限制交易量, 也才因此讓 Ethereum 再實作上可以使用迴圈, 所以如果你有足夠多的手續費... 

> 如果你有很多錢, 這世界就會為你而轉動 - 飛魚

在上面 @focaaby 大大分享給我的文章裏面也有提到, 區塊鍊基本上就是透過讓大多數人認可達到信賴的技術, 這邊在定義何謂「大多數」的認可時候, 我覺得也是一個蠻有趣的思考面

另外說到區塊練上的資料是無法被更改的時候, 講者提到的小插曲也蠻讓我印象深刻的

> 2/14 情人節當天有挖礦工司提供一個服務, 讓情人可以把我愛你的字句存入區塊鍊中, 因為區塊鍊的資料是無法被更改的, 所以就算以後你不愛她了, 也無法改變你曾經愛過他的事實 - 飛魚

我真的滿喜歡這種有趣的小八卦

### 開發學校雲端服務的奇技淫巧 (馬聖豪)

又是一個因為學校網站相關服務太難用, 所以乾脆自幹一個的作品, 感覺在各大校園都會有這個問題, 從首頁設計到選課系統, 總是會有些服務沒辦法滿足學生

雖然演講題目是雲端服務, 但其實就是說明一個第三方服務的架構跟流程設計, 學生提供帳密給該服務, 該服務再把這些個人資料送 requests 到學校主機取得學生的資料, 然後用更好看的 UI 跟 UX 設計呈現給學生使用

剛好最近做了一個爬蟲專案 (這個有機會再寫成文章), 所以讀了一些瀏覽器如何跟網站做互動的資料, 這場演講我才聽的懂, 內容包含動態網頁分析, header 內容, 常見的擋 bot 作法, 只是我到現在還在想破解 CAPTCHA 的方式除了講者的方法, 感覺簡單的英文或數字 CAPTCHA, 建個簡單的圖像辨識 + Machine Learning Model 似乎更好一些？只是不知道會不會有點殺雞用牛刀的感覺

這場演講比較適合有實作過一些東西的人參加, 內容都有說到痛點

## Lightning talk

每次 conference 的 lightninh talk 真的都蠻精彩的, 這次的亮點之一就是最近再網路上瘋狂轉載的文章「[台大資工短時間上榜心得](https://www.ptt.cc/bbs/graduate/M.1489646585.A.95D.html)」神人本人親自現身, 4~5 天準備台大資工考試, 然後正取一這件事情本身本來就是很狂的事情了, 看到本人演講更有種他真的是奇葩的感覺

另外, 我其實一直都很喜歡像是 PCC 或是 Denny 那種統計總結今年年會的一些數據, 當然還有那種 UCCU 的演講方式

---

## 總結

這次跟以往相比我花了比較多時間跟人討論, 感覺這樣 pickup 的東西更多, 然後 BlockChain 果然還是我今天最有感的領域

議程我比較偏好長度 40 分鐘以上的內容, 十分鐘比較像是 Lighting talk 的演講方式, 但講者常常會一直想要補充更多東西導致東西講不完

然後聽到 SITCON X HK 的心得感想覺得有點可惜, 也開始好奇香港那邊的資訊圈發展情況是如何？

> <small>最後感謝 SITCON 小編讓我轉載 Cover</small>
