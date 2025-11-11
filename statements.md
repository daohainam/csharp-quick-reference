# Statements trong C#

---

## Mục lục

- [Statements trong C#](#statements-trong-c)
  - [Mục lục](#mục-lục)
  - [1. Tổng quan \& phân loại](#1-tổng-quan--phân-loại)
  - [2. Top‑level statements (C# 9)](#2-toplevel-statements-c-9)
  - [3. Khối lệnh `{ ... }`, phạm vi \& lifetime](#3-khối-lệnh----phạm-vi--lifetime)
  - [4. Declaration statements](#4-declaration-statements)
    - [4.1 Khai báo biến \& suy luận kiểu (`var`, target-typed)](#41-khai-báo-biến--suy-luận-kiểu-var-target-typed)
    - [4.2 `const` \& `readonly` (local)](#42-const--readonly-local)
    - [4.3 Deconstruction declaration](#43-deconstruction-declaration)
    - [4.4 `ref` local \& `ref readonly` local](#44-ref-local--ref-readonly-local)
    - [4.5 `using` declaration (C# 8+) \& `await using`](#45-using-declaration-c-8--await-using)
    - [4.6 `scoped` (C# 11, nâng cao)](#46-scoped-c-11-nâng-cao)
  - [5. Expression statements](#5-expression-statements)
  - [6. Selection statements: `if`/`else`, `switch`](#6-selection-statements-ifelse-switch)
    - [6.1 `if` / `else`](#61-if--else)
    - [6.2 `switch` (statement) — pattern matching hiện đại](#62-switch-statement--pattern-matching-hiện-đại)
  - [7. Iteration statements: `while`, `do`, `for`, `foreach`, `await foreach`](#7-iteration-statements-while-do-for-foreach-await-foreach)
    - [7.1 `while` / `do`](#71-while--do)
    - [7.2 `for`](#72-for)
    - [7.3 `foreach`](#73-foreach)
    - [7.4 `await foreach` (C# 8) — async streams](#74-await-foreach-c-8--async-streams)
  - [8. Jump statements: `break`, `continue`, `return`, `throw`, `goto`, `yield`](#8-jump-statements-break-continue-return-throw-goto-yield)
    - [8.1 `break` / `continue`](#81-break--continue)
    - [8.2 `return`](#82-return)
    - [8.3 `throw`](#83-throw)
    - [8.4 `goto` \& labeled statement](#84-goto--labeled-statement)
    - [8.5 `yield return` / `yield break` (iterator method)](#85-yield-return--yield-break-iterator-method)
  - [9. Exception handling \& resource: `try`/`catch`/`finally`, `using`/`await using`](#9-exception-handling--resource-trycatchfinally-usingawait-using)
    - [9.1 `try` / `catch` / `finally` + filter `when`](#91-try--catch--finally--filter-when)
    - [9.2 `using` statement (khối) vs `using` declaration](#92-using-statement-khối-vs-using-declaration)
  - [10. Đồng bộ hoá: `lock`](#10-đồng-bộ-hoá-lock)
  - [11. Kiểm soát tràn \& môi trường: `checked`/`unchecked`, `unsafe`/`fixed`](#11-kiểm-soát-tràn--môi-trường-checkedunchecked-unsafefixed)
  - [12. Local functions](#12-local-functions)
  - [13. Empty \& labeled statements](#13-empty--labeled-statements)
  - [14. Mẹo \& best practices](#14-mẹo--best-practices)
    - [Phụ lục: Bộ ví dụ “từ đơn giản đến nâng cao”](#phụ-lục-bộ-ví-dụ-từ-đơn-giản-đến-nâng-cao)

---

## 1. Tổng quan & phân loại

- **Statement**: đơn vị thực thi của ngôn ngữ (khác *expression* là biểu thức cho giá trị).  
- Các nhóm chính:
  - **Declaration**: khai báo biến/const/`using` declaration…
  - **Expression**: lời gọi hàm, gán, `await`…
  - **Selection**: `if`/`else`, `switch` (hỗ trợ **pattern matching** hiện đại).
  - **Iteration**: `while`, `do`, `for`, `foreach`, `await foreach`.
  - **Jump**: `break`, `continue`, `return`, `throw`, `goto`, `yield`.
  - **Exception & resource**: `try`/`catch`/`finally`, `using`, `await using`.
  - **Concurrency**: `lock`.
  - **Misc**: `checked`/`unchecked`, `unsafe`/`fixed`, **top‑level statements**, **local functions**, **empty/labeled**.

---

## 2. Top‑level statements (C# 9)

Cho phép viết chương trình **không cần `Main`** tường minh trong file entry:

```csharp
// Program.cs
using System;

Console.WriteLine("Hello World!");
var name = args.Length > 0 ? args[0] : "guest";
Console.WriteLine($"Hi, {name}");
```

- File top-level có thể `await` trực tiếp.  
- Chỉ nên dùng cho app nhỏ, demo, script; app lớn vẫn nên có `Program.Main` rõ ràng.

---

## 3. Khối lệnh `{ ... }`, phạm vi & lifetime

- Khối `{ ... }` tạo **scope mới** cho **biến cục bộ**.  
- Biến local **không có `readonly`** (trừ `ref readonly`/`in`); `const` là compile-time.

```csharp
int x = 1;
{
    int x2 = x + 1;
    // x2 chỉ tồn tại trong khối
}
// x2 không còn ở đây
```

- **Shadowing**: có thể trùng tên ở scope trong (nên tránh vì giảm rõ ràng).

---

## 4. Declaration statements

### 4.1 Khai báo biến & suy luận kiểu (`var`, target-typed)

```csharp
var n = 42;                 // int
List<string> list = new();  // C# 9 target-typed new
```

### 4.2 `const` & `readonly` (local)

```csharp
const double PI = 3.141592653589793;
```
> `readonly` áp dụng cho **field** (không phải local). Với local, dùng `in`/`ref readonly` để truyền chỉ-đọc.

### 4.3 Deconstruction declaration

```csharp
(var x, var y) = (10, 20);
(int a, int b) = GetPoint();
```

### 4.4 `ref` local & `ref readonly` local

```csharp
int[] arr = {1,2,3};
ref int second = ref arr[1];
second = 99;                 // arr[1] đổi theo

ref readonly int ro = ref arr[0];
// ro = 5; // lỗi: readonly
```

### 4.5 `using` declaration (C# 8+) & `await using`

```csharp
using var stream = File.OpenRead(path); // auto Dispose khi ra khỏi scope
await using var conn = await OpenAsync(); // IAsyncDisposable
```

> Khác `using` statement dạng khối ở mục 9.

### 4.6 `scoped` (C# 11, nâng cao)

Giới hạn lifetime của tham chiếu `ref`/`in` để tránh escape khỏi scope an toàn. Chỉ dùng khi thực sự cần tối ưu `ref`.

---

## 5. Expression statements

Các **biểu thức** có thể trở thành statement: gọi hàm, gán, tăng/giảm, `await`…

```csharp
x = y + 1;
count++;
DoWork();
await Task.Delay(100);
```

> **`throw`** cũng là statement (và từ C# 7 có dạng *throw expression* trong toán tử `?:`/`??`).

---

## 6. Selection statements: `if`/`else`, `switch`

### 6.1 `if` / `else`

```csharp
if (order is null) throw new ArgumentNullException(nameof(order));
if (value is >= 0 and <= 100)
    Console.WriteLine("In range");
else if (value is > 100)
    Console.WriteLine("Too large");
else
    Console.WriteLine("Negative");
```

- Dùng **pattern**: `is null`, `is not null`, `and/or/not`, relational, property/list patterns.

### 6.2 `switch` (statement) — pattern matching hiện đại

```csharp
static string Classify(object? x) => x switch // switch expression (tham khảo)
{
    null            => "null",
    int i when i < 0 => "negative int",
    int             => "int",
    string { Length: 0 } => "empty string",
    string s        => $"string({s.Length})",
    _               => "other"
};
```

`switch` **statement** với pattern:

```csharp
object x = Get();
switch (x)
{
    case null:
        Console.WriteLine("null");
        break;

    case int i when i < 0:
        Console.WriteLine("negative int");
        break;

    case string { Length: > 0 } s:
        Console.WriteLine($"string len={s.Length})");
        break;

    default:
        Console.WriteLine("other");
        break;
}
```

**Lưu ý**:
- `goto case`, `goto default` được hỗ trợ.  
- Không có “rơi qua” (fall‑through) implicit giữa case (phải `goto`).  
- **Switch expression** là *expression*, không phải statement (đặt trong biểu thức trả về/khởi tạo).

---

## 7. Iteration statements: `while`, `do`, `for`, `foreach`, `await foreach`

### 7.1 `while` / `do`

```csharp
while (reader.Read())
{
    // ...
}

do
{
    ReadInput();
} while (!IsValid());
```

### 7.2 `for`

```csharp
for (int i = 0; i < n; i++)
{
    if ((i & 1) == 0) continue; // bỏ qua số chẵn
    sum += i;
}

// Nhiều biến khởi tạo/iterator
for (int i = 0, j = n-1; i < j; i++, j--) { Swap(a, i, j); }
```

### 7.3 `foreach`

```csharp
foreach (var line in File.ReadLines(path))
    Console.WriteLine(line);
```

- Hoạt động trên bất kỳ kiểu có `GetEnumerator()` phù hợp.  
- **`break`/`continue`** vẫn hoạt động như thường.  
- Với `Dictionary<TKey,TValue>`: `foreach (var (k,v) in dict)` deconstruction.

### 7.4 `await foreach` (C# 8) — async streams

```csharp
await foreach (var msg in ReceiveAsync(channel))
    Console.WriteLine(msg);

static async IAsyncEnumerable<string> ReceiveAsync(ChannelReader<string> r)
{
    while (await r.WaitToReadAsync())
        while (r.TryRead(out var m)) yield return m;
}
```

---

## 8. Jump statements: `break`, `continue`, `return`, `throw`, `goto`, `yield`

### 8.1 `break` / `continue`

- `break` thoát khỏi vòng lặp hiện tại hoặc `switch`.  
- `continue` bỏ phần còn lại của vòng lặp & bắt đầu vòng mới.

### 8.2 `return`

```csharp
if (input is null) return;
return value; // với kiểu trả về non-void
```

### 8.3 `throw`

```csharp
throw new InvalidOperationException("Bad state");

// C# hiện đại: throw expression
var x = value ?? throw new ArgumentNullException(nameof(value));
```

### 8.4 `goto` & labeled statement

```csharp
start:
if (!TryStep()) goto start;

switch (kind)
{
    case 0: Handle0(); goto done;
    case 1: Handle1(); goto done;
    default: goto case 0;
}
done:;
```

> Dùng *tiết chế*; đa số trường hợp có thể thay bằng cấu trúc điều khiển rõ ràng hơn.

### 8.5 `yield return` / `yield break` (iterator method)

```csharp
public static IEnumerable<int> Evens(int from, int count)
{
    int n = from;
    while (count-- > 0)
    {
        if ((n & 1) == 0) yield return n;
        n++;
    }
}
```

---

## 9. Exception handling & resource: `try`/`catch`/`finally`, `using`/`await using`

### 9.1 `try` / `catch` / `finally` + filter `when`

```csharp
try
{
    await SaveAsync(); // có thể await trong try/catch
}
catch (HttpRequestException ex) when ((int?)ex.StatusCode == 429)
{
    await Task.Delay(1000);
    // retry nhẹ...
}
catch (Exception ex)
{
    Log(ex);
    throw; // giữ stack
}
finally
{
    Cleanup();
}
```

### 9.2 `using` statement (khối) vs `using` declaration

**Using statement** (khối **nội** scope):

```csharp
using (var stream = File.OpenRead(path))
using (var reader = new StreamReader(stream))
{
    Console.WriteLine(reader.ReadLine());
} // Dispose() được gọi ở đây theo thứ tự ngược
```

**Using declaration** (**C# 8+**, hiệu lực đến hết scope hiện tại):

```csharp
using var stream = File.OpenRead(path);
using var reader = new StreamReader(stream);
// ... dùng reader
// tự Dispose khi rời block hiện tại
```

**`await using`** cho `IAsyncDisposable`:

```csharp
await using var conn = await OpenConnectionAsync();
```

---

## 10. Đồng bộ hoá: `lock`

Đảm bảo **mutual exclusion** cho đoạn critical:

```csharp
private readonly object _sync = new();

void Add(int value)
{
    lock (_sync)
    {
        _total += value;
    }
}
```

- Tương đương `Monitor.Enter/Exit` (an toàn với exception).  
- **Tránh** lock trên `this` hoặc `Type` public (dễ deadlock do bên ngoài cũng lock).  
- Với async: dùng `SemaphoreSlim` thay vì `lock` (vì `lock` không `await` được).

---

## 11. Kiểm soát tràn & môi trường: `checked`/`unchecked`, `unsafe`/`fixed`

```csharp
checked
{
    int c = int.MaxValue + 1; // ném OverflowException
}

unchecked
{
    int d = int.MaxValue + 1; // tràn im lặng
}
```

**Unsafe & fixed** (khi cần interop/hiệu năng thấp‑level):

```csharp
unsafe
{
    int x = 10;
    int* p = &x;
    *p = 20;

    fixed (char* cp = "abc") { /* dùng cp */ }
}
```

> Chỉ bật khi thật sự cần; xem thêm chương *Type System* và *Operators* (pointer ops).

---

## 12. Local functions

Hàm cục bộ nằm bên trong method, **đặt tên được**, có thể `async` hoặc `iterator`:

```csharp
int SumSquares(ReadOnlySpan<int> a)
{
    static int Sqr(int x) => x * x; // static: không capture

    int sum = 0;
    foreach (var x in a) sum += Sqr(x);
    return sum;
}
```

- Khác lambda: local function **ít cấp phát hơn** khi không capture; hỗ trợ `ref/out`/`yield`.

---

## 13. Empty & labeled statements

- **Empty**: chỉ dấu `;` — đôi khi dùng làm **no‑op** (hiếm).  
- **Labeled**: `label:` đứng trước một statement để `goto` tới.

```csharp
; // empty

retry:
if (!TryConnect())
{
    attempts++;
    if (attempts < 3) goto retry;
}
```

---

## 14. Mẹo & best practices

1. **Ưu tiên cấu trúc rõ ràng** thay vì `goto`.  
2. **Luôn `break`** trong `switch` (trừ khi `goto`/`return`), tránh rơi qua.  
3. **Bao try/catch ở rìa hệ thống**; ở sâu bên trong để exception “bubble up” (xem chương *Exceptions*).  
4. **`using` declaration** cho scope dài; **`using` statement** khi cần gói nhóm nhỏ/điểm dispose cụ thể.  
5. Với **async**, dùng `await foreach`/`await using` cho stream/tài nguyên bất đồng bộ.  
6. `lock`: chốt đối tượng **riêng tư**; tránh deadlock; xem xét `SemaphoreSlim` khi cần `await`.  
7. Vòng lặp: cân nhắc **`for`** cho mảng/`Span<T>` (hiệu năng), **`foreach`** cho code đọc dễ.  
8. Tận dụng **pattern matching** trong `if`/`switch` để code ngắn gọn, an toàn null/type.

---

### Phụ lục: Bộ ví dụ “từ đơn giản đến nâng cao”

**Validate & chuyển nhánh bằng pattern**

```csharp
static string Describe(object? x)
{
    if (x is null) return "null";
    if (x is int { } i and >= 0 and <= 100) return $"int[0..100]={i}";
    if (x is string { Length: > 0 } s) return $"string({s.Length})";
    return "other";
}
```

**Pipeline đọc file an toàn tài nguyên**

```csharp
static IEnumerable<string> ReadNonEmptyLines(string path)
{
    using var s = File.OpenRead(path);
    using var r = new StreamReader(s);
    string? line;
    while ((line = r.ReadLine()) is not null)
        if (line.Length > 0) yield return line;
}
```

**Tiêu thụ stream async**

```csharp
await foreach (var evt in SubscribeAsync(topic, ct))
{
    if (evt.Severity >= 3)
        await HandleAsync(evt, ct);
}
```

**Vòng lặp hai con trỏ**

```csharp
for (int i = 0, j = a.Length - 1; i < j; i++, j--)
    (a[i], a[j]) = (a[j], a[i]);
```
