# Phương thức (Method) trong C#

Trong C#, **phương thức (method)** là đơn vị cơ bản để đóng gói logic và định nghĩa hành vi cho một kiểu (`class`, `struct`, `record`, v.v.).  

---

## Mục lục

1. [Khai báo phương thức](#1-khai-báo-phương-thức)
2. [Gọi phương thức](#2-gọi-phương-thức)
3. [Tham số (parameters)](#3-tham-số-parameters)
4. [Truyền theo giá trị và truyền theo tham chiếu](#4-truyền-theo-giá-trị-và-truyền-theo-tham-chiếu)
5. [Safe context](#5-safe-context)
6. [Modifier trên tham số – tổng quan](#6-modifier-trên-tham-số--tổng-quan)
7. [`ref`](#7-ref)
8. [`out`](#8-out)
9. [`ref readonly`](#9-ref-readonly)
10. [`in`](#10-in)
11. [`params`](#11-params)
12. [`this` và extension method](#12-this-và-extension-method)
13. [Iterator method – `yield return` / `yield break`](#13-iterator-method--yield-return--yield-break)
14. [Tài liệu liên quan](#14-tài-liệu-liên-quan)

---

## 1. Khai báo phương thức

### 1.1 Cú pháp tổng quát

```csharp
[attributes]
[modifiers] return_type MethodName(parameter_list)
{
    // Thân phương thức (method body)
}
```

Trong đó:

- `attributes`: các attribute như `[Obsolete]`, `[TestMethod]`, v.v. (có hoặc không).
- `modifiers` (bộ sửa đổi):
  - Access: `public`, `private`, `protected`, `internal`, `protected internal`, `private protected`
  - Hành vi: `static`, `virtual`, `override`, `abstract`, `sealed`, `extern`, `unsafe`, v.v.
- `return_type`: kiểu trả về (`int`, `string`, `void`, `T`, v.v.).
- `MethodName`: tên phương thức (theo convention PascalCase).
- `parameter_list`: danh sách tham số (có thể rỗng).

Ví dụ:

```csharp
public class Calculator
{
    // instance method
    public int Add(int x, int y)
    {
        return x + y;
    }

    // static method
    public static int Multiply(int x, int y)
    {
        return x * y;
    }
}
```

### 1.2 Overload method

Cùng tên, **khác chữ ký** (kiểu/ số lượng/ thứ tự tham số) ⇒ nhiều overload:

```csharp
public class Logger
{
    public void Log(string message) { /* ... */ }

    public void Log(string message, Exception ex) { /* ... */ }

    public void Log(Exception ex) { /* ... */ }
}
```

### 1.3 Expression-bodied method

Dùng khi thân method chỉ là **một expression**:

```csharp
public int Square(int x) => x * x;

public override string ToString()
    => $"{Name} ({Age})";
```

### 1.4 Generic method

Method có tham số kiểu `T`, `TKey`, `TValue`…:

```csharp
public T Max<T>(T a, T b) where T : IComparable<T>
    => a.CompareTo(b) >= 0 ? a : b;

int maxInt = Max(3, 10);         // T => int
string maxStr = Max("a", "z");   // T => string
```

---

## 2. Gọi phương thức

### 2.1 Gọi phương thức instance

Cần **instance** của kiểu:

```csharp
var calc = new Calculator();
int sum = calc.Add(3, 5);
```

### 2.2 Gọi phương thức static

Gọi qua **tên type**:

```csharp
int abs = Math.Abs(-10);
int prod = Calculator.Multiply(2, 3);
```

### 2.3 Named arguments & optional parameters

**Named arguments** – chỉ rõ tên tham số:

```csharp
void SendEmail(string to, string subject, string body = "", bool isHtml = false) { }

SendEmail(
    to: "user@example.com",
    subject: "Hello",
    isHtml: true);
```

**Optional parameters** – tham số có giá trị mặc định:

```csharp
void Log(string message, LogLevel level = LogLevel.Info) { }

Log("Started");                       // level = Info
Log("Critical!", LogLevel.Critical);  // override
```

Lưu ý: giống `const`, giá trị mặc định được embed vào call site (assembly gọi) tại compile-time.

### 2.4 Method chaining / Fluent API

Method trả về chính nó (hoặc object khác) để gọi liên tiếp:

```csharp
var builder = new StringBuilder()
    .Append("Hello ")
    .AppendLine("world");
```

---

## 3. Tham số (parameters)

**Tham số (parameter)** là biến trong khai báo method.  
**Đối số (argument)** là giá trị truyền vào khi gọi method.

```csharp
void Greet(string name, int times) // name, times là parameters
{
    for (int i = 0; i < times; i++)
        Console.WriteLine($"Hello {name}");
}

Greet("Alice", 3); // "Alice", 3 là arguments
```

Các kiểu tham số cơ bản:

- Tham số bình thường (truyền theo giá trị).
- Tham số có modifier: `ref`, `out`, `in`, `params`, `this`.
- Tham số generic (dựa trên `T` của method/type).
- Tham số optional (có default value).

---

## 4. Truyền theo giá trị và truyền theo tham chiếu

### 4.1 Truyền theo giá trị (by value) – mặc định

Khi **không** dùng `ref` / `out` / `in`, C# luôn truyền **theo giá trị**:

- Với **value type**: copy **giá trị**.
- Với **reference type**: copy **reference**, *không phải* copy object.

```csharp
void ChangeInt(int x) { x = 20; }

void ChangeName(Person p)
{
    p.Name = "Bob";       // thay đổi object
    p = new Person();     // chỉ đổi local p
}

int a = 10;
ChangeInt(a);
Console.WriteLine(a); // 10 (không đổi)

var person = new Person { Name = "Alice" };
ChangeName(person);
Console.WriteLine(person.Name); // "Bob"
```

### 4.2 Truyền theo tham chiếu (by reference)

Dùng `ref`, `out`, `in`:

- Method nhận **tham chiếu** tới biến của caller.
- Thay đổi trên tham số ⇒ tác động trực tiếp biến gốc.

```csharp
void Increment(ref int x) => x++;

int n = 10;
Increment(ref n);
Console.WriteLine(n); // 11
```

---

## 5. Safe context

**Safe context** là vùng code:

- Không dùng con trỏ (`*`, `&` trên kiểu pointer, `T*`, `void*`),
- Không thao tác trực tiếp vùng nhớ unmanaged,
- Là chế độ mặc định của C#.

Dù dùng `ref`, `out`, `in`, `ref readonly` thì:

- Vẫn an toàn (CLR đảm bảo type-safe, bounds-check…),
- Không phá vỡ quản lý bộ nhớ của GC.

Chỉ khi bạn dùng từ khóa `unsafe` thì method mới ở **unsafe context**:

```csharp
public unsafe void Foo(int* p) { *p = 42; }

unsafe
{
    int value = 10;
    int* p = &value;
    *p = 20;
}
```

---

## 6. Modifier trên tham số – tổng quan

Các **parameter modifier** quyết định **cách truyền** tham số:

- `ref` – truyền tham chiếu **đọc/ghi**.
- `out` – truyền tham chiếu, dành cho **giá trị đầu ra**.
- `in` – truyền tham chiếu **chỉ-đọc** (readonly by-ref).
- `params` – tham số “danh sách” (varargs).
- `this` – dùng để khai báo **extension method**.

Trong các mục sau, ta sẽ đi chi tiết từng modifier quan trọng.

---

## 7. `ref`

### 7.1 `ref` parameter

- Biến gọi **phải được gán giá trị trước** khi truyền vào.
- Tham số trong method có thể **đọc & gán** – ảnh hưởng trực tiếp biến gốc.

```csharp
void Swap(ref int a, ref int b)
{
    int t = a;
    a = b;
    b = t;
}

int x = 1, y = 2;
Swap(ref x, ref y);

Console.WriteLine(x); // 2
Console.WriteLine(y); // 1
```

Lưu ý:

- Cần ghi `ref` ở **cả khai báo và chỗ gọi**.
- Không dùng `ref` với `const` hoặc biểu thức không phải lvalue.

### 7.2 `ref` local và `ref` return

**Ref local**: alias tới một vị trí dữ liệu.

```csharp
int[] numbers = { 1, 2, 3 };

ref int second = ref numbers[1];
second = 42;

Console.WriteLine(numbers[1]); // 42
```

**Ref return**: method trả về **tham chiếu**, không phải copy:

```csharp
ref int FindFirstPositive(int[] arr)
{
    for (int i = 0; i < arr.Length; i++)
        if (arr[i] > 0)
            return ref arr[i];

    throw new InvalidOperationException();
}

int[] data = { -1, -2, 5, 3 };
ref int v = ref FindFirstPositive(data);
v = 10;

Console.WriteLine(data[2]); // 10
```

Hạn chế:

- Không được trả `ref` tới biến local (vì sẽ bị thu hồi khỏi stack).
- Chủ yếu dùng với mảng, `Span<T>`, struct field.

---

## 8. `out`

`out` dành cho **tham số đầu ra**, thường dùng với pattern `TryXxx`.

Đặc điểm:

- Biến truyền vào có thể **chưa gán** giá trị.
- Method **bắt buộc** gán giá trị cho `out` trước khi `return`.
- Sau khi method kết thúc, biến `out` sẽ mang giá trị mới.

```csharp
bool TryParseInt(string text, out int value)
{
    return int.TryParse(text, out value);
}

// C# 7: declare-out
if (TryParseInt("123", out int number))
{
    Console.WriteLine(number);
}
else
{
    Console.WriteLine("Không phải số");
}
```

Có thể dùng discard `_` nếu không cần giá trị:

```csharp
if (!int.TryParse(input, out _))
{
    Console.WriteLine("Invalid number");
}
```

Pattern `TryXxx` (vd `int.TryParse`, `Dictionary.TryGetValue`) rất phổ biến và nên được áp dụng cho API của bạn khi cần.

---

## 9. `ref readonly`

`ref readonly` là **tham chiếu chỉ-đọc**:

- Dùng cho **ref local** và **ref return**.
- Giúp tránh copy struct lớn nhưng vẫn không cho phép sửa dữ liệu.

Ví dụ trả `ref readonly`:

```csharp
public readonly struct BigStruct
{
    public int A { get; }
    public int B { get; }
}

private readonly BigStruct[] _items;

public ref readonly BigStruct GetItem(int index)
{
    return ref _items[index]; // ref readonly return
}
```

Dùng:

```csharp
ref readonly BigStruct item = ref GetItem(0);
// item = new BigStruct(); // lỗi
Console.WriteLine(item.A);
```

So sánh:

- `ref`          → tham chiếu **đọc/ghi**.
- `ref readonly` → tham chiếu **chỉ-đọc**.
- `in`           → dành cho **tham số** (by-ref readonly parameter).

---

## 10. `in`

`in` (C# 7.2) cho phép:

- Truyền **bằng tham chiếu**,  
- Nhưng **chỉ-đọc** bên trong method.

Phù hợp cho **struct lớn**, truyền nhiều mà không muốn copy.

```csharp
public readonly struct Matrix4x4
{
    // 16 phần tử, khá to để copy
}

public float Determinant(in Matrix4x4 m)
{
    // m là tham chiếu chỉ-đọc
    // m = default; // lỗi
    return 0; // ví dụ
}
```

Gọi method:

```csharp
Matrix4x4 mat = /* ... */;
float det = Determinant(in mat); // rõ ràng
float det2 = Determinant(mat);   // compiler có thể chèn 'in' ngầm
```

Lưu ý:

- Compiler có thể copy trong một số trường hợp để đảm bảo an toàn.
- Dùng `in` chủ yếu vì lý do hiệu năng (struct lớn), không nên lạm dụng khi struct nhỏ.

---

## 11. `params`

`params` cho phép truyền **0, 1 hoặc nhiều đối số** vào một tham số dạng mảng.

Khai báo:

```csharp
void Log(params string[] messages)
{
    foreach (var message in messages)
        Console.WriteLine(message);
}
```

Gọi:

```csharp
Log("A");
Log("A", "B", "C");
Log(); // messages.Length == 0

string[] arr = { "X", "Y" };
Log(arr); // vẫn hợp lệ
```

Quy tắc:

- `params` **phải là tham số cuối cùng** trong danh sách.
- Mỗi method chỉ có **tối đa một** tham số `params`.
- Ở runtime, đây đơn giản là một mảng (`string[]`).

Cẩn thận:

- Mỗi lần gọi có thể tạo mảng mới (nếu không truyền sẵn mảng) → chú ý trong code nhạy hiệu năng.
- Overload với và không `params` có thể gây nhầm lẫn; compiler chọn overload theo luật riêng.

---

## 12. `this` và extension method

### 12.1 `this` trong tham số đầu tiên

`this` trong tham số đầu tiên của một **static method** cho phép khai báo **extension method** – cách “gắn thêm” method cho type đã tồn tại mà không sửa code type đó.

Quy tắc:

- Method phải trong **static class**.
- Method phải **static**.
- Tham số đầu tiên: `this SomeType value`.

```csharp
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string? value)
        => string.IsNullOrEmpty(value);

    public static string ToSlug(this string value)
    {
        return value
            .Trim()
            .ToLowerInvariant()
            .Replace(" ", "-");
    }
}
```

### 12.2 Gọi extension method

Sau khi `using` đúng namespace, có thể gọi như instance method:

```csharp
using MyApp.Extensions;

string? name = null;
if (name.IsNullOrEmpty())
{
    Console.WriteLine("No name");
}

string title = "Hello World C#";
string slug = title.ToSlug(); // "hello-world-c#"
```

Thực chất, compiler dịch thành:

```csharp
StringExtensions.IsNullOrEmpty(name);
StringExtensions.ToSlug(title);
```

### 12.3 Lưu ý khi thiết kế extension method

- Không phá vỡ tính bất biến của type (đặc biệt `string` immutable).
- Tránh behavior bất ngờ, khó đoán.
- Quan tâm tới namespace để tránh xung đột tên và overload không mong muốn.

---

## 13. Iterator method – `yield return` / `yield break`

### 13.1 Iterator method là gì?

**Iterator method** là phương thức:

- Trả về **một sequence** (`IEnumerable`, `IEnumerable<T>`, `IEnumerator`, `IEnumerator<T>`),
- Dùng từ khóa **`yield return`** để trả từng phần tử *từng bước một*,
- Có thể dùng **`yield break`** để kết thúc sequence sớm.

Nhờ đó bạn không phải tự cài đặt `IEnumerator<T>` một cách thủ công.

Ví dụ:

```csharp
public IEnumerable<int> EvenNumbers(int from, int count)
{
    int current = from;

    for (int i = 0; i < count; i++)
    {
        if (current % 2 == 0)
            yield return current;

        current++;
    }
}
```

Dùng:

```csharp
foreach (var n in EvenNumbers(1, 10))
{
    Console.WriteLine(n);
}
```

### 13.2 `IEnumerable<T>` và `IEnumerator<T>` – nền tảng của iterator

`foreach` hoạt động dựa trên `IEnumerable<T>` và `IEnumerator<T>`:

```csharp
public interface IEnumerable<out T>
{
    IEnumerator<T> GetEnumerator();
}

public interface IEnumerator<out T> : IDisposable
{
    T Current { get; }
    bool MoveNext();
    void Reset();
}
```

Khi `foreach (var item in sequence)`:

- Compiler gọi `GetEnumerator()`,
- Lặp: `MoveNext()` → `Current`,
- Kết thúc: `Dispose()`.

Iterator method với `yield` giúp bạn không phải viết bộ cài đặt này.

### 13.3 `yield return` – trả từng phần tử

```csharp
public IEnumerable<int> Range(int start, int count)
{
    for (int i = 0; i < count; i++)
    {
        yield return start + i;
    }
}
```

Sequence kết quả (ví dụ `Range(10, 5)`): `10, 11, 12, 13, 14`.

Đặc điểm:

- **Deferred execution**: logic chỉ thực thi khi bắt đầu duyệt sequence, không phải lúc gọi method.
- Mỗi lần `MoveNext()`: chạy tới `yield return` kế tiếp, lưu state, chờ lần gọi tiếp theo.

### 13.4 `yield break` – kết thúc sớm

```csharp
public IEnumerable<int> UpTo(int max)
{
    for (int i = 0; ; i++)
    {
        if (i > max)
            yield break;

        yield return i;
    }
}
```

Hoặc khi không có dữ liệu:

```csharp
public IEnumerable<string> FindNonEmpty(IEnumerable<string?> source)
{
    if (source is null)
        yield break;

    foreach (var s in source)
    {
        if (!string.IsNullOrWhiteSpace(s))
            yield return s!;
    }
}
```

### 13.5 Iterator & state machine (ý tưởng)

Compiler biến một method như:

```csharp
public IEnumerable<int> Simple()
{
    yield return 1;
    yield return 2;
}
```

thành một class implement `IEnumerable<int>` + `IEnumerator<int>` với trường `_state` và `switch` trong `MoveNext()`.

Bạn chỉ cần nhớ:

- Viết code tuần tự với `yield`,
- Compiler lo việc sinh state machine giúp bạn.

### 13.6 Deferred execution & nhiều lần duyệt

```csharp
var seq = Range(0, 3);

Console.WriteLine("Before foreach");

foreach (var x in seq)
{
    Console.WriteLine(x);
}

Console.WriteLine("After foreach");
```

- `Range` chưa chạy cho tới khi bắt đầu `foreach`.
- Mỗi `foreach` tạo enumerator mới → logic chạy lại.

Nếu muốn **chỉ tính một lần**, hãy materialize:

```csharp
var list = Range(0, 3).ToList(); // thực thi ngay
```

### 13.7 Resource & `try/finally` trong iterator

Iterator method vẫn hỗ trợ `using` / `try/finally`:

```csharp
public IEnumerable<string> ReadLines(string path)
{
    using var stream = File.OpenRead(path);
    using var reader = new StreamReader(stream);

    string? line;
    while ((line = reader.ReadLine()) != null)
    {
        yield return line;
    }
}
```

Khi `foreach` kết thúc (dù bình thường hay exception), enumerator được dispose và `using` đảm bảo resource được giải phóng.

---

## 14. Tài liệu liên quan

- **Chương bất đồng bộ**: [async.md](./async.md) — *async/await, Task/ValueTask, exception/cancellation trong async, và async streams (`IAsyncEnumerable<T>`, `await foreach`).*
