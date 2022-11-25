---
title: "big5難字處理"
date: 2022-10-18T11:14:55+08:00
draft: false
description: "C#的big5難字處理"
tags: ["big5","Csharp"]
type: post
showTableOfContents: true
slug: "big5難字處理"
---


### 判斷轉big5後是不是?
```C#
public bool IsBelongHardWord(char str)
        {
            var big5 = Encoding.GetEncoding("big5");
            
            var cInBig5 = big5.GetString(big5.GetBytes(new[] { str }));
            
            return str != '?' && cInBig5 == "?";
        }
```
### StringInfo的解法
- 上面解法不適用"𨫎"、"𩗴" 這種組合字，因為會被拆成2個char
```C#
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
	string s = "𨫎𩗴嫑堃瀞abc一二三";
	TextElementEnumerator charEnum = StringInfo.GetTextElementEnumerator(s);
	while (charEnum.MoveNext())
	{
		var str = charEnum.GetTextElement();
		var encode = Encoding.GetEncoding(950);
		var big5 = encode.GetString(encode.GetBytes(str));
		($"{str} 難字{str != big5}").Dump();
	}
```


### 參考
https://www.huanlintalk.com/2009/12/unicode-character-and-ranma-12.html