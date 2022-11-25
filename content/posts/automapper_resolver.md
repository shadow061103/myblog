---
title: "AutomapperResolver無法用建構式注入解法"
date: 2022-10-20T17:06:27+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "AutomapperResolver無法用建構式注入解法"
---

### 原本寫法
```C#
 var mapperConfiguration = new MapperConfiguration(mapperConfig =>
            {
                mapperConfig.AddProfile<ControllerProfile>();
                mapperConfig.AddProfile<ServiceProfile>();
            });
            services.AddSingleton(mapperConfiguration.CreateMapper());
```
更新後寫法
```C#
 services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());
```

### Resolver範例
```C#
public class CaseFromResolver : IValueResolver<CaseFromDataModel, CaseFromDto, string>
    {
        public IWebRentCaseSiteService _webRentCaseSiteService;

        public CaseFromResolver(IWebRentCaseSiteService webRentCaseSiteService)
        {
            _webRentCaseSiteService = webRentCaseSiteService;
        }

        public string Resolve(CaseFromDataModel source, CaseFromDto destination, string destMember, ResolutionContext context)
        {
            var dict = _webRentCaseSiteService.GetRentCaseSourceDictionary()
                .ConfigureAwait(false)
                .GetAwaiter()
                .GetResult();

            return dict.ContainsKey(source.CaseFrom) ? dict[source.CaseFrom] : "";
        }
    }
```