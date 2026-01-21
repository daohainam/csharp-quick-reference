# Lập trình bất đồng bộ trong C#  
*(async/await, Task, async streams)*

Chương này tập trung vào **lập trình bất đồng bộ (asynchronous)** trong C# 5.0 (.NET 4.5):

- `async` / `await` và cơ chế state machine bên dưới,
- Các kiểu trả về: `Task`, `Task<T>`, `ValueTask`, `ValueTask<T>`, `async void`,
- Cancellation, progress, exception trong async,
- **Async streams**: `IAsyncEnumerable<T>` và `await foreach`.

---

## Mục lục

- [Lập trình bất đồng bộ trong C#](#lập-trình-bất-đồng-bộ-trong-c)
  - [Mục lục](#mục-lục)
  - [1. Tổng quan: Vì sao cần async?](#1-tổng-quan-vì-sao-cần-async)
  - [2. Async method là gì?](#2-async-method-là-gì)
  - [3. Kiểu trả về của async method](#3-kiểu-trả-về-của-async-method)
    - [3.1 `Task` \& `Task<T>` – phổ biến nhất](#31-task--taskt--phổ-biến-nhất)
    - [3.2 `ValueTask` / `ValueTask<T>`](#32-valuetask--valuetaskt)
    - [3.3 `async void` – trường hợp đặc biệt](#33-async-void--trường-hợp-đặc-biệt)
  - [4. `await` hoạt động như thế nào?](#4-await-hoạt-động-như-thế-nào)
  - [5. Async state machine (ý tưởng)](#5-async-state-machine-ý-tưởng)
  - [6. Capture context \& `ConfigureAwait(false)`](#6-capture-context--configureawaitfalse)
  - [7. Exception trong async method](#7-exception-trong-async-method)
    - [7.1 Khi `await` một Task](#71-khi-await-một-task)
    - [7.2 Khi không `await` Task](#72-khi-không-await-task)
  - [8. Cancellation \& IProgress](#8-cancellation--iprogress)
    - [8.1 CancellationToken](#81-cancellationtoken)
    - [8.2 Báo tiến độ với `IProgress<T>`](#82-báo-tiến-độ-với-iprogresst)
  - [9. Best practices khi dùng async/await](#9-best-practices-khi-dùng-asyncawait)
  - [10. Async streams – `IAsyncEnumerable<T>` \& `await foreach`](#10-async-streams--iasyncenumerablet--await-foreach)
    - [10.1 Vấn đề trước khi có async streams](#101-vấn-đề-trước-khi-có-async-streams)
    - [10.2 `IAsyncEnumerable<T>` \& `IAsyncEnumerator<T>`](#102-iasyncenumerablet--iasyncenumeratort)
    - [10.3 Async iterator method](#103-async-iterator-method)
    - [10.4 Duyệt bằng `await foreach`](#104-duyệt-bằng-await-foreach)
  - [11. Async streams: Cancellation, exception, best practices](#11-async-streams-cancellation-exception-best-practices)
    - [11.1 Cancellation](#111-cancellation)
      - [Pattern 1: `[EnumeratorCancellation]` + `WithCancellation`](#pattern-1-enumeratorcancellation--withcancellation)
      - [Pattern 2: Truyền `CancellationToken` bình thường](#pattern-2-truyền-cancellationtoken-bình-thường)
    - [11.2 Exception \& dispose](#112-exception--dispose)
    - [11.3 So sánh \& best practices](#113-so-sánh--best-practices)

---

## 1. Tổng quan: Vì sao cần async?

Bài toán kinh điển:

- Bạn cần gọi HTTP, truy vấn DB, đọc/ghi file, gọi một service từ xa…
- Nếu dùng API đồng bộ (blocking), thread sẽ phải **chờ** I/O → lãng phí tài nguyên, UI treo, server khó scale.

Ví dụ blocking:

```csharp
// Blocking – thread đứng im chờ
var data = httpClient.GetStringAsync(url).Result;
```

Cách hiện đại với async/await:

```csharp
// Non-blocking – thread được trả lại để làm việc khác
var data = await httpClient.GetStringAsync(url);
```

Điểm quan trọng:

- **Bạn viết code trông như code tuần tự**, try/catch bình thường,
- Nhưng runtime/CLR sẽ:
  - Không block thread khi chờ I/O,
  - “Bẻ” method thành **state machine** + callbacks để tiếp tục sau khi I/O hoàn thành.

---

## 2. Async method là gì?

**Async method** là method:

- Có từ khóa `async`,
- Thường (nên) có ít nhất một `await`,
- Trả về `Task`, `Task<T>`, `ValueTask`, `ValueTask<T>` hoặc `void` (special case).

Ví dụ:

```csharp
public async Task<string> DownloadAsync(string url)
{
    using var client = new HttpClient();
    string content = await client.GetStringAsync(url);
    return content;
}
```

Đặc trưng:

- **Không phải** cứ `async` là chạy trên thread khác – nó **chạy trên thread hiện tại** cho tới khi gặp `await` trên một tác vụ chưa hoàn thành.
- Tại mỗi `await`:
  - Nếu tác vụ đã xong → chạy tiếp như bình thường.
  - Nếu chưa xong → method **tạm thoát ra**, trả về một `Task` cho caller, và khi tác vụ xong, nó sẽ quay lại chạy từ sau `await`.
- Một hàm async không có bất kỳ lời gọi `await` nào bên trong sẽ thực thi đồng bộ, nhưng vẫn trả về `Task` và có thể gây cảnh báo của compiler. Để chạy code thực sự bất đồng bộ trên thread khác, hãy tham khảo phần Thread.

---

Sơ đồ tuần tự (sequence diagram)
```csharp
async Task<int> FooAsync()
{
    var x = await BarAsync();      // (A)
    return x + 1;                  // (B)
}

// Gọi:
Task<int> t = FooAsync();          // (C)
int r = await t;                   // (D)
```

```
Caller                  FooAsync (SM)              BarAsync Task/Awaiter           Context/ThreadPool
  |                           |                              |                               |
  |---- call FooAsync() ----->|                              |                               |
  |                           |-- create SM & Task ----------|                               |
  |                           |-- await BarAsync() ----------> create awaiter                |
  |                           |                              |-- IsCompleted? ---------------|
  |                           |                              |            |                  |
  |                           |          (YES) inline path   |            | (NO) async path  |
  |                           |<----- GetResult() -----------|            |                  |
  |                           |-- do (B) & set result -------|            |                  |
  |<----------- Task done ----|                              |            |                  |
  |                           |                                           |-- OnCompleted(SM.MoveNext)
  |                           |                                           |-- later: completion happens
  |                           |<-------------------- MoveNext ------------(posted via captured context)
  |                           |-- GetResult(); do (B); set result ------- |
  |<----------- Task done ----|                                           |
```

## 3. Kiểu trả về của async method

Các kiểu trả về hợp lệ:

1. `Task`
2. `Task<T>`
3. `ValueTask`
4. `ValueTask<T>`
5. `void` (chỉ dùng cho event handler)

### 3.1 `Task` & `Task<T>` – phổ biến nhất

```csharp
public async Task DoWorkAsync()
{
    await Task.Delay(500);
}

public async Task<int> CalculateAsync()
{
    await Task.Delay(500);
    return 42;
}
```

Dùng:

```csharp
await DoWorkAsync();
int result = await CalculateAsync();
```

Điểm mạnh:

- Dễ compose, `await` được nhiều lần,
- Tích hợp với các API `Task`-based trong .NET.

### 3.2 `ValueTask` / `ValueTask<T>`

Tối ưu cho trường hợp:

- **Đôi khi** kết quả đã có sẵn (sync), không cần tạo `Task` mới,
- Một số API performance-critical dùng `ValueTask<T>` để giảm GC.

Ví dụ:

```csharp
public async ValueTask<int> GetCachedOrComputeAsync()
{
    if (TryGetFromCache(out var value))
        return value; // sync, không cần Task

    int computed = await ComputeAsync();
    SaveToCache(computed);
    return computed;
}
```

Nhược điểm:

- Không nên cache `ValueTask` hoặc `await` nó nhiều lần,
- API phức tạp hơn → chỉ dùng khi thật sự có lý do hiệu năng.

### 3.3 `async void` – trường hợp đặc biệt

```csharp
public async void OnButtonClick(object sender, EventArgs e)
{
    await DoWorkAsync();
}
```

**Chỉ nên dùng cho event handler**, vì:

- Không thể `await` → caller không biết khi nào xong.
- Exception không đi qua `Task`, khó bắt → có thể crash app.

Best practice: **mọi method async nên trả về `Task` hoặc `Task<T>`**, trừ event handler UI.

---

## 4. `await` hoạt động như thế nào?

Cú pháp cơ bản:

```csharp
var result = await SomeAsyncOperation();
```

Một biểu thức **awaitable** phải có:

- `GetAwaiter()` trả về một awaiter,
- Awaiter có:
  - `bool IsCompleted { get; }`
  - `void OnCompleted(Action continuation)` hoặc `UnsafeOnCompleted`
  - `T GetResult()` (hoặc `void GetResult()`)

Các kiểu phổ biến:

- `Task`, `Task<T>`, `ValueTask`, `ValueTask<T>`
- Một số type custom có `GetAwaiter()`.

Quy trình (đơn giản hoá):

1. Gọi `var awaiter = expr.GetAwaiter();`
2. Nếu `awaiter.IsCompleted`:
   - Gọi `awaiter.GetResult()` **ngay**,
   - Tiếp tục chạy code sau `await`.
3. Nếu chưa completed:
   - Đăng ký callback: `awaiter.OnCompleted(continuation)`,
   - Async method **kết thúc tạm thời**, trả về một `Task` chưa hoàn thành,
   - Khi tác vụ hoàn thành → runtime gọi `continuation` → tiếp tục method từ sau `await`.

---

## 5. Async state machine (ý tưởng)

Ví dụ method:

```csharp
public async Task<int> FooAsync()
{
    Console.WriteLine("A");
    await Task.Delay(1000);
    Console.WriteLine("B");
    return 42;
}
```

Compiler sẽ:

- Tạo một struct/class ẩn cài đặt `IAsyncStateMachine`,
- Sinh ra trường `_state` để theo dõi “đang ở đoạn nào”,
- Sinh `AsyncTaskMethodBuilder<int>` để quản lý `Task<int>` trả về,
- Sinh `MoveNext()` với một `switch(_state)`.

Ý tưởng pseudo-code (giản lược):

```csharp
struct FooAsyncStateMachine : IAsyncStateMachine
{
    public int _state;
    public AsyncTaskMethodBuilder<int> _builder;
    private TaskAwaiter _awaiter;

    public void MoveNext()
    {
        int result;
        try
        {
            if (_state == -1)
            {
                Console.WriteLine("A");
                _awaiter = Task.Delay(1000).GetAwaiter();
                if (!_awaiter.IsCompleted)
                {
                    _state = 0;
                    _builder.AwaitOnCompleted(ref _awaiter, ref this);
                    return;
                }
            }

            if (_state == 0)
            {
                _awaiter.GetResult();
            }

            Console.WriteLine("B");
            result = 42;
        }
        catch (Exception ex)
        {
            _builder.SetException(ex);
            return;
        }

        _builder.SetResult(result);
    }

    public void SetStateMachine(IAsyncStateMachine stateMachine) { }
}
```

`FooAsync` thực tế trông như:

```csharp
public Task<int> FooAsync()
{
    var sm = new FooAsyncStateMachine();
    sm._builder = AsyncTaskMethodBuilder<int>.Create();
    sm._state = -1;
    sm._builder.Start(ref sm);
    return sm._builder.Task;
}
```

Bạn không cần nhớ chi tiết, chỉ cần giữ mindset:

> `async`/`await` = compiler sinh state machine để chạy code không blocking, nhìn vẫn như code tuần tự.

---

## 6. Capture context & `ConfigureAwait(false)`

Trong môi trường có **`SynchronizationContext`** (WPF, WinForms, ASP.NET “cũ”):

```csharp
private async void Button_Click(object sender, EventArgs e)
{
    label.Text = "Loading...";
    var data = await client.GetStringAsync(url);
    // Sau await, chạy lại trên UI thread
    label.Text = data;
}
```

Mặc định:

- `await` sẽ **capture context hiện tại**,
- Khi tiếp tục, nó cố quay lại đúng context (UI thread) để bạn được phép update UI.

Để **không capture context**:

```csharp
var data = await client.GetStringAsync(url).ConfigureAwait(false);
```

- Thích hợp trong **library**, background code, nơi không phụ thuộc UI thread.
- Giúp giảm overhead, tránh deadlock trong một số pattern block không tốt.

**Gợi ý dùng:**

- Trong **application/UI**: có thể không dùng `ConfigureAwait(false)` để code đơn giản (trừ khi bạn hiểu rõ dòng chảy context).
- Trong **library** (class library, SDK): nên `await xxx.ConfigureAwait(false)` hầu hết chỗ.

---

## 7. Exception trong async method

### 7.1 Khi `await` một Task

Trong async method:

```csharp
public async Task DoAsync()
{
    try
    {
        await SomeAsyncOperation();
    }
    catch (Exception ex)
    {
        // xử lý ở đây
    }
}
```

- Nếu `SomeAsyncOperation()` trả về `Task` faulted:
  - `await` sẽ ném ra **exception gốc** (không bọc `AggregateException` – trừ khi bạn gọi `.Result`/`.Wait()`).

### 7.2 Khi không `await` Task

```csharp
var task = DoAsync(); // fire-and-forget
```

- Nếu `DoAsync` ném exception sau đó:
  - Nếu không ai `await` hoặc inspect `task.Exception`,
  - Exception có thể bị nuốt, chỉ log ra `TaskScheduler.UnobservedTaskException` (tùy runtime).

Với `async void`:

- Exception “bật” lên `SynchronizationContext` → có thể crash ứng dụng UI nếu không handle.

**Kết luận:** Trừ event handler, tất cả async method nên trả `Task`/`Task<T>` và **luôn được await** hoặc theo dõi kết quả.

---

## 8. Cancellation & IProgress

### 8.1 CancellationToken

API async “đàng hoàng” thường nhận thêm `CancellationToken`:

```csharp
public async Task DoWorkAsync(CancellationToken cancellationToken)
{
    cancellationToken.ThrowIfCancellationRequested();
    await Task.Delay(1000, cancellationToken);
    // ... các thao tác khác dùng token
}
```

Caller:

```csharp
var cts = new CancellationTokenSource();

var task = DoWorkAsync(cts.Token);
cts.Cancel(); // yêu cầu hủy

try
{
    await task;
}
catch (OperationCanceledException)
{
    Console.WriteLine("Đã hủy");
}
```

### 8.2 Báo tiến độ với `IProgress<T>`

```csharp
public async Task DownloadWithProgressAsync(IProgress<int> progress)
{
    for (int i = 0; i <= 100; i += 10)
    {
        await Task.Delay(200);
        progress.Report(i);
    }
}
```

Caller (UI):

```csharp
var progress = new Progress<int>(percent =>
{
    progressBar.Value = percent; // chạy trên UI context
});

await DownloadWithProgressAsync(progress);
```

---

## 9. Best practices khi dùng async/await

1. **Async all the way down**  
   - Đã “đi async” thì đi từ UI tới DAL.  
   - Tránh `.Result`, `.Wait()` vì dễ deadlock.

2. **Tránh `async void`** trừ event handler  
   - Dùng `Task`/`Task<T>` để caller có thể `await` & catch exception.

3. **Luôn `await` hoặc quản lý Task**  
   - Nếu không await, hãy ghi chú rõ ràng (fire-and-forget) và có logger.

4. **Không mix async với blocking sync**  
   - Tránh `Thread.Sleep`, I/O sync bên trong async.  
   - Dùng API async tương ứng (`ReadAsync`, `WriteAsync`, `SendAsync`, `SaveChangesAsync`…).

5. **Không dùng async trong property getter**  
   - Property nên “nhẹ” và trả về ngay.  
   - Dùng method `GetXxxAsync()` thay vì `async` property.

6. **Trong library: dùng `ConfigureAwait(false)`**  
   - Làm code độc lập môi trường (UI/Console/Web…), ít lỗi hơn.

7. **Đặt tên method rõ ràng**  
   - Convention: method async đặt hậu tố `Async`: `GetUserAsync`, `SaveAsync`.

---

## 10. Async streams – `IAsyncEnumerable<T>` & `await foreach`

### 10.1 Vấn đề trước khi có async streams

Trước C# 8:

- Dùng `IEnumerable<T>` / `yield return` → **stream sync**, không `await` được bên trong.
- Dùng `Task<IEnumerable<T>>` → **một cục** collection, chỉ có sau khi xong hết.

Nhưng cần:

- Đọc từng dòng file từ server qua network,
- Nhận từng message từ socket,
- Stream log / sự kiện từ DB…

→ cần **stream async**, xử lý từng phần tử khi nó đến, không phải đợi tất cả.

### 10.2 `IAsyncEnumerable<T>` & `IAsyncEnumerator<T>`

```csharp
public interface IAsyncEnumerable<out T>
{
    IAsyncEnumerator<T> GetAsyncEnumerator(CancellationToken cancellationToken = default);
}

public interface IAsyncEnumerator<out T> : IAsyncDisposable
{
    ValueTask<bool> MoveNextAsync();
    T Current { get; }
}
```

Khác:

- `MoveNextAsync()` → `ValueTask<bool>`, phải `await`.
- Có `DisposeAsync()` cho cleanup async.

### 10.3 Async iterator method

Khai báo:

- `async IAsyncEnumerable<T>` +
- `yield return` bên trong +
- Có thể `await` trong thân.

Ví dụ:

```csharp
public async IAsyncEnumerable<int> CountAsync(int from, int to, int delayMs)
{
    for (int i = from; i <= to; i++)
    {
        await Task.Delay(delayMs);
        yield return i;
    }
}
```

### 10.4 Duyệt bằng `await foreach`

```csharp
await foreach (var item in CountAsync(1, 5, 1000))
{
    Console.WriteLine(item);
}
```

Tương đương (giản lược):

```csharp
await using var e = CountAsync(1, 5, 1000).GetAsyncEnumerator();
while (await e.MoveNextAsync())
{
    var item = e.Current;
    Console.WriteLine(item);
}
```

---

## 11. Async streams: Cancellation, exception, best practices

### 11.1 Cancellation

Có hai pattern phổ biến:

#### Pattern 1: `[EnumeratorCancellation]` + `WithCancellation`

```csharp
public async IAsyncEnumerable<int> CountAsync(
    int delayMs,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    for (int i = 0; i < 1000; i++)
    {
        ct.ThrowIfCancellationRequested();
        await Task.Delay(delayMs, ct);
        yield return i;
    }
}
```

Caller:

```csharp
var cts = new CancellationTokenSource();

await foreach (var x in CountAsync(500, cts.Token)
                     .WithCancellation(cts.Token))
{
    Console.WriteLine(x);
    if (x >= 10) cts.Cancel();
}
```

#### Pattern 2: Truyền `CancellationToken` bình thường

```csharp
public async IAsyncEnumerable<int> CountAsync(
    int delayMs, CancellationToken ct)
{
    for (int i = 0; i < 1000; i++)
    {
        ct.ThrowIfCancellationRequested();
        await Task.Delay(delayMs, ct);
        yield return i;
    }
}
```

Caller:

```csharp
var cts = new CancellationTokenSource();

await foreach (var x in CountAsync(500, cts.Token))
{
    Console.WriteLine(x);
    if (x >= 10) cts.Cancel();
}
```

### 11.2 Exception & dispose

Async iterator method hỗ trợ `try/catch/finally`:

```csharp
public async IAsyncEnumerable<string> ReadLinesAsync(string path)
{
    using var stream = File.OpenRead(path);
    using var reader = new StreamReader(stream);

    try
    {
        while (!reader.EndOfStream)
        {
            var line = await reader.ReadLineAsync();
            if (line is null) yield break;
            yield return line;
        }
    }
    finally
    {
        Console.WriteLine("Done reading.");
    }
}
```

- Khi `await foreach` kết thúc (bình thường, exception hoặc cancel),
- `DisposeAsync()` được gọi, `finally` đảm bảo chạy.

### 11.3 So sánh & best practices

**`Task<IEnumerable<T>>` vs `IAsyncEnumerable<T>`**:

- `Task<IEnumerable<T>>` → lấy tất cả rồi mới xử lý.
- `IAsyncEnumerable<T>` → xử lý từng phần tử khi chúng sẵn sàng.

Chọn async stream khi:

- Dữ liệu lớn / vô hạn,
- Muốn pipeline xử lý streaming (log, message, event…).

**Best practices với async streams:**

1. Dùng khi **từng phần tử** có ý nghĩa (log, stream dữ liệu).
2. Luôn hỗ trợ **cancellation** (token).
3. Bọc `await foreach` trong `try/catch` nếu cần.
4. Đảm bảo cleanup: `using` / `await using` + `finally`.
5. Không dùng async stream khi thực chất bạn luôn cần “lấy hết rồi xử lý” → dùng `Task<List<T>>` là đủ.
6. Không lạm dụng trong logic thuần CPU sync.
