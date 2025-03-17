---
title: Crawling IMDB by API
date: 2018-01-22 00:00:00
categories: Data Engineering
tags: [python, imdb]
---

>   Demonstrate how to crawling IMDB data from scratch

<!-- more -->

這次是為了做「透過電影海報預測評分及分類」的 CNN task 所做的事前準備，我們必須取得至少上千至上萬筆資料，每一筆資料包含一部電影的海報（當作 input x）以及這部電影的評分與分類（當作 output y），雖然也可以純手工用 `requests` + `BeautifulSoup4` 或是 `Selenium` 或是 `Scrapy` 爬 IMDB 網頁上的資訊，但是有 API 可以用，像我這種懶人就會直接用 API 了。

稍微在網路上看了一下，許多人都會推荐使用 [OMDB API](http://www.omdbapi.com/)，在他的網站範例上 query 一次可以發現他的 meta data 真的很齊全，API 也很簡單，申請 Token 也很簡單。

**BUT**

我一直送 request 失敗直到重新檢視信件才發現原來要**有所貢獻**才能使用，雖然資料本來就有其價值，最低價的方案也才 $1 USD/month，但因為我這次只是一次性的，所以就放棄使用 OMDB API 了...

## 爬蟲過程

雖然我也有試著找過別人寫好的 Python library 來使用，像是 [imdb-pie](https://github.com/richardasaurus/imdb-pie)，但那不是我想要的東西，找著找著就發現果然有人做過([文章連結](https://www.johannesbader.ch/2013/11/tutorial-download-posters-with-the-movie-database-api-in-python/))，也因此又發現了另一個叫作 [TMDB API](https://www.themoviedb.org/) 的存在（雖然這個縮寫聽起來很像髒話 XD）。

### 第一步：申請帳號並申請 API Key

根據 [TMDB API DOC](https://developers.themoviedb.org/3/getting-started) 操作即可，申請帳號還蠻簡單的，但是申請 API KEY 的時候覺得有點麻煩要填很多資料，但其實我也沒有填真實資料，想辦法通過前端檢查就好了（誤）。

有了 API KEY 一切才能開始，請放心這些步驟比起手刻爬蟲還是相對輕鬆多了。

### 第二步：取得 imdbId

想要透過 API 快速的寫爬蟲, 有個可以對應的 unique ID 是非常重要的, 在 IMDB 的爬蟲中通常就是指電影海報本來在 IMDB 上的 ID

這邊也不需要爬蟲，去網路上稍微找一下就可以發現其實有一個 [MovieLens](https://grouplens.org/about/what-is-grouplens/) 的團隊有提供整理好的 metadata，雖然並不是包含所有電影的 metadata，但是這對我們的 task 並不造成甚麼影響，直接下載上面提供的 dataset 就可以了。

我這邊使用的是該研究團隊在 2015 釋出的 ml-20m dataset，包含兩萬多筆電影的資料以及兩千多萬筆評分的資料，這邊要補充一下該團隊的研究目標其實是推薦系統，所以他們的評分是他們研究中每位使用者對各個電影的個別評分，這邊需要自己整理資料取得該電影的平均分數。

### 第三步：整理 metadata

如上所述，我們必須將 ml-20m 裏面我們所需的資料都整理一下，這邊我使用的方式是透過 python 的 pandas 做處理：

```python
import pandas as pd

# 三份 ml-20m 中附上的 CSV 檔案：
CSV_MOVIE = 'movies.csv'
CSV_LINK = 'links.csv'
CSV_RATING = 'ratings.csv'

# 透過 pandas 讀檔，並且捨棄 row number 資訊：
df_movie = pd.read_csv(CSV_MOVIE).reset_index(drop=True)
df_links = pd.read_csv(CSV_LINK).reset_index(drop=True)
df_rate = pd.read_csv(CSV_RATING).reset_index(drop=True)

# df_movie 跟 df_links 根據 id 的欄位取交集
df = pd.merge(df_movie，df_links，how='inner'，on='movieId')

# 整理電影的平均評分並捨棄不需要的欄位
df_group_rate = df_rate.groupby(['movieId']，as_index=False).mean()
df_group_rate = df_group_rate.drop('userId'，axis=1)
df_group_rate = df_group_rate.drop('timestamp'，axis=1)

# 將所有資料整合成同一張 data frame
df = pd.merge(df，df_group_rate，how='inner'，on='movieId')
```

### 第四步：爬蟲

接下來參考上面提到的[文章連結](https://www.johannesbader.ch/2013/11/tutorial-download-posters-with-the-movie-database-api-in-python/) 了解 TMDB API 以及 imdbId 的 pattern 就可以直接對海報 url 送 request 然後存檔：

```python
import requests
import time
import random

from urllib.request import urlretrieve

APIKEY=<your API key>
URL = 'http://api.themoviedb.org/3/configuration?api_key={}'.format(APIKEY)

# 要透過 TMDB API 取得海報的話需要三種資訊：base url + size + relative path
# 透過 APIKEY 我們可以先取得圖片 API 的 base url 以及 size 的設定
# size 這邊我會選擇 size[-1] 也就是海報原始尺寸
resp = requests.get(URL)
config = resp.json()
base_url = config['images']['base_url']
sizes = config['images']['poster_sizes']

# 從我們上一步整理好的 data frame 中一行一行操作
for i，row in df.iterrows():
    save_filename = row['title']

    # 整理圖片 API 格式並取得圖片資訊
    img_req = 'http://api.themoviedb.org/3/movie/tt0{imdbid}/images?api_key={key}'
    img_req = img_req.format(imdbid=row['imdbId']，key=APIKEY)
    img_resp = requests.get(img_req)
    img_info = img_resp.json()
    
    # 取得海報欄位資訊並透過 urlretrieve 下載
    if 'posters' not in img_info:
        print('no poster: {}'.format(img_req))
        continue
        
    img_rel_path = img_info['posters'][0]['file_path']
    img_url = '{}{}{}'.format(base_url，sizes[-1]，img_rel_path)
    try:
        urlretrieve(img_url，save_filename)
    except Exception as e:
        print(row['title'])
        print(img_url)
    time.sleep(random.randint(1,2))
    
# 儲存 metadata
df.to_csv('metadata.csv')
```

---

## 後記

這算是這個任務最簡單的作法之一，只是送 request 的過程中還是會遇到 connection timeout 的問題，這邊我並沒有針對這個處理做重送的動作，因為資料量也才兩萬多筆而且也只有一次性，邊做其他事情邊檢查有沒有 timeout 就可以了。

這次是因為我們的需求只要一萬張以上的海報跟 metadata 資料，所以用這種方式即可，如果想要拿到最新的資訊可能就不能用 movie lens 的資料了，可能要再去找找看是否有人整理，或是就考慮真的付一點小錢去使用 OMDB（聽說這個以前是免費的？）。

爬蟲一直以來都只是一種手段，其實也沒有必要每次都把他當作一個專案來做，一定要自己去分析網頁或是一定要做到 async 很快的爬完資料，更何況一直以來都是建議遵循有 API 就用 API 的原則，不要蠻橫的硬幹對別人的 server 亂送 request XDDDD。
