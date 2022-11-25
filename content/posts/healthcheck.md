---
title: "grpc使用http2_health_check問題"
date: 2022-10-20T16:58:58+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "grpc使用http2_health_check問題"
---

### grpc內appsetting設定
```
"Kestrel": {
        "EndPoints": {
            "Grpc": {
                "Url": "http://+:4270",
                "Protocols": "Http2"
            }
        }
    }
```

http/2支援可以用http或https
但是因為瀏覽器的http/2只支援https
所以health check只能用https才連的到
可以用curl指令去打grpc的endpoint
### curl

預設安裝git會安裝的版本，預設是不支援http/2的
路徑在C:\Program Files\Git\mingw64\bin\
![](https://imgur.com/pfdfulL.jpg)


`curl --http2 http://localhost:5000/health`
`curl --http2-prior-knowledge -X GET http://192.168.249.122:7202/backend/health`

要去這邊抓最新版本
https://curl.se/windows/
然後解壓縮檔案，在/bin裡面有curl.exe檔案
把這路徑加到環境變數，再移到system32上面
![](https://imgur.com/XiRaK2E.jpg)
新版的會支援http/2
https://stackoverflow.com/questions/51008686/whats-meaning-of-insecure-connection-in-grpc


### 信任憑證
`dotnet dev-certs https --trust`
https://docs.microsoft.com/zh-tw/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0&tabs=visual-studio#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos
https://medium.com/%E7%A8%8B%E5%BC%8F%E8%A3%A1%E6%9C%89%E8%9F%B2/windows-%E4%B8%BB%E6%A9%9F%E5%A6%82%E4%BD%95%E5%95%9F%E7%94%A8-tls-1-2-d72825f791b9