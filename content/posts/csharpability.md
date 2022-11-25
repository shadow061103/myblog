---
title: "Csharp本事筆記"
date: 2022-10-20T17:04:27+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "Csharp本事筆記"
---

### 共變性逆變性
- 無法在編譯時期抓出型別不相容的錯誤
- 泛型可以把小的型別當作大的來操作，但因為宣告是`<out T>`，所以只能用在輸出的場合
- 任何參考型別（reference type）的泛型串列，你都可以將它當作`IEnumerable<object>`的物件來操作
- 共變性:把較小型別指派給較大型別
- 逆變性:把較大型別指派給較小型別

```C#
List<string> stringList = new List<string>();
List<object> objectList = stringList; // 即使C# 4 也不能這樣寫!
但是可以
List<string> stringList = new List<string>();
IEnumerable<object> objects = stringList;
```

##### `IEnumerable<T>`都是唯讀的，可以當作參數使用


### nullable
- 只有實質型別可以用Nullable< T>
```C#
int? x = null;
int? y= x ?? 5;
//等同於
int? y= =(x != null)? x : 5;
```

### 委派
> 可以在方法裡外包一個函式，方法在執行的時候可以回頭呼叫那個涵式
- C#內建委派
    -  Action<in T1, in T2, in T3>(T1 arg1, T2 arg2, T3 arg3, T4 arg4) 無回傳值
    -  TResult Func<in T1, in T2, out TResult>(T1 arg1, T2 arg2); 有回傳值
![](https://imgur.com/EipJksD.jpg)
### 事件
> 基於委派衍生出來的語法

- 加上event關鍵字，就只能在publisher使用
- 不要有回傳值，因為可能有很多訂閱者，也不要用ref或out傳參數給發行者，因為參數可能被任意改變
- 使用EventArgs包裝參數
```C#
public delegate void TextChangedEventHandler(object sender, EventArgs args);
```
- 可以使用內建的EventHandler
```C#
public delegate void EventHandler(object sender, EventArgs eventArgs);

//宣告
public event EventHandler TextChanged;
```
- 觸發事件的時候要先檢查是否null
- EventHandlerList可以集中管理事件

### 委派與事件
#### 相同
- 支援晚期繫結
- 支援一個或多個訂閱者
- 加入與移除處理常式（handler）的語法雷同
- 都是透過方法呼叫來觸發事件和呼叫委派
#### 相異
- 事件不一定要有訂閱者
在設計元件時，如果你的元件必須呼叫訂閱者提供的方法，那麼就應該選擇使用委派。相反地，就算沒有任何訂閱者，你的元件依然能夠獨自完成所需之任務，那麼就該選擇事件
- 需要回傳值的時候，必須使用委派
宣告委派的時候，其方法可以傳回值，也可以不傳回值；而用來定義事件的委派型別，其方法通常會宣告成void，亦即沒有回傳值

### Tuple
- 舊寫法，要用item1、item2取值
- System.Tuple類別(參考型別)
```C#
static Tuple<string, int> GetInfo()
{
	return new Tuple<string,int>("AAA",50);
}

var t=GetInfo();
t.Item1.Dump();
```
- 新寫法可以自訂名稱
- System.ValueTuple(結構型別)
```C#
static (string name,int age) GetInfo2()
{
	return ("BBB",60);
}
var t3=GetInfo2();
t3.name.Dump();
```
- 自動推導名稱，但名稱不能重複
```C#
string name = "mike";
int age = 20;
var tuple = (Name: name, Age: age);// 明確指定元素名稱
tuple = (name, age); // 自動推導元素名稱
```
### 分解式
- 可以把類別物件或Tuple分解給一或多個變數
- 方法名稱一定要使用Deconstruct
```C#
public class Box
{
	public int Width {get;set;}
	public int Height {get;set;}
	
	public Box(int width,int height)
	{
		Width=width;
		Height=height;
	}
	public void Deconstruct(out int width, out int height)
	{
		width=this.Width;
		height=this.Height;
	}
}

//console
var box=new Box(20,50);
var (width,height)=box;
```
#### C# 7新語法
- 可以使用when了
```
case int[] numbers when numbers.Length > 0:
Box box when (box.Width > 0)
```
- is可以指派變數
```C#
if (aa is string s)
	{
		aa="BBB";
		
	}
```
- ref local 有點類似替身的概念
```
int temp=0;
ref int alias=ref temp;
```
- ref return
```C#
static int _number=10;
static ref int refreturn()
{
	return ref _number;
}
//接值
ref int n=ref refreturn();
```
- ref readonly可以禁止修改值
- in參數:是傳參考的且唯讀，等於readonly ref，可以改善程式效率
- private protecter:類別本身與同一組建內子類別存取

### Linq
- 惰性求值，只有以下狀況才會啟動查詢
    - 傳回單一元素或數值的操作，例如Count()、First()、Max()
    - 有內容轉換有關的的操作，例如：ToArray()、ToList()、ToDictionary()、ToLookup()