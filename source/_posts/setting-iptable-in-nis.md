---
title: Setting iptable in NIS
date: 2016-12-26 00:00:00
categories: DevOps
tags: [ubuntu, nis, iptable]
---

> 紀錄上古時代的 NIS 使用紀錄

<!-- more -->

# NIS ?

不論是實體機器或是 VPS (Virtual Private Server)，當手上需要管理的 server 愈來愈多的時候，我們都不希望逐一在 server 上面做設定，於是就有了中央管理的服務出現，**NIS (Network Information Service)** 就是其中一種。

## NIS 的安全性

NIS 比起其他中央管理帳號的服務，安裝與設定是相對簡單，但是安全性來說是相當低的。

>  <p><strong style="color:red;">Anyone who can get access to the daemon can dump your password lists</strong>. If they can do that, then they have your passwords. It doesn't matter that the passwords are encrypted; they are plaintext equivalent (since authentication is done with encrypted passwords, you don't need to know the text password, you just need to write an app to provide the encrypted one to the authentication system correctly).<p><small> [Ubuntu wiki](https://help.ubuntu.com/community/SettingUpNISHowTo) </small>

NIS 自己提供的防護這邊暫且不提，有興趣的可以去看看 [Ubuntu wiki](https://help.ubuntu.com/community/SettingUpNISHowTo)，或是透過鳥哥先學概念，但是要注意如果你是用 Ubuntu 的話會發現鳥哥上面的指令或是路徑你會找不到（廢話，人家是用 CentOS）。像是 NIS 要設定 nisdomainname 時，Ubuntu 上並沒有 `/etc/sysconfig/network`這份檔案，就算查到在 Ubuntu 上面應該是要去找 `/etc/network/interfaces`，你也看不到上面有 nisdomainname 設定參數（補充：其實直接透過 `nisdomainname [domain name]` 這個指令就可以設定）。

## 設定 iptables

### 建議

- **NIS slave server**：在設定 iptables 之前有些要建議的事項，如果你管理的 server 有其他人正在使用（e.g. 公司同事），請先確保你有 NIS slave 當備用，不然 iptables 一設定錯誤。你自己被擋掉到還好，全公司的人都被擋掉的話<del>（大家就可以提早下班了）</del>，想也知道這是多麼大的麻煩。

- **local sudo 帳號 or 直接存取**：同樣的，如果對語法不熟常常會自己把自己鎖在門外，人能夠在機器前面最好，local 帳號只能應對 NIS 服務失效，但如果你把所有 IP 都擋掉，還是只能透過 console 直接存取。

### 設定 NIS master

設定 `/etc/hosts.allow` 跟 `/etc/hosts.deny` 以外，比較常用的是設定 iptables，我個人還會裝 `iptables-persistent`，參考一些網路上的設定方式比較喜歡寫 shell script 然後把 service 跟針對 ip 的規則這些分開，但其實這些都只是個人喜好，最後 iptables rule 設定正確就可以了。

#### Step 1: 安裝

```bash
$ sudo apt-get update
$ sudo apt-get install iptables iptables-persistent
```

#### Step 2: 固定 NIS 需要用到的 port

我個人比較喜歡 INPUT filter policy 設定為 DROP，然後再針對 IP 或是 port 去設定條件，這樣的設定算是比較嚴格，而且設定的過程中，也可以複習一下自己管理的網路架構圖跟開啟的服務有哪些

**RPC (Remote Procedure Call) port**

NIS 是透過 **RPC** 來做遠端查詢，你可以透過指令 `rpcinfo -p` 查看詳細資訊。首先要先確定我們有開啟 RPC 服務，看到 `portmapper` 代表 RPC 有開啟，你可以發現使用的是 port 111，如果你沒看到的話請先執行 `sudo service portmap start` 開啟服務

幸運的是，RPC 是固定使用 port 111，所以我們後面可以直接針對 port 111 設定防火牆規則

**NIS port**

如果你一開始在設定 NIS 時並沒有考慮到 iptables，沒有固定 NIS 服務的 port，那每次你做 `sudo service ypserv restart` 或是 `sudo service ypbind restart` 時，你會發現他的 port 會一直變，所以下一步我們要固定這些服務的 port

透過你常用的編輯器打開 `/etc/default/nis`，我們需要固定的 port 有四個，在這邊我們假設我們要設定 port 875~878，這邊要注意一下 `YPPASSWDDARGS` 的設定參數不太一樣

1. YPSERVARGS = "-p 875"
2. YPBINDARGS = "-p 876"
3. YPPASSWDDARGS = "--port 877"
4. YPXFRDARGS = "-p 878"

#### Step 3: 針對 NIS 需要用到的服務新增 rule

這邊你可以再次透過 `rpcinfo -p` 觀察發現，哪些需要用 tcp，哪些需要用 udp，然後再根據這個去設定 iptables rule，為了稍微增加一點點安全性，可以不要用 ALL 就不要用 ALL

```bash
$ iptables -A INPUT -p tcp -i eth0 -s [要開放的 server IP] -m multiport --dport 111,875,876,878 -j ACCEPT

$ iptables -A INPUT -p udp -i eth0 -s [要開放的 server IP] -m multiport --dport 111,875,876,877,878 -j ACCEPT
```

#### Step 4: 重啟服務

這邊要注意的是，如果你直接重啟 ypserv 和 ypbind 會出現：`YPBINDPROC_DOMAIN: Domain not bound`，因為你沒有通知 RPC server 你做了變更，這邊要先重啟 portmap，再重啟 ypserv 和 ypbind 才會成功

```bash
$ sudo service portmap restart
$ sudo service ypserv restart
$ sudo service ypbind restart
```

#### Step 5: 測試是否 NIS 有正常服務

首先，因為我們是要測試 NIS 在防火牆環境中是否能夠執行，請先在 NIS client 設定 iptables，記住不要針對 NIS master IP 做設定，這樣就算可以連線但意義不同

然後我們稍早有提過為了不要讓其他人的服務中斷，我們要確保 NIS slave 的存在，現在要測試的時候我們就先在一台沒人用的 NIS client 上面編輯 `/etc/yp.conf` 這份檔案，確認我們設定的 NIS server 只有我們剛剛設定過的 NIS master

接著重啟 `ypbind` 之後，我們可以透過 NIS 提供的測試指令：`yptest` 來檢查

---

### 後記

上面說的注意事項其實我都踩過了，連最嚴重的「全公司的人無法連線」我都遇到了，當時是大概是晚上六七點，已經超過可以進入機房的時間，所以我也沒辦法進去修，只能一直跟大家道歉，然後在一些沒有被影響到的 server 上幫他們佈署環境，隔天早上9點多再馬上進機房解決。

指令下錯的瞬間真的臉都白了⋯⋯

雖然這篇內容的東西也不算複雜，但是真的是找了很多論壇討論串才解決，因為真的幾乎沒人在用 NIS，如果有中文的教學也幾乎都跟鳥哥內容差不多（也就代表著都是 CentOS 環境），然後關於 iptables 的設定，如果確定 rule 都沒有問題，建議可以設定成重新開機時也預先啟動 iptables，我個人是透過指令 `iptables-persistent save` 儲存 rule。

### Reference

* [鳥哥的 Linux 私房菜](http://linux.vbird.org/linux_server/0430nis.php)
* [Ubuntu wiki](https://help.ubuntu.com/community/SettingUpNISHowTo)
* [Linux 設定 iptables](http://blog.pulipuli.info/2011/07/linuxiptables.html)
 