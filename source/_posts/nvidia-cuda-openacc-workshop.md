---
title: NVIDIA GPU CUDA/OpenACC WorkShop 紀錄
date: 2019-04-10 00:00:00
updated: 2020-12-09 01:44:48
categories: GPU
tags: [cuda, openacc]
---

>   參加 NVIDIA 舉辦的 Workshop 紀錄

<!-- more -->

# Accelerate Science Applications

對於開發者來說使用 GPU 主要透過三種途徑

-   Libraries (e.g. cuDNN)
-   Compiler Directive (e.g. OpenACC)
-   Programming Language (e.g. CUDA)

這次主要介紹的是 [OpenACC](https://www.openacc.org/), 範例都是 C/ C++ / Fortran

## OpenACC Directive + PGI Compiler

![openacc-demo](openacc-demo.png)

如 OpenACC 官網所述的圖片，我們簡單的透過 acc 來告知編譯器哪些程式碼的部份可以做平行化，一般比較常用的還是以下幾種

-   `#pragma acc parallel loop`：對 loop 操作做平行化
-   `#pragma acc kernels`：讓機器自己決定哪裡可以做平行化
-   `#pragma acc data clauses`：資料搬移

對於 loop 平行化的處理有幾點特別提到的是

1.  支援 nested loop，但是建議需要平行化的部份都加上 acc directive
2.  kernels 雖然是自動判斷，但是有時候可能會沒辦法最佳平行，如果可以的話還是建議使用 parallel loop
3.  loop 裡面有些可能是需要 sequential 的操作，這邊也有提供 loop seq 的寫法來告知編譯器

當你做完這些平行化的操作之後，可以透過 [PGI](https://www.pgroup.com/) Compiler 來編譯成可平行化的程式，通常透過 -ta 這個參數來決定，PGI 除了可以編譯 GPU 平行以外也可以編譯 CPU 平行，詳細的細節要再看文件

>   OpenACC 跟 PGI 都不是 NVIDIA 自己的東西，而只是 NVIDIA 的合作夥伴，這邊有與會者提到 OpenCL，但 NVIDIA 表示因為那邊是一個聯盟，所以其實很難決定未來方向跟新增的 feature 該包含什麼，因此他們也還是推 OpenACC + PGI 的組合

![nvidia-visual-profiler](nvidia-visual-profiler.png)

如果你做完這些操作執行程式後發現 GPU 加速並沒有提昇太多，甚至還是輸 CPU 平行化加速的話，可以考慮使用 nvidia-visual-profiler 來做 profiling，有可能是時間都花在 data movement 了

因為 CPU 跟 GPU 記憶體是分開的，搬移時再透過 bus 傳資料 (通常是 PCI)，會受限於 bus 的頻寬 (現在的天花板大概是 1x Gbps)，這時候就可以用 OpenACC 來做 data movement 決定什麼時候該把 CPU memory 裡的資料搬到 GPU memory 了

-   `#pragma acc data copyin`：把資料從 CPU 搬到 GPU memory
-   `#pragma acc data copyout`：把資料從 GPU 搬到 CPU memory

其他還有建立暫存的方式等方式，這邊就可以讓開發者自己決定什麼資料在什麼時候搬到 GPU 會是最佳的解法，尤其在 loop 中特別容易會發生這樣的狀況

如果資料量太大，這邊也可以決定只搬移部份資料進 GPU 運算就好，如同陣列操作一樣給定 index 或是 size 大小就可以了

## Unified memory

>   俗話說：懶惰是工程師的美德

後來工程師們覺的連寫 pragma 都有點懶，所以 NVIDIA 後來又推出 Unified memory，抽象概念是將 CPU 跟 GPU 的記憶體整併成一個 virtual memory，但實際上就是讓機器自己去決定什麼時候該做 data movement

**這邊利用到的是記憶體存取時的一個狀況叫做「Page fault」**

系統中管理記憶體的單元叫做 MMU，當我們跟系統要記憶體空間時，其實是以 Page 為單位，如果在執行程式的過程中發現資料不再記憶體內就會發生 Page fault
Unified memory 就是當發生 Page fault 的時候就會去做 data movement，看當時是在哪裡發生的決定要做 copyin 還是 copyout，雖然這樣是比較方便但是手動決定哪裡該做 data movement 還是可以達到較好的優化

>   這邊有與會者提到是否會發生 OOM (out-of-memory) 的情形，NVIDIA 方面表示這就要看顯卡版本，如果是比較舊的就會，新的話會有額外方式盡量避免這種狀況
>   
>   假如當 GPU 剩餘記憶體空間不足，它會先搬一些資料回去 CPU 記憶體，再把當下要處理的資料搬進 GPU 內運算

另外還有提到 GPU Direct，不過那是 GPU 之間的資料搬移，不是這邊所討論的問題

## CUDA Programming Language

上面提到的 GPU 加速跟平行化其實就算不是現在 AI 套餐程式碼也會用到，但隨著 ML / DL 近幾年的關注，Python 在 GPU 上的 support 也是越來越多,這邊主要就是介紹一下 Python 上幾個跟 CUDA 有關的套件

-   [CuPy](https://cupy.chainer.org/)：numpy + scipy on GPU
-   [Numba](http://numba.pydata.org/)：JIT compiler
-   [PyCUDA](https://documen.tician.de/pycuda/)：在 Python 中跑 CUDA coda
-   [RAPDIS](https://rapids.ai/index.html)：scikit-learn on GPU

以使用上的簡易程度來說是：CuPy/RAPDIS > Numba > PyCUDA
以套件靈活程度來說則是：CuPy/RAPDIS < Numba < PyCUDA

### CuPy

![Cupy demo code](cupy-demo-code.png)

這邊基本上只需要改 import 的套件就可以直接使用了

點進去官網的時候發現除了 NVIDIA 有 support 這個專案以外，居然連日本很強的 Preferred Network 都有在支援

### Numba

![numba demo code](numba-demo-code.png)

這邊可以發現簡單的應用只要透過 Python decorator 宣告就可以，不過要注意的是 numba 同時支援 CPU JIT (just-in-time)，如果使用的是 `@numba.jit` 那就會是 CPU 版本的

### PyCUDA

![PyCUDA demo code](PyCUDA-demo-code.png)

這邊是可以讓你在 Python code 中內嵌 C code 來跑 CUDA 相關的程式

### RAPIDS

![RAPIDS Open GPU Data Science](rapids-open-gpu-data-science.png)

RAPIDS 主要就是 Nvidia 推的一個集合，針對圖片或是訊號等等不同處理目的都有前綴為「cu」的一個套件，如果要看更詳細的介紹可以到官網上面看

## Reference

-   OpenACC https://www.openacc.org/
-   PGI Compiler https://www.pgroup.com/
-   nvidia-visual-profiler https://developer.nvidia.com/nvidia-visual-profiler
-   CuPy https://cupy.chainer.org/
-   Numba http://numba.pydata.org/
-   PyCUDA https://documen.tician.de/pycuda/
-   RAPIDS https://rapids.ai/
-   知乎 numba 基本應用 https://zhuanlan.zhihu.com/p/27152060
-   統一計算架構CUDA的概念及模型 https://www.ithome.com.tw/node/77749

## Comments

目前專案流程主要分為 data engineering, training/evaluation, data postprocessing, 而大家只會關心中間訓練模型時 GPU 的使用狀況

看得出來 NVIDIA 其實連 data 的前後處理也有 pipeline 可以處理，因為這邊經常才是最耗時的地方，短短半天的時間其實吸收蠻多東西的，也還有很多部份沒有補充上來

尤其在記憶體管理那邊我的印象都快模糊了，知道 CPU 記憶體是階層式的，但不太清楚什麼情況或是怎麼操作才能把該加速的資料放到 cache 或是 GPU memory，GPU 上的記憶體架構與管理也沒學過

如果有哪裡資訊不完整或是講錯的再麻煩糾正，感謝