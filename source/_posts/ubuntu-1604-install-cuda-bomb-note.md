---
title: Ubuntu 16.04 安裝 CUDA + cuDNN + nvidia driver 的踩雷心得 (非安裝步驟詳解)
date: 2018-05-24 13:57:12
updated: 2020-12-09 13:57:12
categories: Data Engineering
tags: [ubuntu, nvidia, cuda, cudnn]
---
>   網路上有些人說 nvidia driver 安裝在 Ubuntu 上本來就很多雷，也有人說用 docker 裝的話一切都很方便，而我說你沒踩到雷只是因為你不是在擦屁股的那一個

<!-- more -->

# 在開始安裝之前

如果是 PC 裝 GPU 的話記得先注意一下你的顯卡有沒有因為太重，導致接口變形然後跟主版接觸不良，因為我嘗試很多種裝法都裝不起來之後發現只是接觸不良的問題…

我的需求是為了能夠跑 tensorflow-gpu，所以在測試方面會直接跑一個需要demo code ，這邊我直接選擇 keras project 裡的 mnist_cnn.py

[**keras-team/keras**](https://github.com/keras-team/keras/blob/master/examples/mnist_cnn.py)

你可以直接透過下面的方式下載並透過 nvidia-smi 監控

```shell
$ wget -c https://raw.githubusercontent.com/keras-team/keras/master/examples/mnist_cnn.py
$ python3 mnist_cnn.py # 執行 demo code
$ nvidia-smi -l # 不斷印出 GPU 使用狀況
```

這邊要成功執行必須有幾點要檢查

-   CUDA 版本是否支援 cuDNN 版本
-   cuDNN 是否支援 tensorflow 版本
-   nvidia driver 環境變數的路徑是否有設定正確
-   CUDA 路徑是否有設定正確

相關版本支援的檢查可以查看[**官網**](https://www.tensorflow.org/install/install_linux#nvidia_requirements_to_run_tensorflow_with_gpu_support)，以上幾點就是再踩雷的過程中特別需要檢查的地方，另外建議一下去 nvidia 官網辦個帳號，因為下載相關程式的時候會用到

[**Installing TensorFlow on Ubuntu | TensorFlow**](https://www.tensorflow.org/install/install_linux#nvidia_requirements_to_run_tensorflow_with_gpu_support)

CUDA 下載位置

[**CUDA Toolkit Archive**](https://developer.nvidia.com/cuda-toolkit-archive)

cuDNN 下載位置

[**cuDNN Archive**](https://developer.nvidia.com/rdp/cudnn-archive)

# 重要套件的路徑

這裡只是為了方便所以整理一下 (以 default 路徑為主)

-   `/usr/local/cuda` :
    不一定存在，通常會是某個指定 CUDA 版本的 symbolic link，可透過該路徑底下的 `version.txt` 檢查
-   `/usr/local/cuda-8.0` : 以 cuda 8.0 為例的實際路徑
-   `/usr/local/cuda-8.0/lib64` `/usr/local/cuda-8.0/include` : 
    cuDNN 相關位置，如果要手動更改版本可以下載完之後直接複製相關檔案到這兩個路徑底下，然後更改 symbolic link 指向的位置
-   `/usr/lib/nvidia-{version}` : 
    nvidia driver 安裝位置，若是裝 cuda 8.0 會一次下載 `nvidia-375` 與 `nvidia-384` ，使用時要注意環境變數的設定

上述東西都裝好後，仍需檢查設定的環境變數是否有錯，這些依照大家想在甚麼時候啟動這些環境變數決定

1.  `~/.bashrc` 等相關 shell 設定檔
2.  `~/.profile` 個人使用者環境變數設定檔
3.  `/etc/profile` 全體使用者環境變數設定檔

-   `CUDA_HOME=/usr/local/cuda-8.0` : 
    以 cuda 8.0 舉例，也可以設定成 `/usr/local/cuda`
-   `LD_LIBRARY_PATH=${CUDA_HOME}/lib64:/usr/local/lib:/usr/lib/nvidia-375` : 以 cuda 8.0 舉例， nvidia driver default 375 version
-   `PATH=${CUDA_HOME}/bin:${PATH}` : 
    cuda 中的工具也設定在環境變數中方便使用，e.g. `nvcc`

# 測試失敗重來就好

如果你測試失敗想重裝的話，記得要先把原先的全部都先砍掉

1.  清除 nvidia 相關套件

```shell
$ sudo apt-get purge nvidia*
$ sudo apt-get autoremove
```

\2. 透過 dpkg 檢查是否有非透過 apt-get 安裝的需要移除

```shell
$ sudo dpkg -l 'nvidia'
$ sudo dpkg --remove nvidia-{name}
```

---

# nvidia driver 相關的雷

## Ubuntu desktop 重複 login 畫面無法登入

首先當然現看你密碼有沒有打錯(廢話)，桌面版 login 發生問題我會直覺想到 X server

1.  `.Xauthority` 出現問題:
    首先透過 CTRL+ALT+F1 進入 tty 並登入，將紀錄了 X session 相關驗證資料的 `.Xauthority` 備份並砍掉舊的再重新登入
2.  nvidia driver 版本有問題:
    同上述切換到 tty1 後清掉所有 nvidia driver (purge) 再重新登入
3.  內外顯 driver 衝突 (Nouveau v.s. Nvidia):
    nouveau 是 open source driver，當我們裝好 nvidia driver 之後最好把 nouveau disable，可參考以下指令

```shell
$ echo -e "blacklist nouveau\nblacklist lbm-nouveau\noptions nouveau modeset=0\nalias nouveau off\nalias lbm-nouveau off\n" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
$ echo options nouveau modeset=0 | sudo tee -a 
$ /etc/modprobe.d/nouveau-kms.conf
sudo update-initramfs -u
```

## nvidia-smi command not found

同常在安裝 cuda 的過程中就會以 dependency 的方式被安裝成功了才對，如果失敗的話我比較建議乾脆重裝 cuda

1.  硬體故障 (這篇一開始就說了)
2.  nvidia driver 環境變數設定錯誤
3.  nvidia driver 本身發生錯誤

這邊可以直接進入 nvidia driver 位置檢查，e.g. `/usr/lib/nvidia-375/bin` 中應該會有 `nvidia-smi` 這隻程式，可以直接執行看看錯誤訊息是甚麼

```
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
```

如果看到上述錯誤訊息，建議可以往硬體狀況開始檢查，接著再檢查 disable Secure boot 是否可以解決

---

# runfile 派系安裝的雷

安裝步驟可以參考

[**Installing Nvidia CUDA 8.0 on Ubuntu 16.04 for Linux GPU Computing (New Troubleshooting Guide)**](https://www.linkedin.com/pulse/installing-nvidia-cuda-80-ubuntu-1604-linux-gpu-new-victor/)

[**How can I install CUDA on Ubuntu 16.04?**](https://askubuntu.com/questions/799184/how-can-i-install-cuda-on-ubuntu-16-04?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

## /var/share/dpkg/info 更動

執行 runfile 最後發現某些檔案遺失的時候，網路上有些文件會叫你先備份 `/var/share/dpkg/info` 然後刪掉原本的在重新跑 apt-get 安裝

要注意這會導致你之後在執行 apt-get 的時候會出現大量的警告訊息

```
dpkg: warning: files list file for package 'libssh2-1:amd64' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'libkrb5-3:amd64' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'libwrap0:amd64' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'libcap2:amd64' missing; assuming package has no files currently installed
...
```

雖然也有解法說出現這種警告訊息就執行類似下面這段 script 重新安裝 package (但明明套件都存在系統上只是 dpkg 找不到)

```shell
for package in $(sudo apt-get install catdoc 2>&1 | grep "warning: files list file for package '" | grep -Po "[^'\n ]+'" | grep -Po "[^']+");
do
  sudo apt-get -y install --reinstall "$package"
done
```

這邊需要花費非常大量的時間重跑，非常不建議更改 `/var/share/dpkg/info`

---

# deb 派系安裝的雷

安裝步驟可以參考

[**在 Ubuntu 14.04/16.04 安裝 NVIDIA 顯卡驅動程式、 CUDA Toolkit 及 cuDNN**](https://medium.com/@maniac.tw/在-ubuntu-14-04-16-04-安裝-nvidia-顯卡驅動程式-cuda-toolkit-及-cudnn-875e294530ed)

這邊強烈建議在安裝時要事先關掉 X server，通常是指關掉 `lightdm`

```shell
$ sudo service lightdm stop
```

## 安裝時發現 cuda 版本與自己抓的 deb (local) 版本不同

```bash
#!/bin/bash

# install CUDA Toolkit v8.0
# instructions from https://developer.nvidia.com/cuda-downloads (linux -> x86_64 -> Ubuntu -> 16.04 -> deb (network))
CUDA_REPO_PKG="cuda-repo-ubuntu1604_8.0.61-1_amd64.deb"
wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/${CUDA_REPO_PKG}
sudo dpkg -i ${CUDA_REPO_PKG}
sudo apt-get update
sudo apt-get -y install cuda

# install cuDNN v6.0
CUDNN_TAR_FILE="cudnn-8.0-linux-x64-v6.0.tgz"
wget http://developer.download.nvidia.com/compute/redist/cudnn/v6.0/${CUDNN_TAR_FILE}
tar -xzvf ${CUDNN_TAR_FILE}
sudo cp -P cuda/include/cudnn.h /usr/local/cuda-8.0/include
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda-8.0/lib64/
sudo chmod a+r /usr/local/cuda-8.0/lib64/libcudnn*

# set environment variables
export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64\${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

在測試別人寫好的 script 時有遇到，明明是下載 cuda 8.0 可是在 apt-get install cuda 的時候卻發現是安裝 cuda 9.2 (但網路文件都是寫這樣安裝)

這邊可能是因為你的 source list 上面已經有 cuda 9.2 的來源了，所以你必須要直接指定要裝 cuda 8.0 才會成功

```shell
$ sudo apt-get install coda-8.0
```

## libcublas.so.x.0 cannot open shared object file no such file or directory

cuda 或是 cuDNN 相關檔案找不到的錯誤訊息可以往兩個方向 debug

1.  環境變數設置確認是否正確
2.  cuDNN 版本是否正確

環境變數的問題請參考上面重要套件的路徑，cuDNN 版本參考上面下載位置到官網下載與 CUDA 相容的版本

---

如有錯誤，歡迎聯絡修正補充