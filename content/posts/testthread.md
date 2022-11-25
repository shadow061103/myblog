---
title: "測試Task.WhenAll執行緒"
date: 2022-10-20T17:05:06+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "測試Task.WhenAll執行緒"
---
```C#
async Task Main()
{

	var tasks = new List<Task<string>>();
	foreach (var i in Enumerable.Range(1, 30))
	{
		var task = download("https://www.cmoney.tw/finance/technicalanalysis.aspx?s=2399",i);
		tasks.Add(task);
	}
	await Task.WhenAll(tasks);
}
async Task<string> download(string url,int id)
{
	($"{id}:1:{Thread.CurrentThread.ManagedThreadId}").Dump();
	var client=new WebClient();
	($"{id}:2:{Thread.CurrentThread.ManagedThreadId}").Dump();
	var task=client.DownloadStringTaskAsync(url);
	($"{id}:3:{Thread.CurrentThread.ManagedThreadId}").Dump();
	string content=await task;
	($"{id}:4:{Thread.CurrentThread.ManagedThreadId}").Dump();
	return content;
}
```