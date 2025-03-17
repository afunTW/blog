---
title: Paper Study - RCNN
date: 2018-02-14 00:00:00
categories: AI
tags: [deep learning, computer vision, paper, rcnn]
---

> arxiv 論文: [https://arxiv.org/abs/1311.2524](https://arxiv.org/abs/1311.2524)

<!-- more -->

## 簡介

R-CNN 算是證明深度學習應用在 Object Detection 問題的分水嶺

在此之前都是透過人工處理特徵, mAP (mean Average Precision) 提升非常緩慢, 但是後來 CNN 以及透過大樣本訓練出來的模型的出現, 我們就可以用深度學習方式取出圖片特徵, 並且有還不錯的進步

![Computer Vision Tasks](https://i.imgur.com/SCzd0oL.png)

Computer Vision 在深度學習上的應用主要是上面範例圖中的四大題目, 其中 R-CNN 要解決的是 Object Detection, 其主要貢獻為

- 透過 CNN 提取特徵
- 用大樣本 pretrain model 之後再透過 小樣本做 fine-tune 解決 overfitting 的問題

## 訓練

![RCNN](https://i.imgur.com/Pp1J62N.jpg)

整個過程大概可分為幾個部份, 以下會再根據每個階段做簡單介紹

1. Region Proposal
2. CNN extract feature
3. SVM classfication
4. bounging box regression

### Region Proposal

Object Detection 一開始會先透過 sliding window 的方式提取出多個建議框, 然後再去判讀建議框內是否有我們要判別的物體, 而這個建議框就是這邊提到的 Region Proposal, 在往後的文章及論文中提到的 ROI (Region of Interest) 也是指一樣的東西

傳統的作法是透過 sliding window 搭配不同的 window size 暴力提取 Region Proposal, 而 R-CNN 是透過 Selective Search 演算法提取約 2000 個 Region Proposal, 大概的作法是

1. 透過 Efficient Graph-Based Image Segmentation 取得原始分割區域
2. 計算區域相似度, 將相似度高的區域合併
    - 顏色空間多樣化
    - 相似多樣化
        - 顏色相似度: 對每個 color channel 取 25 bins 做 color histogram
        - 紋理相似度: SIFT-like feature
        - 大小相似度: 讓小區域先合併
        - 吻合相似度: 合併後 ROI 差異
3. 提取每個區域的 ROI

> Selective Search 細節可參考: [http://blog.csdn.net/mao_kun/article/details/50576003](http://blog.csdn.net/mao_kun/article/details/50576003)

### CNN extract feature

#### Preprocess

- Resize Region Proposal
- Normalization

根據 SS 提取出來的 Region Proposal 大小不一, 因此在輸入 CNN 之前會先對 Region Proposal 做切割或是變形 (Crop/Warp) resize 成相同大小, 這篇論文是 resize 成 (227, 227), 之後再個別對每個 Region Proposal 減掉個別的 mean 做 normalization

#### CNN

經過 resize 跟 normalization 後的 Region Proposal 再當作 input 輸入 CNN, 最後 output 4096 維的 feature. 因為我們有 2000 個 Region Proposal, 所以這邊的 output 就會是 (2000, 4096)

### SVM classification

SVM 是用來做 binary classification 的分類器, 假設我們有 20 種物件要做辨別, 那就需要 20 個 SVM 分類器, 權重矩陣就是 (4096, 20)

因此 Region Proposal 經過 CNN 再經過 SVM 最後 ouput 會是 **(2000, 4096) X (4096, 20) = (2000, 20)**, 最後再對剩下的分類結果做 NMS(non maximum suppression)[^NMS], 剔除重疊的 Region Proposal, 得到得分比較高的 Region Proposal

[^NMS]: NMS ([detail](http://www.cnblogs.com/makefile/p/nms.html)) 可以視為局部最大搜索

### Bounding Box Regression

對所有分類剩餘的 Region Proposal 做 Regression 修正位置成為最終要輸出的 bounding box

我們在計算最後的 mAP 時是根據 IOU (Intersection over Union), 也就是**兩個 Region 的交集面積 / 兩個 Region 的聯集面積**, 以面積覆蓋率來計算定位精準度, 因此我們透過 SS 選出來的 Region Proposal 即使經過 SVM 分類成功, 還是有可能因為 IOU 的關係，算出來的 mAP 不夠高

![Explain Regression](https://i.imgur.com/UFWnsyK.jpg)

以上圖舉例, 黃色框是 SS 取出的多個 Region Proposal, 紅色框是我們事先標記好的實際位置, 我們稱之為 Ground Truth

1. 經過 SS 取出的的結果
2. 透過 NMS[^NMS] 把重複的 Region Proposal 剔除
3. 我們會發現實際上定位的紅色框與黃色框還是有一定差距, 這邊再透過 Regression 對黃色框做修正

## 結果

雖然在論文中有做了其他實驗, 例如更換 CNN 架構, 測試不同資料集, 但是這邊介紹 R-CNN 主要是因為他是 Deep Learning 在 Object Deteion 上應用的里程碑, 實際上 mAP 還是達不到可以做產品的標準, 而且訓練的時間也非常長, 需要的硬碟空間也非常大, 這些缺點都會在之後的系列論文中一一改善

## Reference

- [RCNN blog](http://blog.csdn.net/wopawn/article/details/52133338)
- [Detection 系列文章 - 馮校](http://blog.csdn.net/xyfengbo/article/details/70227173)
- [Detection 系列文章 - venus](http://www.cnblogs.com/venus024/p/5717766.html)

---

## 補充

因為 Deep Learning 在圖片上的結果是相對其他領域顯著的, 所以我接下來會紀錄一些這邊的發展過程, 但是一些比較久遠的論文（像這篇就是）我會先跳過技術細節, 因為這些之後也會被取代掉, 而且因為實作成本太大, 所以我也不會實際去測試過, 如果有錯誤再麻煩告知, 也歡迎討論
