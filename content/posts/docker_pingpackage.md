---
title: "docker 安裝ping套件&proxy"
date: 2022-10-24T15:36:51+08:00
draft: false
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---

# docker 安裝ping套件&proxy
- 安裝套件
apt-get update
apt-get install -y iputils-ping

- 設定proxy
export http_proxy=http://secproxy.evertrust.com.tw:6588 &&
export https_proxy=https://secproxy.evertrust.com.tw:6588