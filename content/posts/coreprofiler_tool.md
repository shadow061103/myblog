---
title: "CoreProfiler_效能監測工具"
date: 2022-10-20T16:46:25+08:00
draft: false
tags: ["Csharp","coreprofiler"]
type: post
author: "Steve Lin"
description: ""
slug: "CoreProfiler 效能監測工具"
---
# CoreProfiler 效能監測工具
### 安裝nuget套件
- CoreProfiler.Web
- CoreProfiler
### 註冊StartUp
在 Configure加上`app.UseCoreProfiler(true);`
### 加上coreprofiler.json檔案
檔案屬性要改成內容-有更新才複製
```json
{
  "circularBufferSize": 200,
  "filters": [
    {
      "key": "/coreprofiler",
      "value": "/coreprofiler",
      "type": "CoreProfiler.ProfilingFilters.NameContainsProfilingFilter, CoreProfiler"
    },
    {
      "key": "static files",
      "value": "ico,jpg,js,css,svg,json,ttf,woff,woff2,eot",
      "type": "CoreProfiler.ProfilingFilters.FileExtensionProfilingFilter, CoreProfiler"
    }
  ]
}
```

### 如何使用
- 在要監看的程式包起來
```C#
var stepName = $"{nameof(這邊是監看的類別)}.{nameof(這邊是監看的方法)}";
using (ProfilingSession.Current.Step(stepName))
{ 
            
}
```
- 使用attribute，需要安裝套件AspectInjector.Broker
```C#
[Injection(typeof(CoreProfilerAspect))]
    public class CoreProfilingAttribute : System.Attribute
    {
        /// <summary>
        /// ProfilingName.
        /// </summary>
        /// <value>The name of the profiling.</value>
        public string ProfilingName { get; set; }

        /// <summary>
        /// ProfilingStep.
        /// </summary>
        /// <value>The profiling step.</value>
        public IDisposable ProfilingStep { get; set; }

        public CoreProfilingAttribute()
        {
        }

        public CoreProfilingAttribute(string profilingName)
        {
            this.ProfilingName = profilingName;
        }
    }
```
```C#
/// <summary>
    /// CoreProfilerAspect
    /// </summary>
    [Aspect(Scope.Global)]
    public class CoreProfilerAspect
    {
        /// <summary>
        /// Invokes the specified name.
        /// </summary>
        /// <param name="name">The name.</param>
        /// <param name="arguments">The arguments.</param>
        /// <param name="method">The method.</param>
        /// <param name="type">The type.</param>
        /// <param name="triggers">The triggers.</param>
        /// <returns></returns>
        [Advice(Kind.Around, Targets = Target.Method)]
        public object Invoke(
            [Argument(Source.Name)] string name,
            [Argument(Source.Arguments)] object[] arguments,
            [Argument(Source.Target)] Func<object[], object> method,
            [Argument(Source.Type)] Type type,
            [Argument(Source.Triggers)] System.Attribute[] triggers
        )
        {
            var attribute = triggers.OfType<CoreProfilingAttribute>().FirstOrDefault();
            string stepName = attribute is null
                ? $"{type.Name}.{name}"
                : string.IsNullOrWhiteSpace(attribute.ProfilingName)
                    ? $"{type.Name}.{name}"
                    : attribute.ProfilingName;
            using (ProfilingSession.Current.Step(stepName))
            {
                return method(arguments);
            }
        }
    }

```
- DB connection的用法(連線改成自己的)
```C#
return new ProfiledDbConnection(new SqliteConnection("Data Source=./mvcefsample.sqlite"), () =>
            {
                if (ProfilingSession.Current == null)
                    return null;

                return new DbProfiler(ProfilingSession.Current.Profiler);
            });
```
### 執行應用程式
連到/coreprofiler/view