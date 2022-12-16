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
    //直接用ReadLine讀也可以
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
### 串流轉成string
```
 public static string ToStringContent(this Stream stream)
        {
            var reader = new StreamReader(stream, Encoding.GetEncoding(950), true);
            var content = reader.ReadToEnd();
            return content;
        }
```


### 壓縮跟解壓縮檔案
- 使用套件 `DotNetZip`

擴充方法
```
public static class ZipHelper
    {
        public static Stream Zip(IEnumerable<ZipFileModel> zipFileModels, string password = "")
        {
            var stream = new MemoryStream();
            using var zipFile = new ZipFile(Encoding.GetEncoding(950));

            //檔案是0kb或空的時密碼無作用
            if (!string.IsNullOrEmpty(password))
                zipFile.Password = password;

            foreach (var model in zipFileModels)
                zipFile.AddEntry(model.FileName, model.FileStream.ToMemoryStreamBytes());

            zipFile.Save(stream);

            return stream;
        }

        public static MemoryStream UnZip(this Stream stream, string fileName, string password = "")
        {
            var ms = new MemoryStream();
            using (var zipFile = ZipFile.Read(stream))
            {
                zipFile.Password = password;
                zipFile[fileName].Extract(ms);
                ms.Position = 0;
            }
            return ms;
        }
    }


```
- 使用.net core的ZipFile.OpenRead
Namespace:System.IO.Compression

```
 using (ZipArchive archive = ZipFile.OpenRead(zipPath))
        {
            foreach (ZipArchiveEntry entry in archive.Entries)
            {
                if (entry.FullName.EndsWith(".txt", StringComparison.OrdinalIgnoreCase))
                {
                    // Gets the full path to ensure that relative segments are removed.
                    string destinationPath = Path.GetFullPath(Path.Combine(extractPath, entry.FullName));

                    // Ordinal match is safest, case-sensitive volumes can be mounted within volumes that
                    // are case-insensitive.
                    if (destinationPath.StartsWith(extractPath, StringComparison.Ordinal))
                        entry.ExtractToFile(destinationPath);
                }
            }
        }
```
https://learn.microsoft.com/en-us/dotnet/api/system.io.compression.zipfile.openread?view=net-7.0

- [ExtractToDirectory](https://learn.microsoft.com/zh-tw/dotnet/api/system.io.compression.zipfile.extracttodirectory?view=net-7.0)