---
title: "用CodeMaid進行程式碼分析"
date: 2022-10-24T15:51:51+08:00
draft: false
tags: ["單元測試"]
type: post
author: "Steve Lin"
description: ""
---

> 分析>計算方案的程式法度量

## 五種指標
- 可維護性指標 (Maintainbility Index)
- 循環複雜度 (Cyclomatic Complexity)
- 繼承深度 (Depth of Inheritance)
- 類別結合程度 (Class Coupling)
- 程式碼行數 (Lines of Code)
### 可維護性指標
- 算出介於 0 到 100 之間的指數值，代表維護程式碼的相對難易程度。 值愈高表示可維護性愈佳
- 綠色等級介於 20 和 100 之間，表示程式碼的可維護性良好。
- 黃色等級介於 10 和 19 之間，表示程式碼的可維護性適中。
- 紅色等級是介於 0 和 9 之間的等級，表示可維護性低。
### 循環複雜度
- 測量程式碼在結構上的複雜程度。建立此複雜度的方式是計算程式流程中不同程式碼路徑的數目
- 要越低越好，方法不超過10或15
### 繼承深度
- 指出延伸到類別 (Class) 階層的根 (Root) 的類別定義數目。 階層愈深，可能愈難找出定義與/或重新定義特定方法和欄位的位置
- 越低越好
### 類別結合程度
- 透過參數、區域變數、傳回型別、方法呼叫、泛型或樣板具現化、基底型別、介面實作、外部型別上定義的欄位以及屬性修飾等，測量特殊類別的結合程度
- 良好的軟體設計應指定聚結性 (Cohesion) 高但結合程度 (Coupling) 低的型別和方法。
### 程式碼行數
- 指出程式碼中行數的約略值。 這個數目是以 IL 程式碼為依據，因此不是原始程式碼檔案中精確的行數
- 如果數目非常大，表示型別或方法嘗試執行的工作可能過多，而應該分割工作，也表示該型別或方法可能難以維護