---
title: "docker的網路(II)"
date: 2022-10-24T15:40:50+08:00
draft: false
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---

# docker的網路(II)
- 基於Linux的network container裡面的Network Nampspaces
- 透過Veth pair(Virtual Ethernet Pair)虛擬網路通道出去
- docker利用bridge連接到宿主機的橋接器 就可以跟其他container通訊
### 主要組成
- Network Nampspaces:實現網路的隔離
- Veth Pair:打穿隔離環境的網路傳輸資料通道
- Linux Bridge:網路交換機，宿主機的橋接器
- Iptables:提供網路資料穿透與NAT等功能
### 容器網路模型CNM
實作Libnetwork，實作上面那些東西
- Sandbox:容器中隔離網路配置的虛擬環境，有獨立的網路組態等資訊，類似Network Nampspace
- Endpoint:傳遞網路資料的通道入口，類似Veth Pair
- Network:由一組端點組成，同一個網路的端點可以互相通訊
### 預設網路
- docker network ls可以顯示所有網路清單，首次啟用會有3個預設網路
- bridge是容器預設的網路，會對應到docker0
- 想改變容器網路可以在run的時候用`--network`修改
- host是使用宿主機的網卡
- `docker network inspect bridge`可以查看相關資訊
```
PS C:\Users\KuanFu> docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
0d96b30c3fca   bridge    bridge    local
8f72209b8925   host      host      local
76e669bab0e6   none      null      local
```
### 自訂網路跟相關指令
> 想要讓幾個特定容器互相溝通，而不是所有容器都用bridge互相存取
- 建立網路 `docker network create`
	- 預設會使用bridge
	- 可以用--subnet 建一個有子網範圍的容器網路 
```
//create一個網路 --driver可以簡化-d
docker network create --driver bridge isolated

docker network create --subnet 10.10.200.0/24 cnet
docker network inspect cnet
```
- 刪除網路`docker network rm [name]`

### 容器跟外部通訊
container連到doker0(虛擬網卡)會再轉發到其他網卡，要正常通訊需要IP forward正常運作，docker run可以用--ip-forward控制，預設是開啟
```
# sudo sysctl net.ipv4.conf.all.forwarding
net.ipv4.conf.all.forwarding = 1

0代表ip forward禁用
```
外部連接到容器靠port映射，是基於iptables防火牆(DNAT)
`sudo iptables -t nat -L -n`可以查看
-P可以找宿主機可以用的port，是一個範圍
```
sudo more /proc/sys/net/ipv4/ip_local_port_range
32768	60999

```
### 容器連接網路
- 用`--network`指定要連接的網路，可以指令預設的或自己建的網路
`docker run -it --network cnet bustbox`
- 容器連接到指定的網路可以用`docker network connect cnet bustbox` 前面是網路名稱，後面是container名稱
- 可以斷開網路`docker network disconnect cnet busybox`
### 自訂橋接器
```
先安裝套件
#sudo apt-get install bridge-utils
#sudo brctl addbr ymbr0
#sudo ip addr add 192.168.99.1/24 dev ymbr0
#sudo ip link set dev ymbr0 up
顯示橋接器相關資訊
#sudo ip addr show ymbr0
4: ymbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 5a:e7:bc:16:b6:75 brd ff:ff:ff:ff:ff:ff
    inet 192.168.99.1/24 scope global ymbr0
       valid_lft forever preferred_lft forever

還沒啟動docker時可以用docker daemon命令加-b 或--bridge參數告知docker採用指定的橋接器，
如果已經啟動的話要先停止再替換docker0
sudo dockerd --bridge ymbr0


```
