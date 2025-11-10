# Exception & Error Handling trong C#

## Mục lục

- [Exception \& Error Handling trong C#](#exception--error-handling-trong-c)
  - [Mục lục](#mục-lục)
  - [1. Tổng quan \& triết lý](#1-tổng-quan--triết-lý)
  - [2. Cú pháp cơ bản: `try`/`catch`/`finally`/`throw`](#2-cú-pháp-cơ-bản-trycatchfinallythrow)
  - [3. Hệ phân cấp ngoại lệ \& các ngoại lệ phổ biến](#3-hệ-phân-cấp-ngoại-lệ--các-ngoại-lệ-phổ-biến)
  - [4. Bắt ngoại lệ đúng cách](#4-bắt-ngoại-lệ-đúng-cách)
    - [4.1 Bắt cụ thể trước, tổng quát sau](#41-bắt-cụ-thể-trước-tổng-quát-sau)
    - [4.2 Không bắt `Exception` vô tội vạ](#42-không-bắt-exception-vô-tội-vạ)
    - [4.3 `catch` rỗng là mùi hôi (code smell)](#43-catch-rỗng-là-mùi-hôi-code-smell)
  - [5. Rethrow, bọc \& bảo toàn stack trace](#5-rethrow-bọc--bảo-toàn-stack-trace)
    - [5.1 Rethrow đúng](#51-rethrow-đúng)
    - [5.2 Bọc ngoại lệ với InnerException](#52-bọc-ngoại-lệ-với-innerexception)
    - [5.3 Bảo toàn stack trace thủ công (nâng cao)](#53-bảo-toàn-stack-trace-thủ-công-nâng-cao)
  - [6. Exception filter với `when`](#6-exception-filter-với-when)
  - [7. Quản lý tài nguyên: `finally`, `using`, `await using`](#7-quản-lý-tài-nguyên-finally-using-await-using)
    - [7.1 `finally` để đảm bảo thu hồi](#71-finally-để-đảm-bảo-thu-hồi)
    - [7.2 `using` \& `await using` (khuyến nghị)](#72-using--await-using-khuyến-nghị)
  - [8. Thiết kế ngoại lệ tuỳ biến (Custom exceptions)](#8-thiết-kế-ngoại-lệ-tuỳ-biến-custom-exceptions)
    - [8.1 Khi nào cần?](#81-khi-nào-cần)
    - [8.2 Hướng dẫn](#82-hướng-dẫn)
  - [9. Ngoại lệ trong async/await \& song song](#9-ngoại-lệ-trong-asyncawait--song-song)
    - [9.1 `await` *unwrap* ngoại lệ](#91-await-unwrap-ngoại-lệ)
    - [9.2 `.Result` / `.Wait()` bọc trong `AggregateException`](#92-result--wait-bọc-trong-aggregateexception)
    - [9.3 Nhiều task: `Task.WhenAll`](#93-nhiều-task-taskwhenall)
    - [9.4 `async void` \& unobserved exceptions](#94-async-void--unobserved-exceptions)
    - [9.5 TPL/Parallel/PLINQ](#95-tplparallelplinq)
  - [10. Cancellation vs Exception](#10-cancellation-vs-exception)
  - [11. Global handling \& logging](#11-global-handling--logging)
    - [11.1 Ứng dụng desktop (WPF/WinForms)](#111-ứng-dụng-desktop-wpfwinforms)
    - [11.2 ASP.NET Core](#112-aspnet-core)
    - [11.3 Worker/Service](#113-workerservice)
  - [12. Hiệu năng \& quy tắc sử dụng ngoại lệ](#12-hiệu-năng--quy-tắc-sử-dụng-ngoại-lệ)
  - [13. Kiểm tra tràn số: `checked` / `unchecked`](#13-kiểm-tra-tràn-số-checked--unchecked)
  - [14. Cheat sheet \& checklist](#14-cheat-sheet--checklist)

---

## 1. Tổng quan & triết lý

- Ngoại lệ là **luồng điều khiển bất thường** dùng để báo lỗi **không mong đợi**.  
- **Không dùng ngoại lệ cho luồng điều khiển bình thường** (ví dụ kiểm tra tồn tại file, parse… → ưu tiên `TryXxx`).  
- Ngoại lệ **tốn chi phí**: ném/bắt tạo stack trace → tránh ở hot-path.

---

## 2. Cú pháp cơ bản: `try`/`catch`/`finally`/`throw`

```csharp
try
{
    DoWork();
}
catch (IOException ex)
{
    Log(ex);
    // xử lý/khôi phục
}
finally
{
    Cleanup(); // luôn chạy, kể cả khi có/không có exception
}
```

- `finally` **luôn** chạy (trừ khi process bị terminate).  
- `throw;` **ném lại** ngoại lệ hiện tại, giữ stack trace.

Ném ngoại lệ:

```csharp
if (arg is null)
    throw new ArgumentNullException(nameof(arg));

throw new InvalidOperationException("State is invalid");
```

---

## 3. Hệ phân cấp ngoại lệ & các ngoại lệ phổ biến

```
System.Object
└─ System.Exception
   ├─ System.SystemException
   │  ├─ NullReferenceException
   │  ├─ IndexOutOfRangeException
   │  ├─ InvalidOperationException
   │  ├─ ArgumentException
   │  │  ├─ ArgumentNullException
   │  │  └─ ArgumentOutOfRangeException
   │  ├─ NotSupportedException
   │  ├─ FormatException
   │  ├─ OverflowException
   │  ├─ DivideByZeroException
   │  ├─ TimeoutException
   │  └─ ...
   ├─ System.IO.IOException (và các ngoại lệ con)
   ├─ System.Net.Http.HttpRequestException
   ├─ System.Threading.Tasks.TaskCanceledException
   ├─ System.OperationCanceledException
   └─ (ngoại lệ tuỳ miền / thư viện khác)
```

**Gợi ý nhận diện nhanh** (không đầy đủ):
- **Lập trình phòng vệ/đầu vào**: `ArgumentNullException`, `ArgumentOutOfRangeException`, `FormatException`  
- **Trạng thái sai**: `InvalidOperationException`, `NotSupportedException`  
- **I/O**: `IOException` (file lock, mất kết nối…), `FileNotFoundException`, `DirectoryNotFoundException`  
- **Số học**: `DivideByZeroException`, `OverflowException` (khi `checked`)  
- **Thời gian**: `TimeoutException`  
- **Mạng/HTTP**: `HttpRequestException`  
- **Huỷ**: `OperationCanceledException` / `TaskCanceledException`

---

## 4. Bắt ngoại lệ đúng cách

### 4.1 Bắt cụ thể trước, tổng quát sau

```csharp
try
{
    Save();
}
catch (IOException ex)
{
    // xử lý cho I/O
}
catch (Exception ex) // tổng quát, đặt sau
{
    // fallback / log
    throw; // thường rethrow để không nuốt lỗi
}
```

### 4.2 Không bắt `Exception` vô tội vạ
- Chỉ bắt ở **biên rìa** (UI, job runner, middleware) để log và hiển thị thông báo thân thiện.  
- Ở tầng trong (domain, repository…), hãy để lỗi **bubble up**.

### 4.3 `catch` rỗng là mùi hôi (code smell)

```csharp
catch
{
    // nuốt lỗi, gây khó debug
}
```

Hãy **ít nhất log** hoặc chuyển sang trạng thái an toàn.

---

## 5. Rethrow, bọc & bảo toàn stack trace

### 5.1 Rethrow đúng

```csharp
catch (Exception)
{
    // Do something...
    throw; // ✅ giữ nguyên stack
}
```

**Tránh**: `throw ex;` vì sẽ **reset** stack trace.

### 5.2 Bọc ngoại lệ với InnerException

```csharp
try
{
    ParseConfig();
}
catch (FormatException ex)
{
    throw new ConfigurationException("Config invalid", ex);
}
```

- Cung cấp **ngữ cảnh miền** giàu thông tin (`ConfigurationException`) nhưng **không làm mất** exception gốc (`InnerException`).

### 5.3 Bảo toàn stack trace thủ công (nâng cao)

```csharp
using System.Runtime.ExceptionServices;

try { ... }
catch (Exception ex)
{
    // custom logic...
    ExceptionDispatchInfo.Capture(ex).Throw(); // giữ stack
    throw; // unreachable
}
```

---

## 6. Exception filter với `when`

Lọc điều kiện **trước khi** vào khối `catch`:

```csharp
try
{
    await SendAsync();
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.TooManyRequests)
{
    await Task.Delay(1000);
    // retry nhẹ
}
```

- Không đổi stack trace (khác với `if` bên trong `catch`).  
- Hữu ích cho **retry**, **log có điều kiện**, **phân loại xử lý**.

---

## 7. Quản lý tài nguyên: `finally`, `using`, `await using`

### 7.1 `finally` để đảm bảo thu hồi

```csharp
Stream? s = null;
try
{
    s = File.OpenRead(path);
    // ...
}
finally
{
    s?.Dispose();
}
```

### 7.2 `using` & `await using` (khuyến nghị)

```csharp
using var stream = File.OpenRead(path);
using var reader = new StreamReader(stream);
string text = reader.ReadToEnd();
```

Async:

```csharp
await using var conn = await OpenConnectionAsync();
var rows = await conn.QueryAsync(...);
```

- Dựa trên `IDisposable` / `IAsyncDisposable`.  
- **Đảm bảo** giải phóng kể cả có ngoại lệ.

> Xem thêm: mẫu *Dispose pattern* với resource unmanaged (nếu cần).

---

## 8. Thiết kế ngoại lệ tuỳ biến (Custom exceptions)

### 8.1 Khi nào cần?
- Cần thêm **ngữ cảnh nghiệp vụ** (domain-specific).  
- Muốn caller **bắt riêng** lỗi của domain.

### 8.2 Hướng dẫn
- Kế thừa trực tiếp từ `Exception` (hoặc một nhánh hợp lý).  
- Đặt hậu tố `Exception`, có ctor chuẩn.

```csharp
[Serializable]
public class ConfigurationException : Exception
{
    public ConfigurationException() { }
    public ConfigurationException(string message) : base(message) { }
    public ConfigurationException(string message, Exception inner) : base(message, inner) { }
    protected ConfigurationException(
      System.Runtime.Serialization.SerializationInfo info,
      System.Runtime.Serialization.StreamingContext context) : base(info, context) { }
}
```

> Với .NET hiện đại, `[Serializable]`/ctor serialization chỉ cần nếu bạn thật sự cần cross-appdomain/interop cũ.

---

## 9. Ngoại lệ trong async/await & song song

### 9.1 `await` *unwrap* ngoại lệ
```csharp
try
{
    await TaskThatFailsAsync();
}
catch (Exception ex)
{
    // ex là exception gốc, KHÔNG phải AggregateException
}
```

### 9.2 `.Result` / `.Wait()` bọc trong `AggregateException`
```csharp
try
{
    TaskThatFailsAsync().Wait(); // hoặc .Result
}
catch (AggregateException aex)
{
    foreach (var ex in aex.Flatten().InnerExceptions)
        Log(ex);
}
```

### 9.3 Nhiều task: `Task.WhenAll`
```csharp
try
{
    await Task.WhenAll(tasks);
}
catch
{
    // một hay nhiều task lỗi → Exception từ task đầu tiên; có thể duyệt tasks để lấy tất cả
    var faults = tasks.Where(t => t.IsFaulted).Select(t => t.Exception).ToList();
}
```

### 9.4 `async void` & unobserved exceptions
- `async void` **chỉ** dùng cho event handler.  
- Lỗi trong `async void` đi vào `SynchronizationContext` → có thể **crash** app nếu không bắt.

### 9.5 TPL/Parallel/PLINQ
- `Parallel.For/ForEach` & `Parallel.Invoke` ném `AggregateException`.  
- PLINQ (`AsParallel()`) lỗi cũng gói trong `AggregateException`.

---

## 10. Cancellation vs Exception

- Huỷ là **tình huống dự kiến** → dùng `CancellationToken`.  
- Khi token bị huỷ: ném `OperationCanceledException` (hoặc `TaskCanceledException`) **kèm token**.

```csharp
public async Task WorkAsync(CancellationToken ct)
{
    ct.ThrowIfCancellationRequested();
    await Task.Delay(500, ct);
}
```

- Caller **phân biệt** giữa lỗi thật và huỷ:

```csharp
try
{
    await WorkAsync(cts.Token);
}
catch (OperationCanceledException) when (cts.IsCancellationRequested)
{
    // coi như “bình thường”
}
```

---

## 11. Global handling & logging

### 11.1 Ứng dụng desktop (WPF/WinForms)
- `AppDomain.CurrentDomain.UnhandledException` – bắt ngoại lệ **không bắt được** (cuối).  
- WPF: `Application.DispatcherUnhandledException`  
- WinForms: `Application.ThreadException`

> Dùng để **log & báo lỗi thân thiện**, không nên tiếp tục chạy nếu trạng thái không an toàn.

### 11.2 ASP.NET Core
- Dùng middleware `UseExceptionHandler` cho production; `UseDeveloperExceptionPage` cho dev.  
- Log qua `ILogger`. Trả về response chuẩn hoá (ProblemDetails).

### 11.3 Worker/Service
- Quấn entrypoint bằng `try/catch` tổng → log + exit code phù hợp.  
- Lập lịch job: đảm bảo **retry policy** hợp lý (exponential backoff, circuit breaker).

---

## 12. Hiệu năng & quy tắc sử dụng ngoại lệ

- **Không dùng ngoại lệ cho luồng thường** → thiết kế API theo cặp:
  - `Parse` **ném lỗi** khi input sai nghiêm trọng;  
  - `TryParse` **không ném**, trả `bool` + `out`.
- **Guard clauses** sớm với `ArgumentNullException.ThrowIfNull(arg);`.  
- **Thông báo rõ ràng**: message ngắn gọn, nêu *ngữ cảnh* và *cách khắc phục* nếu có.  
- **Không lạm dụng bắt/đổi ngoại lệ**: chỉ bọc khi bạn **thêm giá trị ngữ cảnh**.  
- **Đừng nuốt lỗi** (catch rỗng) – luôn log/lộ diện ở rìa hệ thống.  
- **Đo lường**: exceptions đắt đỏ → tránh ném trong vòng lặp nội, hot path.

---

## 13. Kiểm tra tràn số: `checked` / `unchecked`

```csharp
int a = int.MaxValue;
int b = 1;

checked
{
    // ném OverflowException
    int c = a + b;
}

unchecked
{
    // tràn im lặng (wrap-around)
    int d = a + b;
}
```

- Có thể bật `checked` theo **khối**, **biểu thức**, hoặc **mức project**.

---

## 14. Cheat sheet & checklist

**Bắt đầu**  
- [ ] Bắt ngoại lệ **cụ thể** trước, tổng quát sau.  
- [ ] Không bắt `Exception` tràn lan; chỉ ở rìa app để log/hiển thị.  
- [ ] Luôn `throw;` khi rethrow, **không** `throw ex;`.  
- [ ] Dùng `when` để lọc điều kiện (retry, throttle…).

**Tài nguyên**  
- [ ] Ưu tiên `using` / `await using` cho `IDisposable` / `IAsyncDisposable`.  
- [ ] Chỉ dùng finalizer khi cần tài nguyên unmanaged.

**Async/Parallel**  
- [ ] Trong async, **luôn** `await` Task (tránh nuốt lỗi).  
- [ ] Tránh `async void` (trừ event).  
- [ ] `WhenAll` → biết cách thu thập nhiều lỗi (`IsFaulted`, `Exception`).

**Thiết kế API**  
- [ ] Phân biệt lỗi business vs lỗi hệ thống.  
- [ ] Dùng `TryXxx` cho lỗi dự kiến/nhẹ; `Xxx` ném exception cho điều kiện bất thường.  
- [ ] Custom exception có tên rõ, ctor đầy đủ, bọc `InnerException` khi cần.

**Hiệu năng & logging**  
- [ ] Log đủ ngữ cảnh (request-id, user, input chính).  
- [ ] Tránh ném/bắt liên tục trong hot path.  
- [ ] Theo dõi `UnhandledException`/`UnobservedTaskException` để phát hiện rò rỉ lỗi.

---

> Kết hợp chương này với **Methods**, **Async**, **Type System** sẽ giúp bạn xây dựng API rõ ràng, an toàn và dễ vận hành.
