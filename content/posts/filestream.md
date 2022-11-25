---
title: "檔案串流彙整"
date: 2022-10-20T16:49:58+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "檔案串流彙整"
---

連結
https://dotblogs.com.tw/kinanson/2017/06/03/142558

### 開啟檔案轉成Stream
```C#
private MemoryStream GetFileStream(string filePath, string fileName)
        {
            var path = Path.Combine(filePath, fileName);
            using var fileStream = new FileStream(path, FileMode.Open);

            var memoryStream = new MemoryStream();

            fileStream.CopyTo(memoryStream);

            return memoryStream;
        }
```
### 讀取本地端檔案
```C#
var result=new List<string>();
	var path = @"D:\merchantcode.txt";
	using var reader = new StreamReader(path, Encoding.GetEncoding(950));
	var content = await reader.ReadToEndAsync();
	using var reader2 = new StringReader(content);
	string line;
	while ((line = reader2.ReadLine()) != null)
	{
		result.Add(line);
	}
	
	//result.Dump();
	return result;
```