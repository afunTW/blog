---
title: Docker for deep learning primer
date: 2018-09-05 13:35:25
updated: 2020-12-09 13:35:25
categories: Data Engineering
tags: [docker, deep learning]
---
>   在深度學習專案中的第一次導入 Docker
>   
>   這邊並非 Docker tutorial，細節請右轉 http://lmgtfy.com/?q=Docker+tutorial

<!-- more -->

# 評估需求的正確姿勢

深度學習的專案中，撇除資料的討論（通常應該要有另外一個地方存放資料，e.g. DataServer）如果要把環境佈署到其他 Server 上，不外乎要看以下幾點

-   OS ( 目前習慣 Ubuntu 16.04 on Server)
-   CUDA + cuDNN
-   Python
-   Python Dependencies

隨著時間與版本更新，有些人希望可以嘗試新版本的功能，有些人則希望可以固定現在的版本讓專案穩定的繼續開發，以一般 Python 專案的角度而言，Python 套件可以撰寫 `requirement.txt` ，這樣 code 搬到其他 Server 後只要根據 `requirement.txt` 安裝套件就可以了

更進一步就是使用虛擬環境將專案環境隔開，有些人會使用 `conda` 或是 `virtualenv` + `virtualenvwrapper` ，但我目前的使用習慣是 `pipenv` ，除了可以紀錄在虛擬環境中安裝的套件以外，還可以進一步檢查套件與套件之間的依賴性以及區別出 dev 所需的套件 ( 會產生 `Pipfile` 與 `Pipfile.lock` )

>   reference: [pypa/pipenv](https://github.com/pypa/pipenv)

Python Dependencies 可以透過上面這些方式維護與獨立環境，Python 版本的話我目前則是使用 `pyenv` 在自己的家目錄底下安裝不同版本的 Python 版本，搭配 `pipenv` 使用的話基本上就可以把單純的 Python 專案獨立開來了

>   [pyenv/pyenv](https://github.com/pyenv/pyenv)

根據上面這些作法，一般專案應該是沒甚麼問題，但是 Deep Learning framework 的版本跟 CUDA / cuDNN 這些有非常強的依賴關係，如果 Server 上大家需要使用的 CUDA 版本號不同，或是未來需要在不同版本的 OS 測試才需要來考慮 docker 這個作法

# Docker + pyenv + pipenv

![Dockerfile](docker-file-demo.png)

## 環境配置

有些專案可能自己本身就包含不同的 Python 版本與多個虛擬環境 (e.g. pipeline 拆成多個環境)，所以我這邊的作法就乾脆在 docker 裏面使用 `pyenv` + `pipenv` 原因上面都有提到過了，因此 `Dockerfile` 主要需要撰寫的部份只有安裝 `pyenv` + `pipenv` 以及先行指定一個 Python 版本

另外有一個 `pyenv`的小優點是他會一並根據你要下載的 Python 版本，連同對應的 `pip` 都下載，只要在路徑中包含 `~/.pyenv/versions/3.5.2/bin` 就可以使用，而這個優點是可以降低 `pip` 的一些錯誤

```
Traceback (most recent call last): 
File "/usr/bin/pip3", line 9, in from pip import main

ImportError: cannot import name 'main'
```

## 新手錯誤

在撰寫 `Dockerfile` 的過程中算是不斷錯誤嘗試，有些事情第一次使用的話容易會忘記

-   default shell：假如你是習慣 `bash` 的話可以使用 `SHELL` 設定
-   `RUN` 在跑完後不會維持到下一行：寫在 `.bashrc` 再用 `source` 的方式通常不會達到你期待中的樣子
-   `PATH` 的更新可以透過 `ENV` 設定
-   透過 `pyenv` 下載的 `python3` 與 `pip` 可以透過路徑的更新取得，不用透過 alias

# 專案環境

前面有提到通常資料都會放在其他地方，在 docker 環境的話，我是直接使用 host volume 的方式掛進去，而 code 的部份就是在該專案中寫一份 `Dockerfile` ，以上面配置的 CUDA 環境的 images 為基礎將 code 複製進環境再包成該專案專屬的 image

## Optional setting

如果該專案持續在開發中，可以在 DockerHub 中設置 Automated build 與 GitHub 連動，沒有經過任何設置的話，當你 push commit 到任何一個 branch 就會直接在 DockerHub 的機器上幫你 build images，這樣就不用在自己機器上 build 好後再 push，在其他 server 上就可以直接 pull