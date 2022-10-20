---
title: "保哥非同步課程"
date: 2022-10-20T16:48:10+08:00
draft: false
tags: ["Csharp","課程筆記"]
type: post
author: "Steve Lin"
description: ""
showTableOfContents: true
slug: "保哥非同步課程"
---


# 保哥非同步課程
- thread要執行要進CPU排隊，所以不一定先start的會先跑
- 非同步不一定是multi thread(user thread)，File.WriteAsync 會叫dreiver去處理寫檔，完成後會告知已完成
- 會造成deadlock的情境
	- 用windowform、WPF有內建syncronizecontext
	- 宣告方法是有用async、await關鍵字
	- 呼叫非同步方法的時候用.Result() 或wait()
- .Wait()跟.Result會block thread，不會還回去，只有用await才會還
- 沒有等待的動作，就不會有把執行緒還回去的動作
- CPU bound的工作可以用Task.Run()去實作，這種方式會去thread pool拉一條新的thread,IO bound 的用async await就可以
- thread pool會動態長大，如果電腦8核16的邏輯處理單元，一開始只會分配16個thread，把thread用完還會一直分配，但是用.Result會block thread，就會等到有Thread可以用的時候才會繼續往下做
- .net core把HttpContext.Current刪掉了,要改成注入的方式

### Task的完成狀態
- IsCompleted 這個值表示工作是否已經完成執行（無論成功或失敗都叫完成）
- IsCompletedSuccessfully這個值表示工作是否已經完成執行（只有成功才會為true）
- IsCanceled這個值表示工作是否因取消才完成執行
- IsFaulted這個值表示工作是否因未處理的例外狀況才完成執行
### Task.Status屬性列舉
- Canceled	6	
工作確認取消動作，不論是因為工作在語彙基元處於信號狀態時使用自己的 CancellationToken 擲回 OperationCanceledException，或是工作的 CancellationToken 信號在工作開始執行之前便已存在。 如需詳細資訊，請參閱工作取消。
- Created	0
工作已初始化但尚未排程。
- Faulted	7	
工作因未處理的例外狀況而完成。
- RanToCompletion	5	
工作已成功完成執行。
- Running	3	
工作正在執行，但尚未完成。
- WaitingForActivation	1	
工作正在等候由 .NET 基礎結構從內部啟動並排程。
- WaitingForChildrenToComplete	4	
工作已完成執行，而且在暗中等候附加的子工作完成。
- WaitingToRun	2	
工作已排定執行，但尚未開始執行。
[參考](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.taskstatus?view=net-6.0#system-threading-tasks-taskstatus-faulted)
![](https://imgur.com/euVRRIq.jpg)

### 會封鎖執行緒的等候方法
#### 沒有回傳值
- MethodAsync().Wait() 封鎖並等待MethodAsync 方法執行完成
- Task.WaitAll(Task[]) 封鎖並等候所有Task 工作完成
- Task.WaitAny(Task[]) 封鎖並等候任一Task 工作完成
#### 有回傳值
- MethodAsync().Result 封鎖並等待MethodAsync 方法執行完成後的回傳值

### 前景執行緒(Foreground Thread)
- 所有前景執行緒必須全部結束執行，否則處理程序無法結束執行
- 無論是否還有背景執行緒在執行，沒有前景執行緒在跑，該處理程序就一定會結束
- new Thread() 預設就是前景執行緒
### 背景執行緒(Background Thread)
- new Task() 預設就是背景執行緒
- 從執行緒集區(Thread Pool)取得的執行緒，一定是背景執行緒
- Task.Run 從執行緒集區取得一個執行緒，因此是背景執行緒
- NET 處理程序(Process)啟動後，僅會有一個前景執行緒
- 每個執行程序都可以產生出許多前景執行緒或背景執行緒

### 非同步的異常狀況發生整理
- 使用async void 會捕捉不到異常
#### thread
執行緒可以拋出例外，但是無法捕捉到，因此會造成應用程式關閉
#### Task
- 沒有透過「等待」的Task 當例外發生時，並不會向外拋出例外 
    - 直接回傳Task 的呼叫方式，等同於「沒有等待」的非同步呼叫，自然也無法透過try/catch 捕捉到例外
- 有透過「等待」機制處理的Task 才有機會透過try/catch 捕捉例外
![](https://imgur.com/xn5RSef.jpg)

### 非同步工作取消
- 使用CancellationTokenSource 使用CancellationTokenSource.Token 這個屬性取得CancellationToken
- CancellationToken.IsCancellationRequested
- CancellationToken.ThrowIfCancellationRequested
收到取消訊號，會拋出OperationCanceledException
- 可以設定取消時間
```C#
CancellationTokenSource ctsTimeout = new CancellationTokenSource();

ctsTimeout.CancelAfter(8000);
```
- 組合多個CancellationTokenSource，如果有其中一個取消，都會進到取消狀態
```C#
 CancellationTokenSource ctsUser = new CancellationTokenSource();
CancellationTokenSource ctsTimeout = new CancellationTokenSource();
CancellationTokenSource combinationCTS = CancellationTokenSource.CreateLinkedTokenSource(ctsUser.Token, ctsTimeout.Token);
傳入combinationCTS.Token
```

### 偵錯平行應用程式檢視單一執行續的呼叫堆疊
- [偵錯] [視窗] [執行緒]
- [偵錯] [視窗] [呼叫堆疊] 
- 按兩下[執行緒] 視窗中的執行緒，使它成為目前執行緒。
    - 目前執行緒具有黃色箭號。
    - 當您變更目前執行緒時，它的呼叫堆疊會出現在[呼叫堆疊] 視窗中
### Sleep跟Delay差異
- Thread.Sleep 會造成當時執行緒被封鎖(例如，UI執行緒無法執行任何指令，造成螢幕UI 控制項無法正常更新)
- Task.Delay 因為使用非同步等候方式，所以當時的執行緒是不會被封鎖的，螢幕UI 控制項可以正常更新


https://www.huanlintalk.com/2016/01/asyc-deadlock-in-aspbet.html