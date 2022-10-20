---
title: "非同步程式"
date: 2022-10-20T16:53:24+08:00
draft: false
tags: ["Csharp","note"]
type: post
author: "Steve Lin"
description: ""
slug: "非同步程式"
---

# 非同步程式

### 同步執行的問題
- 需要跟使用者互動的應用程式來說，如果有某項工作要花很⻑的時間才能跑完，使用者就只能在螢幕前發呆
- 某個應用程式進入無窮迴圈將導致其他應用程式暫停，整個作業系統看起來就像當掉

### 處理序
- 隔離應用程式的基本單位
- 當使用者開啟某應用程式，作業系統會將它載入記憶體並開始執行，這個載入記憶體中運行的應用程式實體（instance），便稱為處理序
- 會占用一個記憶體區塊，是獨立的虛擬位址空間，包含該應用程式的程式碼及相關資源，而且此空間只有該應用程式實體能夠存取
- 應用程式跟OS可以透過處理序分隔，但CPU沒辦法，所以要靠**執行緒**

### 執行緒
> Windows 會給每一個處理序（process）配發一個專屬的執行緒（其功能近似於CPU），而當某應用程式進入無窮迴圈，其所屬之處理序形同凍結，但其他處理序（擁有各自的執行緒）並未凍結；它們都還能夠繼續運行
- 執行緒是Windows 作業系統用來虛擬化CPU 的概念
- 執行緒就是用來切割CPU的基本單位，可以同時執行多項工作
- 一個處理序可以同時跑多個執行緒
- 建一條thread要配置1MB的記憶體，包含執行緒核心物件、環境區塊（Thread Environment Block）、使用者模式堆疊、核心模式堆疊等等

### Context Switch
- 切割並分配CPU運算時間給應用程式的各個執行緒
- 把CPU內部各暫存器的值保存至目前執行緒的內部資料結構
- 挑選下一個"幸運"的執行緒。若該執行緒屬於另一個處理序，則在切換之前，Windows 還必須切換虛擬位址空間，這樣CPU 才能存取到正確的程式碼和資料
- 從選中的執行緒之內部資料結構載入CPU暫存器的值
- 建立、摧毀、管理和切換執行緒都得額外消耗一些記憶體空間，所以沒必要不要多建額外的thread
- CLR 回收資源時，它會先暫停所有的執行緒，等到回收動作完成後才恢復

### 執行緒優先順序
- 優先順序分成32個等級，0-31
- 優先權越高愈能分到更多CPU時間
- 等級低的會發生starvation的情況，容易分不到資源，但是多核心的機器可以減少這種狀況
- 用處理序的優先順序類別跟執行緒的優先順序決定等級
#### 處理序的優先順序
- 即時（RealTime）
    - 可能會影響OS正常運作 
- 高（High）
    - I/O 執行時間短的才適用 
- 高於標準（Above Normal）
- 標準（Normal）
- 低於標準（Below Normal）
- 閒置（Idle）
```C#
var p=Process.GetCurrentProcess();
p.PriorityClass=ProcessPriorityClass.High;
```
#### 執行緒的優先順序
- 閒置（Idle）
- 最低（Lowest
- 低於正常（Below Normal）
- 正常（Normal）
- 高於正常（Above Normal）
- 最高（Highest）
- 時間緊迫（Time-Critical）
![](https://2.bp.blogspot.com/-FIDFRjeEI6U/UYfStTRCwsI/AAAAAAAAG2w/vEleYH4DbyY/s1600/thread+priority+table.png)
```C#
var t = new Thread(() => { "aaa".Dump();});
t.Priority=ThreadPriority.Highest;
```
### 名詞解釋
- 並行（concurrency）：一次處理多件工作。例如：使用者一邊輸入文字，應用程式在背後一邊執行拼字檢查
- 多執行緒（multithreading）：特別指以多執行緒的方式來實現並行（concurrency）
- 平行處理（parallel processing）：把工作切分成多個小單位，並分別交給多條執行緒來同時執行
- 非同步處理（asynchronous processing）：是並行（concurrency）的一種形式，但不必然（甚至會避免）使用執行緒，而是採用「承諾」（promise ）或回呼事件（callback event）的方式來達到並行的效果
- 非同步工作會在將來某個時間執行完畢，執行這些工作的時候不會擋住當前的執行緒，也就是說呼叫非同步工作的執行緒仍然可以繼續執行其它工作，等非同步工作完成會透過callback event/promise/

### 專屬執行緒dedicated thread
> 直接建立一條執行緒專門執行某件工作
- 命名空間System.Threading.Thread
- 每個應用程式都有預設執行緒，稱為主執行緒main thread
```C#
//建立工作執行緒
Thread t1 = new Thread(MyTask);
t1.Start();

//要傳入委派方法
public delegate void ThreadStart();
public delegate void ParameterizedThreadStart(Object obj);
```
- Thread有個IsAlive屬性，代表執行緒是否正在運行
- 可以用Join方法，主執行緒會等待完成才繼續執行之後的程式
### 執行緒共享變數
- 如果有一個全域變數會讓兩個執行緒共用，取值沒問題，但是修改變數值會有問題
- 遇到要同時修改變數的情況，可以暫時讓執行緒同步，也就是排隊
### 執行緒同步化（thread synchronization）
- 用lock鎖定:直接把要同步的程式碼區塊包起來，等鎖定的物件被釋放才能換另一個
- 其它同步化技巧，slim結尾是比較新也輕量化
    -  Mutex
    -  SemaphoreSlim(非同步鎖)
    -  AutoResetEvent
    -  ManualResetEventSlim
    -  CountDownEvent
    -  Barrier
    -  ReaderWriterLockSlim
    -  SpinWait
```C#
private object locker = new Object();
lock (locker)
{
//code
}
```
```C#
//宣告
private static SemaphoreSlim semaphoreSlim = new SemaphoreSlim(1, 1);

 await semaphoreSlim.WaitAsync();
 try
 {
 //code
 }
 finally
 {
 semaphoreSlim.Release();
 }
```
### 前景執行緒vs. 背景執行緒
- 當某個應用程式中所有的前景執行緒都停止時，CLR 會停止該應用程式的所有背景執行緒（而且不會拋出任何異常），並結束應用程式
- 若只是停止背景執行緒，不會造成應用程式結束
- 一定要執行完的可以放前景，可以中斷或不重要的放背景執行
```C#
Thread t = new Thread(MyTask);
t.IsBackground = true;
```

### 使用專屬執行緒的時機
- 欲執行的工作需要花較⻑時間才能執行完畢（例如10 分鐘以上）
- 希望某些執行緒擁有特殊優先權(不建議)，預設等級是"正常"，如果要有特權可以個別建立並設定
- 希望某些執行緒以前景執行緒的方式運作，以避免工作還沒完成，應用程式就被使用者或其他程序關閉(執行緒集區的執行緒都是背景執行，有可能被CLR結束)
- 執行緒開始工作後，你可能需要在某些情況下提前終止執行緒(用Thread的Abort方法)

### 執行緒集區Thread Pool
> .net CLR實做集區的概念，讓應用程式可將已完成任務的執行緒丟進集區裡面待命，等到有其它非同步工作需要執行，就可以透過集區中閒置的執行緒來負責執行
#### 運作方式
- 每一個CLR 有一個執行緒集區，而且那個CLR 管理的所有App Domains 都會共享這個執行緒集區。如果一個應用程式會同時載入多個CLR，則每一個CLR 都有它自己的執行緒集區
- 內部有一個工作請求佇列，當有需要非同步操作時，可以呼叫api把工作傳進queue
- 一開始pool是空的，CLR會建新的thread來負責，等執行完CLR不會摧毀，而是放到pool休息，等下一次使用
- 在集區中的執行緒如果閒置一段時間沒使用會自動摧毀
- 佇列中的工作如果大量增加，導致集區的執行緒來不及消化，會決定是否要加入新的執行緒

#### 冷知識
.NET 3.5每個CPU核心可以有250條執行緒，4.0改善，會根據機器的記憶體大小來決定執行緒集區的上限，**通常最多可有1023條工作執行緒，以及1000 條I/O 執行緒**

### 工作執行緒與I/O執行緒(輸入/輸出)
- worker thread pool負責執行與CPU 運算有關的工作
- I/O thread pool負責處理I/O操作，讀寫檔案、網路I/O、資料庫處理等等
- System.Threading.Timer：適用於定期執行特定背景工作的場合
```C#
//工作執行緒 會把工作加進佇列
static Boolean QueueUserWorkItem(WaitCallback callBack);
static Boolean QueueUserWorkItem(WaitCallback callBack, Object state);

delegate void WaitCallback(Object state);

//sample
ThreadPool.QueueUserWorkItem(MyTask);
```
### 基於事件的非同步模式（EAP）
> Event-based Asynchronous Pattern
- 每一項非同步操作都會有兩個成員：一個是用來起始非同步工作的方法，另一個則是工作完成時觸發的事件
- 要先設定好非同步工作完成時要callback 的事件處理常式。此步驟必須在呼叫非同步方法之前進行
- 呼叫非同步方法，會另外建一條執行緒，而且立刻返回呼叫端，等到非同步工作完成，會主動呼叫先前預設好的事件處理常式

```C#
 using (var client = new WebClient())
            {
                client.DownloadStringCompleted += WebDownloadStringCompleted;
                client.DownloadStringAsync(new Uri("http://huan-lin.blogspot.com"));
            }
```
### 基於工作的非同步模式（TAP）
> Task-based Asynchronous Pattern
- 依賴Task Parallel Library(TPL)
    -  System.Threading
    -  System.Threading.Tasks
- 基礎是Task和Task< TResult>
- 一個由Task代表的非同步工作是由工作排程器task scheduler來決定執行策略(命名空間System.Threading.Tasks.TaskScheduler)
- 工作排程器是以thread pool為基礎，向thread pool要求一個worker thread
- 用Task類別建立非同步工作會用到執行緒，async/await則不見得會用到
- 如果要等多項工作完成，可以用`Task.WaitAll()`或`Task.WaitAny()`
```
//先建立task 要執行才start
var task = new Task(MyTask);
task.Start();
//task.Wait(); 如果要等他執行完
```

```
Task task = Task.Run(() =>
                {
                    for (int i = 0; i < 500; i++)
                    {
                        Console.Write("[" + Thread.CurrentThread.ManagedThreadId + "]");
                    }
                });
```
#### 冷知識-三種適合非同步處理的場合
- I/O 處理，包括檔案存取、網路通訊、資料庫存取（通常包含檔案和網路）等等。比如說，常見的web service 呼叫就是屬於網路I/O
- 是類似儀表板（dashboard）之類的頁面，它需要一次呈現許多區塊，而各區塊的內容又是透過其他檔案或遠端網路呼叫的方式取得，非同步方式先取得資料就先顯示
- 需要⻑時間處理的HTTP 請求。比如說，收到某個HTTP 請求時，必須啟動一個⻑時間的工作，並且等待那件工作執行完畢，才將工作的結果傳回用戶端
- Task物件代表非同步工作，但不等同於執行緒，用她的時候CLR會自動幫你管理執行緒

### Async/await
- 使用await 關鍵字來等待非同步工作執行完畢，然後取得其執行結果
- 使用await的地方會暫且記住所在的位置，並且令程式控制流程立刻返回呼叫端繼續往下執行
- 直到呼叫端需要取得非同步工作的執行結果時，控制流才又會回到剛剛await 暫且保留而未執行的程式碼，等待那個非同步工作執行完畢，並傳回其執行結果。
- task.Result是個阻斷式呼叫，必須等待目標工作完成才能繼續往下執行
![](https://raw.githubusercontent.com/huanlin/async-book-support/master/Images/ch03/async-await-flow.org.png)
- 程式碰到await的時候控制流才會切開成兩個，await之前是一個同步執行的，await之後的是另一個同步執行的，分屬不同控制流
- async方法參數不能使用ref或out修飾詞
- 方法不能宣告成async void，因為會捕捉不到例外錯誤
- 碰到await會切換成另一條Thread，*但只限於console類的應用程式*

### SynchronizationContext
> 是System.Threading 命名空間裡的一個類別，它代表當時的同步環境資訊
> 用途在簡化非同步工作間的執行緒切換操作
- window form或ASP.NET有UI的應用程式，跟UI有關的操作，最後都要回到UI執行緒上面執行
- async和await 的一個好處在於它使用了SynchronizationContext來確保await之後的延續工作總是在呼叫await敘述之前的同步環境中執行
- window form控制項要靠[Invoke方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.windows.forms.control.invoke?redirectedfrom=MSDN&view=net-5.0#System_Windows_Forms_Control_Invoke_System_Delegate_)切換回UI執行緒
- await敘述會記住當前所在位置，就是靠`SynchronizationContext.Current`屬性取得當下環境資訊，如果不是null，就會以他做為當前環境資訊，若是null，就會以當前的TaskScheduler物件決定其後續的執行緒環境
- console應用程式`SynchronizationContext.Current`一定是null，所以碰到await會切換成另一條thread
- ASP .NET(AspNetSynchronizationContext)、WPF(DispatchSynchronizationContext)、
Windows Forms(WindowsFormsSynchronizationContex)
- 建議不要使用`.Result`去取值，因為會鎖死context，可以改用`ConfigureAwait(false)`代表不要用之前獲得的SynchronizationContext來恢復執行await之後的程式
- ASP .NET core中沒有SynchronizationContext物件

### 不要寫假的非同步方法!
不要用`Task.Run()`把同步方法位造成非同步，因為Task類別會向thread pool調動一個工作執行緒來執行工作，如果大量使用可能會有效能或延展性（scalability）不佳的問題

### 小觀念
- await等待的是某個方法傳回的Task物件，不管那個方法是不是async方法，只要回傳物件類型是Task或TasK< T >就可以被await
- 建構式不能有async關鍵字，但可以寫非同步工廠方法來建
- 非同步延遲Task.Sleep(1000)不會阻擋當前控制流，Thread.Sleep()會擋
- Task.WhenAll可以等待一組工作完成，如果都是同一種型別，會回傳一個陣列
- Task.WhenAny等待任意一個工作完成，只回傳最快執行完的那個工作，其它工作會繼續執行，只是結果會被忽略
- 設定工作的延續使用`ContinueWith()`
```C#
 var taskA = Task.Run(() => Console.WriteLine("起始工作...."));

Task taskB = taskA.ContinueWith( antecedentTask => 
                {
                    Console.WriteLine("從 Task A 接續 Task B.");
                    System.Threading.Thread.Sleep(4000); // 等待幾秒
                } );
```
- Task​Continuation​Options設定前導工作發生狀況時才觸發某種工作
- 在同步方法中呼叫非同步方法，可以用微軟提供的[AsyncHelper](https://github.com/IdentityServer/IdentityServer3.EntityFramework/blob/master/Source/Core.EntityFramework/Serialization/AsyncHelper.cs)
### 前景背景
- Thread是前景執行緒
- Task是背景執行緒
- 所有前景執行緒關閉，process也會關閉


### 小記
網站開發美學有講非同步的處理原理，如果是CPU的作業用同步方法比較適合，因為用非同步不會節省時間，IO bound或網路IO的作業才用非同步