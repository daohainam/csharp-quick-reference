# Lập trình Thread

---

## Mục lục

- [Lập trình Thread](#lập-trình-thread)
  - [Mục lục](#mục-lục)
  - [1. Tổng quan:](#1-tổng-quan)
  - [2. `Thread` cơ bản](#2-thread-cơ-bản)
    - [2.1 Tạo \& start thread](#21-tạo--start-thread)
    - [2.2 Background vs Foreground](#22-background-vs-foreground)
    - [2.3 Join/Interrupt/Sleep/Yield](#23-joininterruptsleepyield)
    - [2.4 Priority \& đặt tên](#24-priority--đặt-tên)
  - [3. Thread Pool](#3-thread-pool)
    - [3.1 Queue công việc](#31-queue-công-việc)
    - [3.2 `Task.Run` \& quan hệ với pool](#32-taskrun--quan-hệ-với-pool)
    - [3.3 Điều chỉnh min/max threads](#33-điều-chỉnh-minmax-threads)
  - [4. Đồng bộ hóa \& bộ công cụ](#4-đồng-bộ-hóa--bộ-công-cụ)
    - [4.1 `lock`/`Monitor`](#41-lockmonitor)
    - [4.2 `Interlocked` \& `Volatile`](#42-interlocked--volatile)
    - [4.3 `ManualResetEventSlim`/`AutoResetEvent`](#43-manualreseteventslimautoresetevent)
    - [4.4 `SemaphoreSlim`](#44-semaphoreslim)
    - [4.5 `ReaderWriterLockSlim`](#45-readerwriterlockslim)
    - [4.6 `SpinLock`/`SpinWait`](#46-spinlockspinwait)
  - [5. Biến theo thread: `ThreadStatic`, `ThreadLocal<T>`, `AsyncLocal<T>`](#5-biến-theo-thread-threadstatic-threadlocalt-asynclocalt)
  - [6. Cancellation: kiểu hợp tác](#6-cancellation-kiểu-hợp-tác)
  - [7. Mẫu Producer/Consumer](#7-mẫu-producerconsumer)
    - [7.1 `BlockingCollection<T>` (sư dụng Task để chạy Thread trong ThreadPool)](#71-blockingcollectiont-sư-dụng-task-để-chạy-thread-trong-threadpool)
    - [7.2 `System.Threading.Channels`](#72-systemthreadingchannels)
  - [8. Chẩn đoán \& đo đạc](#8-chẩn-đoán--đo-đạc)
  - [9. Best practices \& cảnh báo](#9-best-practices--cảnh-báo)

---

## 1. Tổng quan:

- **Thread**: luồng OS thực thi code CPU. Bạn có thể tạo thủ công với `new Thread(...)`.  
- **Thread Pool**: nhóm luồng dùng chung. `Task.Run`, `ThreadPool.QueueUserWorkItem` **mượn** luồng tại đây để chạy.  

> Trong .NET hiện đại, ưu tiên **`async/await`** và **Task/TPL**, chỉ quay về **Thread** khi cần kiểm soát thấp‑level hoặc tác vụ đặc thù.

---

## 2. `Thread` cơ bản

### 2.1 Tạo & start thread

```csharp
using System;
using System.Threading;

void Work(object? state)
{
    Console.WriteLine($"[{Thread.CurrentThread.ManagedThreadId}] Start: {state}");
    Thread.Sleep(500);
    Console.WriteLine($"[{Thread.CurrentThread.ManagedThreadId}] Done");
}

var t = new Thread(Work); // ParameterizedThreadStart (object?)
t.Start("job-1");
t.Join();
```

**C# hiện đại** (delegate/lambda mạnh mẽ):

```csharp
var t2 = new Thread(() =>
{
    Console.WriteLine($"[{Thread.CurrentThread.ManagedThreadId}] Heavy work...");
    Thread.Sleep(200);
});
t2.Start();
t2.Join();
```

### 2.2 Background vs Foreground

```csharp
var bg = new Thread(() => Thread.Sleep(10_000)) { IsBackground = true };
bg.Start();
// Process có thể thoát dù bg chưa xong (background không giữ cho process sống).
```

### 2.3 Join/Interrupt/Sleep/Yield

```csharp
var t = new Thread(() =>
{
    try
    {
        while (true) Thread.Sleep(1000); // chờ lâu
    }
    catch (ThreadInterruptedException) { Console.WriteLine("Interrupted"); }
});
t.Start();
Thread.Sleep(500);
t.Interrupt(); // đánh thức khỏi Sleep/Wait
t.Join();
```

### 2.4 Priority & đặt tên

```csharp
var t = new Thread(() => { /* ... */ })
{
    Name = "Worker#1",
    Priority = ThreadPriority.AboveNormal
};
t.Start();
```

> **Không** nên lạm dụng Priority. Lập lịch OS & thread pool đã tối ưu tốt.

---

## 3. Thread Pool

Thread Pool là một tập hợp các thread chạy sẵn để xử lý các nhiệm vụ, sử dụng thread pool thay vì tạo mới các **Thread**
giúp chúng ta kiểm soát được số lượng thread có trong hệ thống. Vì việc chuyển đổi giữa các thread tốn kém tài nguyên, do
vậy khi hệ thống càng có nhiều thread, thời gian dành cho việc chuyển đổi càng chiếm nhiều thời gian, thread pool giúp giữ 
việc dùng các thread được hiệu quả.

### 3.1 Queue công việc

```csharp
using System.Threading;

ThreadPool.QueueUserWorkItem(_ =>
{
    Console.WriteLine($"Work on pool thread {Thread.CurrentThread.ManagedThreadId}");
});
```

### 3.2 `Task.Run` & quan hệ với pool

```csharp
await Task.Run(() => CpuBound());
```

- `Task.Run` **mượn** worker thread từ pool.  
- Tránh chạy tác vụ **blocking dài** trên pool (sẽ làm cạn pool) — dùng `TaskCreationOptions.LongRunning` để tách thread:

```csharp
var longTask = Task.Factory.StartNew(
    () => LongBlocking(), 
    CancellationToken.None,
    TaskCreationOptions.LongRunning, // gợi ý tạo dedicated thread
    TaskScheduler.Default);
```

### 3.3 Điều chỉnh min/max threads

```csharp
ThreadPool.GetMinThreads(out var minW, out var minIO);
ThreadPool.SetMinThreads(workerThreads: Math.Max(minW, Environment.ProcessorCount*2), completionPortThreads: minIO);

// Xem số còn trống
ThreadPool.GetAvailableThreads(out var availW, out var availIO);
Console.WriteLine($"Available worker={availW} IOCP={availIO}");
```

> Nâng MinThreads có thể giảm độ trễ burst, nhưng **thận trọng** để tránh thừa luồng → context switch nhiều.

---

## 4. Đồng bộ hóa & bộ công cụ

> Tham khảo các bài học về đồng bộ hóa trong khóa học .NET nền tảng.

### 4.1 `lock`/`Monitor`

```csharp
private readonly object _gate = new();
private int _counter;

void Inc()
{
    lock (_gate)
    {
        _counter++;
    }
}
```

- `lock` ≈ `Monitor.Enter/Exit` (tự động trong presence of exception).  
- **Không lock trên `this` hoặc type public** (dễ deadlock từ code ngoài).
- C# 13 (.NET 9) hỗ trợ kiểu System.Threading.Lock để lock hoạt động hiệu quả hơn.

### 4.2 `Interlocked` & `Volatile`

```csharp
int x = 0;
Interlocked.Increment(ref x);
Interlocked.Add(ref x, 10);
var old = Interlocked.Exchange(ref x, 123);
```

`Volatile.Read/Write` đảm bảo **thứ tự nhìn thấy** giữa threads:

```csharp
using System.Threading;
volatile bool _done; // hoặc Volatile.Read/Write cho field thường

void Worker()
{
    while (!Volatile.Read(ref _done)) { /* spin */ }
}
void Stop() => Volatile.Write(ref _done, true);
```

### 4.3 `ManualResetEventSlim`/`AutoResetEvent`

```csharp
var evt = new ManualResetEventSlim(false);

new Thread(() =>
{
    Console.WriteLine("Init...");
    Thread.Sleep(500);
    evt.Set(); // mở cổng cho tất cả waiter
}).Start();

evt.Wait(); // chặn tới khi Set()
Console.WriteLine("Go!");
```

- `AutoResetEvent` đánh thức **một** waiter mỗi lần `Set()`.  
- Bản `Slim` hiệu năng tốt cho in‑process; không cross-process.

### 4.4 `SemaphoreSlim`

Giới hạn **đồng thời N** tác vụ:

```csharp
var sem = new SemaphoreSlim(3);
await sem.WaitAsync();
try
{
    await WorkAsync();
}
finally
{
    sem.Release();
}
```

### 4.5 `ReaderWriterLockSlim`

Đọc song song nhiều, ghi độc quyền:

```csharp
var rw = new ReaderWriterLockSlim();
void Write(Action action)
{
    rw.EnterWriteLock();
    try { action(); } finally { rw.ExitWriteLock(); }
}
T Read<T>(Func<T> f)
{
    rw.EnterReadLock();
    try { return f(); } finally { rw.ExitReadLock(); }
}
```

### 4.6 `SpinLock`/`SpinWait`

- **Spin** hữu ích khi lock **rất ngắn** và contention **thấp** (tránh context switch).  
- **Cẩn thận** starvation; thường `lock`/`SemaphoreSlim` đủ tốt.

---

## 5. Biến theo thread: `ThreadStatic`, `ThreadLocal<T>`, `AsyncLocal<T>`

```csharp
[ThreadStatic]
static int _counterPerThread; // mỗi thread có bản sao riêng

var local = new ThreadLocal<int>(() => 42);
Console.WriteLine(local.Value); // 42 (mỗi thread khởi tạo riêng)
```

- `ThreadStatic` **không** chạy field initializer per-thread (giá trị default của T).  
- `AsyncLocal<T>` lan truyền theo **async context** (không phải theo thread thuần).

---

## 6. Cancellation: kiểu hợp tác

```csharp
var cts = new CancellationTokenSource();
var t = Task.Run(async () =>
{
    while (true)
    {
        cts.Token.ThrowIfCancellationRequested();
        await Task.Delay(100, cts.Token);
    }
}, cts.Token);

cts.Cancel();
try { await t; }
catch (OperationCanceledException) { Console.WriteLine("Canceled"); }
```

- Với `Thread`, không còn `Abort` trong .NET hiện đại (không an toàn). Hãy **hợp tác** qua `CancellationToken`/cờ tự quản.

---

## 7. Mẫu Producer/Consumer

### 7.1 `BlockingCollection<T>` (sư dụng Task để chạy Thread trong ThreadPool)

```csharp
using System.Collections.Concurrent;

var queue = new BlockingCollection<int>(boundedCapacity: 100);

// Producer
var prod = Task.Run(() =>
{
    for (int i = 0; i < 1000; i++) queue.Add(i);
    queue.CompleteAdding();
});

// Consumers (thread pool)
var consumers = Enumerable.Range(0, Environment.ProcessorCount).Select(_ => Task.Run(() =>
{
    foreach (var item in queue.GetConsumingEnumerable())
        Process(item);
})).ToArray();

await Task.WhenAll(consumers.Prepend(prod));
```

### 7.2 `System.Threading.Channels`

Hiệu năng cao, không block (async‑friendly):

```csharp
using System.Threading.Channels;

var ch = Channel.CreateBounded<int>(new BoundedChannelOptions(100)
{
    SingleWriter = false,
    SingleReader = false
});

// Producer
_ = Task.Run(async () =>
{
    for (int i = 0; i < 1000; i++)
        await ch.Writer.WriteAsync(i);
    ch.Writer.Complete();
});

// Consumer
await foreach (var item in ch.Reader.ReadAllAsync())
    Process(item);
```

---

## 8. Chẩn đoán & đo đạc

```csharp
ThreadPool.GetMaxThreads(out var maxW, out var maxIO);
ThreadPool.GetAvailableThreads(out var availW, out var availIO);
Console.WriteLine($"Pool: availW={availW}/{maxW}, availIO={availIO}/{maxIO}");
```

- Dùng **`Stopwatch`** đo thời gian; **PerfView/dotnet-trace** để phân tích contention/CPU.  
- **`ConcurrentQueue`**/`Channels` có counters hữu ích (EventSource).

---

## 9. Best practices & cảnh báo

- **Ưu tiên async I/O**; chỉ dùng thread cho CPU-bound hoặc API không async.  
- **Không** tạo quá nhiều threads — để thread pool điều phối (work‑stealing, hill‑climbing).  
- Tác vụ **blocking dài** → `TaskCreationOptions.LongRunning` hoặc **Thread** riêng.  
- **Đồng bộ tối thiểu**: dùng `Interlocked` khi đủ; chỉ `lock` khi cần vùng critical dài.  
- Dọn dẹp đúng: `CancellationToken`, `using` cho resource, `try/finally`.  
- **UI** (WPF/WinForms) có **thread affinity**: cập nhật UI trên UI thread (`SynchronizationContext/Post/Send`, `Dispatcher.Invoke`).  
- **Đo đạc trước tối ưu**; stress test để phát hiện race condition/deadlock.
- Với I/O nặng: tách pipeline I/O sang **async + Channels**, phần CPU dùng `Parallel.ForEach` để tối ưu xử lý.
