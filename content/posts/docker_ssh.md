---
title: "SSH服務"
date: 2022-10-24T15:39:33+08:00
draft: true
tags: ["docker","ssh","linux"]
type: post
author: "Steve Lin"
description: ""
---

# SSH服務
> 遠端操作主機的方式，docker也可以用ssh連接和存取內部容器
- Linux下安裝OpenSSH
- 定義在應用層和傳輸層的安全協定
- 相較於FTP,Telnet更安全
### Linux安裝SSH
```
sudo apt-get purge openssh-server
sudo apt-get install openssh-server
sudo service sshd start
sudo service ssh status 
/lib/systemd/systemd-sysv-install enable ssh
預設會裝到/usr/sbin下
啟動方式是守護狀態 要用-D轉到前台運行
sudo /usr/sbin/sshd -D
```
啟動程式後會監聽port 22，就可以接收用戶端請求了
windows用戶端程式是PuTTY

### 指令
- 登入 
	- `ssh user@host` -p可以指定port
	- `ssh host` 不指定登入使用者
### 應用情境
把一些設定檔放在資料卷容器
然後起一個SSH的容器去操作這個資料卷容器
```
docker create --name config -v /etc/nginx alpine echo Nginx Configuration
docker create --name code -v /var/web alpine echo Root
docker create --name web --volumes-from code --volumes-from config -p 80:80 -p 443:443 nginx:1.10
docker pull rastasheep/ubuntu-sshd
docker run -d -P --name sshd --volumes-from code --volumes-from config rastasheep/ubuntu-sshd

//查詢配置的port 
docker port sshd 22
//然後就可以用本地端工具連了
ssh root@host -p 12345  //密碼是root
```
### 建構SSH服務映象
- commit的方式
```
進去安裝ssh軟體
docker run -it ubuntu:16.04 /bin/bash
#apt-get update && apt-get install -y openssh-server
//建立登入用的帳密
# mkdir /var/run/sshd
# groupadd wgroup
# useradd -g wgroup wuser
# passwd wuser
//root帳號的配置 預設是不能密碼登入
# more /etc/ssh/sshd_config | grep PermitRootLogin
//sed更改組態 變可以密碼登入
# sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
#exit
docker commit -m 'SSH Server' 3c10fae1bbcd ymdot/sshd
//提交成image
docker images
docker run -d -p 10022:22 ymdot/sshd /usr/sbin/sshd -D
啟動完就可以登進去看
```
測完這個container執行起來會直接退出，沒辦法運行
用dockerfile才可以

### 官方Dokerfile
```
FROM ubuntu:16.04
  
RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:screencast' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
  
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
  
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
  
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```
- 用`docker build -t ubuntu_sshd .` 建image
- 記得root:screencast 密碼要改掉
- `docker run -d -p 49154:22 -v /home/user/data:/mnt/data --name my_server ubuntu_sshd`
    - -d (detach) 讓container 生成後不要占用 terminal。
    - -p 指定主機上的 port 49154  接上 container 的 port 22，實際暴露的 port 除了不要覆蓋到原本主機的 port 22 外，可以自行選定，基於安全性原則一定要改。
    - -v 將主機上的資料夾 /home/user/data mount 到 container 的 /mnt/data 上。
    - –name 命名這個虛擬機的名稱 my_server `
- ssh root@[host] -p 49154(host主機IP)

https://www.itread01.com/p/125355.html