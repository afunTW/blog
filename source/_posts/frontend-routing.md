---
title: Learning F2E - Routing
date: 2016-11-28 00:00:00
categories: Frontend
tags: [routing]
---

> 紀錄學習 F2E 的瞬間

<!-- more -->

這次改一下寫文章的方式，把碎碎念的部份放到最後，直接先紀錄一下這篇想整理的東西

做網站的時候，除非是單頁靜態網頁，像是在 github.io 上放單頁履歷以外，大部份都有點擊某個連結然後更換顯示內容的需求，而這邊對於新手來說，第一關應該就是 routing

## Routing

如果有找過關於 NodeJS 資料的人應該對於 express 不陌生，在關於 express 的範例程式中，當你看完 http 模組之後緊接著就會是 routing 的教學

```js
 var express = require('express'); // 引入 express 
 var app = express();   
 
 // 在 '/' 路徑底下要做的事情
 app.get('/', function(req, res) {  
  res.send('Express routing at /');  
 });
 
 // 在 '/about' 路徑底下要做的事情
 app.get('/about', function(req, res) {  
  res.send('Express routing at /about');  
 });
 
 var server = app.listen(3000, function() {  
  console.log('Listening on port 3000');  
 });     
```

然而在許多人都關注的前端框架中，你多少也會看到 routing 的字眼，例如說 vue-router

```js
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// routing rule
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

const router = new VueRouter({
  routes: routes
})

const app = new Vue({
  router
}).$mount('#app')
```

在概念上，所謂的 Routing 很像是交通號誌，決定了資料的目的地，但是以上提到的這兩種又有蠻明顯的細節差異，對於新手很難一下子理解哪裡不一樣

## Server-side / Client-side

簡單一點的說法就是

- **server-side routing**：express 等...
- **client-side routing**：vue-router/react-router/ngrouter 等...

一個是在 server 做 routing 一個是在 client 做 routing，現在知道差別了，然後呢？
對於網頁來說到底實際上差別再哪裡？

舉例來說，假如我現在點擊了 `<a href="/about"></a>` 這個 HTML element

**Server-Side Routing**

1. 瀏覽器偵測到使用者點擊 `<a href="/about"></a>`
2. 瀏覽器發出 HTTP GET request 到 URL 去尋找 `/about`
3. Server 處理這個 request，然後回傳一個 response 回去（通常是 HTML 文件）
4. 瀏覽器砍掉舊頁面，換上剛下載好的新頁面

**Client-Side Routing**

1. 瀏覽器偵測到使用者點擊 `<a href="/about"></a>`
2. Client 端的程式（Routing library 之類的東西，如 vue-router）監聽到這個 event，然後發現這不是外部連結，就不會讓瀏覽器發出 HTTP GET request
3. Routing Library 會去更改顯示的 URL
4. Routing Library 根據 routing rule 去更改網頁狀態
5. 網頁根據狀態重新 render

因此有些人會得到一個結論，Server-side routing 適合拿來做多頁應用，Client-side routing 適合拿來做單頁應用（SPA），但真的要探討的話，重點應該在這兩者的處理方式所衍生的問題

## Others

雖然說只要花時間都學的起來，但老實說我覺得 Client-side Routing 相較之下比較難

而有些人會討論到 traffic 的問題，Server-side 是需要用到的時候再去跟 Server 要資料，可能會有較多的 response time，但 Client-side 則是一開始就把 Javascript 下載下來，但通常 Javascript 會是滿滿的商業邏輯，這又是否會造成一開始要 loading 比較久，等等的問題

另外也有人提到 Server-side 對於 SEO（搜尋引擎最佳化）比較好，因為比較好做爬蟲等等考慮上更多面向的問題，如果我有看到會再陸續在這邊補齊

剩下的我覺得都是 by case 的問題而已，例如說，這兩者是否能夠一起用？甚麼情況下要用甚麼樣的 Routing 方式？

這邊因為我自己還不太熟，所以目前只能整理筆記到這部份，歡迎其他前端大大補充

## Reference

- [When to use “client-side routing” or “server-side routing”?](http://stackoverflow.com/questions/23975199/when-to-use-client-side-routing-or-server-side-routing)
- [Deep dive into client-side routing](http://krasimirtsonev.com/blog/article/deep-dive-into-client-side-routing-navigo-pushstate-hash)
- [Client Routing (using react-router) and Server-Side Routing](http://stackoverflow.com/questions/28553904/client-routing-using-react-router-and-server-side-routing)

---

因為我本人對網站的東西真的是非常新手，雖然有在關注一些新聞，但以往都抱持著等前端生態再穩定一點才要投入時間學習。但漸漸的開始覺得不是這麼一回事，不管是甚麼樣的領域，先求有再求好，就算是手刻一個程式碼很髒很雜的網頁，也勝過知道一堆框架但沒寫過任何關於網頁的人強（以前的我）

最近剛完成的就是我覺得程式碼很髒很醜的網頁，HTML 幾乎全部手刻，Javascript 幾乎都是用 jQuery 在處理使用者事件而已，需要拿資料的時候再送 ajax 到 php，然後用 PDO 的傳統寫法跟 MySQL 拿資料

我知道這種寫法很蠢，但也知道其他的技術就是為了解決這些蠢事才衍生出來的，希望以後 Moli blog 也可以多介紹這部份，就算不是講技術，至少讓新手有關鍵字可以去學，然後大概知道 Modern Web design 會遇到的問題跟相關的解決方案有哪些選擇
 