---
title: "Messagepack"
date: 2022-10-20T17:01:33+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "MessagePack"
---
### Nuget
```
Install-Package MessagePack
//分析器
Install-Package MessagePackAnalyzer
//擴展
Install-Package MessagePack.ImmutableCollection
Install-Package MessagePack.ReactiveProperty
Install-Package MessagePack.UnityShims
Install-Package MessagePack.AspNetCoreMvcFormatter
```
### Class範例
- class上要加上`[MessagePackObject]`
- 屬性加上`[Key(0)]`或`[Key"name"]
- `[IgnoreMember]`可以不序列化
- 作者建議Key使用int，因為會比較快，但是轉出的json不會有key name
- key建議不要跳數字，否則會出現`[null,null,null,0,null,null,null,null,null,null,0]`

### 序列化對象
- 可以是public class或struct
- 必須標記`[MessagePackObject]`跟`[key]`
- `[MessagePackObject]`可以用`[DataContract]`取代，`[key(0)]`可以用`[DataMember(Order = 0)]`或`[DataMember(Name = string)]`取代
```C#
 [MessagePackObject]
    public class PackClass
    {
        // Key 是序列化索引，对于版本控制非常重要。
        [Key(0)]
        public int Age { get; set; }

        [Key(1)]
        public string FirstName { get; set; }

        [Key(2)]
        public string LastName { get; set; }

        // 公共成员中不序列化目标，标记IgnoreMemberAttribute
        [IgnoreMember]
        public string FullName { get { return FirstName + LastName; } }
    }
//結果 [99,"hoge","huga"]
    
 [MessagePackObject]
    public class Sample2
    {
        [Key("foo")]
        public int Foo { get; set; }

        [Key("bar")]
        public int Bar { get; set; }
    }    
// 結果 {"foo":123,"bar":456}
[DataContract]
    public class PackClass
    {
        [DataMember(Order = 0)]
        public int Age { get; set; }

        [DataMember(Order = 1)]
        public string FirstName { get; set; }

        [DataMember(Order = 2)]
        public string LastName { get; set; }

        [IgnoreDataMember]
        public string FullName { get { return FirstName + LastName; } }
    }
```
### 使用
```C#
// 序列化
var bytes = MessagePackSerializer.Serialize(obj);
//反序列化
var mc2 = MessagePackSerializer.Deserialize<PackClass>(bytes);
//轉json
var json = MessagePackSerializer.ConvertToJson(bytes);

//不想寫attribute的話也可以直接轉
public class ContractlessSample
    {
        public int MyProperty1 { get; set; }
        public int MyProperty2 { get; set; }
    }
var data = new ContractlessSample { MyProperty1 = 99, MyProperty2 = 9999 };

var bin = MessagePackSerializer.Serialize(
                data,
                MessagePack.Resolvers.ContractlessStandardResolver.Options);
```

#### 資料來源
https://github.com/neuecc/MessagePack-CSharp
https://www.cnblogs.com/stulzq/p/8039933.html
