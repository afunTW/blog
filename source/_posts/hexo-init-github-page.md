---
title: Hexo - 基本設定與 GitHub Page 發佈流程
date: 2020-11-14 00:00:00
updated: 2020-12-10 01:29:31
categories: Blog
tags: [hexo, github]
---

> 透過 hexo 建立基礎的部落格並透過 CI 建立發布流程

<!-- more -->


# 基本指令與介紹

Hexo 是透過 Node 產生靜態網頁的部落格框架，建議可以邊操作邊跟著官方文件 ([doc](https://hexo.io/zh-tw/)) 先在 local 建立 hello world blog

## 安裝

因為觀望 hexo 一陣子了，所以安裝的時候如果沒有其他擔心的，可以都升到建議版本以上

-   Node 版本 10 以上 → 為了使用 `npm` 安裝相關套件
-   Git → 為了發佈與更新部落格系統
-   hexo-cli (安裝完 Node 之後才能透過 npm 下載) → 透過 hexo 指令與部落格系統互動
    -   `npm install -g hexo-cli`

## 建立部落格

因為安裝了 `hexo-cli` 所以基本上所有操作都會透過 `hexo CMD` 來達到目的

-   `hexo init` 建立 hexo 基本的檔案結構
-   `hexo s` 在 local 端啟動 server，可以在 [localhost](http://localhost) 看到生成靜態網頁後的部落格系統
-   `hexo g` 生成靜態網頁
-   `hexo cl` 清除已經產生過的靜態檔案
    -   在設定過程可以開著 server 就好，基本上只有在 GitHub Page 上設定時會用到
-   `hexo d` 發佈部落格系統

### 最初的盲區：初始化 hexo 檔案結構失敗 (GitHub Page repo)

一開始我的目的就是打算把 hexo 架在 GitHub Page 上

因為 `[afunTW.github.io](<http://afuntw.github.io>)` repo 中已經存在其他檔案，hexo 初始化發現不是空的資料夾所以爆錯

```bash
$ git clone <https://github.com/afunTW/afunTW.github.io>
$ cd afunTW.github.io
$ hexo init // Error
```

其實後來發現要發佈到 GitHub 上面可以透過 `hexo d` 發佈，不用走以前熟悉的 `add - commit - push` 的方式，而是直接建立一個空資料夾並且在裏面初始化就好

```bash
$ mkdir myblog
$ cd myblog
$ hexo init
```

## 了解基本觀念

基本指令其實不難理解，接下來大家應該會慢慢研究 `_config.yaml` 裏面的參數怎麼設定才是想要的，然後選喜歡的主題做下載，網路上許多的文章會以比較多人在用的 `next` 當作說明參考

-   建議主題選擇比較多人維護而且有持續更新的
-   hexo 基本分為新增 page 與 post
    -   page 是頁面 e.g. archive, about, category, tag, ... ，可以在 `scaffolds` 內檢視
    -   post 代表部落格文章，可以在 `sources` 內檢視
-   `sources` 內的東西會被完整保留，考慮到會推到 GitHub 上面可以善加利用

# 建立發佈到 GitHub Page 的完整流程

## 選擇一：直接 push 靜態網頁

```bash
$ hexo cl
$ hexo g
$ hexo d
```

透過 `hexo` 指令可以在 local 端生成靜態文件並且根據 `_config.yaml` 內的 git 設定直接 push

但這樣的結果會是直接把靜態網頁都 push 到 `master`

-   優點：過程簡單好理解
-   缺點：原始檔案與設定檔只存在 local

## 選擇二：push 原始檔案與設定檔，透過 travis 發佈

>   為了避免裝置更換與其他意外遺失設定檔，後來選擇這種方式發佈

概念上與[官方文件](https://hexo.io/zh-tw/docs/github-pages)中的敘述相同，我會將流程設定為

-   把原始檔跟設定檔 push 到 `master` branch

-   Travis 服務偵測到 

    ```
    master
    ```

     有更新，啟動 CI

    -   根據原始檔跟設定檔生成靜態網頁
    -   將靜態網頁發佈到 `gh-pages`

-   GitHub Page 呈現 `gh-pages` 結果，大功告成

### GitHub 相關設定

參考 hexo 提供的[範例 repo](https://github.com/hexojs/hexo-starter)

-   設定 `.gitignore`
-   hexo 主題 `theme` 通常會是其他人的 github repo，將其設定為 `.gitmodules`
-   設定 `Profile > Settings > Application > Travis > Configure`
    -   啟動 Travis 支援
    -   設定 Travis 限制存取，並將 `[USERNAME.github.io](<http://username.github.io>)` 的 repo 加入存取名單
-   設定 `Profile > Settings > Developer settings > Personal access tokens`
    -   產生 token 給 Travis 使用，權限設定 `public_repo`

![Setting travis on github](setting-travis-on-github.png)

![generate access token](generate-access-token.png)

### Travis 相關設定

根據上述設定權限後，到 Travis 設定 token 為環境變數 (這邊取名為 `GH_TOKEN`)

![Travis setting access token](setting-access-token-on-travis.png)

之後在 repo 中加入 `.travis.yml` 設定 Travis 相關參數

```yaml
sudo: false
language: node_js
node_js:
    - 10 # use nodejs v10 LTS
cache: npm
branches:
    only:
        - master # build master branch only
before_script:
    - npm install
    - hexo clean
script:
    - hexo generate # generate static files
deploy:
    provider: pages
    skip-cleanup: true
    github-token: $GH_TOKEN
    keep-history: true
    on:
        branch: master
    local-dir: public
```

設定完整並 git push 到 master branch 之後，就會觸發 Travis CI 流程，成功的話到 [afunTW.github.io](http://afuntw.github.io/) 就會看到網頁結果，大功告成
