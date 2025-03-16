---
title: Download gdrive big file with curl
date: 2018-08-04 13:44:06
updated: 2020-12-09 13:44:06
categories: Data Engineering
tags: [gdrive, crawling, curl]
---
>   每個人的條件都不一樣, 所以才有了 Workaround solution

<!-- more -->

# 依賴 GitHub 的解法

在 Deep Learning 的專案中, 為了測試或是 demo 都會提供事先訓練好的 pre-trained models, 而這些 models 往往 size 都是動輒 > 100 MB.

>    reference: [What is my disk quota? - User Documentation](https://help.github.com/articles/what-is-my-disk-quota/)

在 GitHub 的官網中可以看到單一文件最大只允許 100 MB, 因此建議如果有比較大的檔案或是 binary file, 可以使用 git-lfs 將大型檔案存在其他地方

每個帳號在 GitHub.com 上都有 1 GB 的空間可以使用, 透過 Setting > Billing > Git LFS Data 可以檢查剩餘空間

![](github-lfs-billing-verview.png)

這是一個好解法, 也是另外一個學習門檻, 如果沒有其他 server 可以使用, GitHub 所提供的 1 GB 對於儲存 model 來說真的只是杯水車薪, 尤其是當你的 deep learning 專案中有做 ensemble 需要將多個 model 集成的時候

# 透過 Google Drive 的 Workaround

說到足夠大的儲存空間，應該會有很多學生想到的是以前在學校使用的 google drive (就算是個人使用的免費空間至少也有 15GB), 而這就是這次 workaround 的必要條件

上傳檔案並將其權限公開之後, 需要透過 API 才方便寫在 shell 中使用

>   [https://drive.google.com/uc?export=download&id=](https://drive.google.com/uc?export=download&id=$file_id)<FILE_ID>

Ubuntu 上大家經常使用的下載指令無非是 wget 跟 curl, 如果是針對 google drive 的工具也有人會推荐 [gdrive](https://github.com/prasmussen/gdrive)

說一點題外話, 其實在剛接觸指令的時候, 被教導取得 source code 或是文件請愛用 wget 下載, 不要使用複製貼上, 因為各種縮排或是 encoding 可能會被改變

```shell
# remember to replace the FILE_ID
$ wget https://drive.google.com/uc?export=download&id=<FILE_ID>
```
透過上面的指令下載, 結果當然是失敗 (不然就沒有必要寫這篇文章), 而原因其實跟我們上面提到的有關 (model size 動輒 > 100 MB), 而 Google 網站上在 Security 的部份有提到目前下載檔案如果檔案超過 100 MB 就無法掃描檢查該檔案是否有病毒

>    [Upload files to Google Drive - G Suite Administrator Help](https://support.google.com/a/answer/172541?hl=en)

實際上操作的時候就會跳出視窗詢問你是否確定要繼續下載, 所以你如果單純使用 wget 或是 curl 的話, 只會拿到該頁面的 HTML 檔案而已

![](check-virus-preview.png)

```shell
# remember to replace the FILE_ID
$ wget --no-check-certificate https://drive.google.com/uc?export=download&id=<FILE_ID>
```

過去透過 `--no-check-certificate` 似乎可以繞過檢查但嘗試後發現並不行

>   2019.04.11 Update: 感謝 Sean 指正，`--no-check-certificate` 只是不檢查 HTTPS 憑證有效性，跟掃毒沒關係

## 先說結論

```shell
download_from_gdrive() {
    file_id=$1
    file_name=$2
    
    # first stage to get the warning html
    curl -c /tmp/cookies \
    "https://drive.google.com/uc?export=download&id=$file_id" > \
    /tmp/intermezzo.html

    # second stage to extract the download link from html above
    download_link=$(cat /tmp/intermezzo.html | \
    grep -Po 'uc-download-link" [^>]* href="\K[^"]*' | \
    sed 's/\&amp;/\&/g')
    curl -L -b /tmp/cookies \
    "https://drive.google.com$download_link" > $file_name
}

# download_from_gdrive <FILE_ID> <OUTPUT_FILENAME>
```

說起來其實步驟不多也不難理解, 參考了 [StackOverflow](https://stackoverflow.com/a/32742700/4070887) 上的作法, 帶入爬蟲的概念用指令取得頁面上真正的下載連結送出第二次請求就可以成功下載

---

第一個步驟主要是先取得檔案, 透過 curl 的指令送出請求, 我們同時將 cookies 跟 HTML 都存在 tmp 中, 根據 Ubuntu 的設定不同, 存在該資料夾底下的檔案會在一段時間後被系統自動刪除

第二個步驟則是要從 HTML 中把下載連結 parse 出來, 首先第一個難關就是用 grep 寫 regular expression, 這邊是直接參考別人的寫法, 如果之後失效的話也可以先從這個地方開始檢查 (突然懷念起 BeautifulSoup4 的美好)

接著 sed 處理的只是單純的將 & 在 HTML tag 中的表示方式轉回一般符號的型式, 最後再附上 cookies 對實際下載連結送出請求即可

## 大功告成！