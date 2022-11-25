---
title: "針對enum寫attribute並取值"
date: 2022-10-20T16:54:37+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "針對enum寫attribute並取值"
---

### 1.設定Attribute
```
public class StringValueAttribute : Attribute
{
	public string Value { get; private set; }

	public StringValueAttribute(string value)
	{
		this.Value = value;
	}
}
```

### 寫enum

```
public enum CategoryCode
{
	//餐飲
	[StringValue("A")]
	Food,

	//購物
	[StringValue("B")]
	Shopping,

	//美容
	[StringValue("C")]
	Cosmetic,

	//寵物
	[StringValue("D")]
	Pet,

	//生活服務
	[StringValue("E")]
	Life,

	//線上購物
	[StringValue("F")]
	EC,

	//生活繳費
	[StringValue("G")]
	Bill,

	//其他
	[StringValue("Z")]
	Other
}
```

### 3.寫擴充方法
```
public static class CategoryCodeExtension
{
	public static string GetStringValue(this CategoryCode value)
	{
		var attributes = (StringValueAttribute[])value
		   .GetType()
		   .GetField(value.ToString())
		   .GetCustomAttributes(typeof(StringValueAttribute), false);
		   
		return attributes.Length > 0 ? attributes[0].Value : string.Empty;
	}
}
```