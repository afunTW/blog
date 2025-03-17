---
title: Paper Study - SPPNet
date: 2018-02-16 00:00:00
categories: AI
tags: [deep learning, computer vision, paper, sppnet]
---

> arxiv 論文: [https://arxiv.org/abs/1406.4729](https://arxiv.org/abs/1406.4729)

<!-- more -->

## 簡介

雖然 R-CNN 開啟了深度學習應用在 Object Detection 的時代, 但仍然存在許多缺點

- 浪費硬碟空間
- 部份訊息遺失
- 重複提取特徵

為了提升準確度與訓練速度, SPP-Net 主要是要解決部份訊息遺失以及重複提取特徵的問題

## 訓練

首先我們要先了解 R-CNN 的問題成因

### 部份訊息遺失

![Crop_Warp](https://i.imgur.com/cXf9biF.jpg)

因為 CNN 內的 fully-connected layer 必須跟每個 pixel 做連接, 需要指定 input/output 的 neuron 個數, 因此 R-CNN 的作法是在輸入 CNN 之前就先對每個 Region Proposal 做切割或是變形 (Crop/Warp) 來 resize 成一樣的大小, 但是這種作法會導致部份訊息的遺失

### 重複提取特徵

每個 Region Proposal 都會輸入 CNN 來提取特徵, 但是會有大量的 Region Proposal 是出自同一張圖片, 這其實是對同一張圖片上的某些部份重複提取特徵, 這也是 R-CNN 非常耗時的原因之一

### SPP-Net 解法

既然最終目的是取得每個 Region Proposal 各自的特徵, 而且這些特徵都出自同一張圖, 那麼我們是否將原始圖片輸入 CNN 取得整張圖片的特徵之後, 再透過「某種 Mapping」找到每個 Region Proposal 在整張圖片的特徵中相對應的位置, 最後再透過「某種 resize」把所有特徵變成同樣的大小, 最後再經過 fully-connected layer 就好？

![Mapping](https://i.imgur.com/PFOQjP2.jpg)

如上圖所示, 在 CNN 裏面的原圖經過多層 convolution 跟 pooling 之後產生的 feature map 相對位置還是不變的, 基於這個特性, 我們可以從原圖上 Region Proposal 的位置找到他映射在 feature map 上的位置, 而這個「某種 Mapping」就是根據 convolution layer 中 filter 的 stride 去計算相對位置, 細節請參考 Reference 或是論文

剩下的問題是: **要如何把這些在 feature map 上透過相對位置取得的 Region Proposal resize 成相同大小而又不遺失太多訊息？**

這就是 SPP-Net 的核心貢獻, Spatial Pyramid Pooling (SPP), 主要的想法是先決定 output 大小, 然後所有圖片再設計各自的 filter 參數 (window size, stride) 讓最後的每個 feature 大小都一樣, 另外這邊之所以叫作 Pyramid Pooling (金字塔 Poolㄛng) 是因為考慮到每個 Region Proposal 大小細節不一, 所以同一個 Region Proposal 需要經過多個不同 scale 的 filter 然後再做 concat

![SPP](https://i.imgur.com/qc9yKbA.jpg)

## 結果

以網路結構來說, SPP-Net 主要更改了 CNN 的架構, 前面透過 SS 演算法取得 Region Proposal 的方式以及最後經過 SVM 分類 + bounding box regression 的算法都跟 R-CNN 一樣, 論文中也實驗了多種 input size 或是使用不同的 CNN model 等等其他方式, 我這邊就不贅述

SPP-Net 發現透過整張圖片取特徵再抓相對位置比起 Region Proposal 個別經過 CNN 取得的特徵還要準, 這邊我會把他解釋成有些特徵是需要綜觀全圖才會得知, 所以不應該以 Region Proposal 當作 input image

以下附上 R-CNN 作者 (rbg 大神) 在 ICCV 2015 的投影片中比較 R-CNN 跟 SPP-Net (Kaiming He 大神) 的數據
![RCNN_SPP_compare](https://i.imgur.com/gdidoRR.jpg)

## Reference

- [SPP-Net 論文詳解](http://blog.csdn.net/v1_vivian/article/details/73275259)
- [Detection 系列文章 - 馮校](http://blog.csdn.net/xyfengbo/article/details/70227173)
- [Detection 系列文章 - venus](http://www.cnblogs.com/venus024/p/5717766.html)

---

## 補充

這邊同樣的很多數學跟演算法我比較沒有去深究, 原因是這些方法還是存在一些冗餘, 在接下來的論文中就會被取代了

但是這邊可以注意的是 R-CNN 作者跟 SPP-Net 作者都是出自 FAIR (Facebook AI Research)...

- rbg homepage: [http://www.rossgirshick.info/](http://www.rossgirshick.info/)
- Kaiming He Homepage: [http://kaiminghe.com/](http://kaiminghe.com/)
