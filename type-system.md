# Hệ thống kiểu dữ liệu trong C#

Trong C#, **mọi thứ đều là kiểu (type)** – từ số nguyên, chuỗi, class, struct, đến `void`.  
Hiểu hệ thống kiểu là nền tảng để nắm vững C# và .NET.

Nội dung chương này:

- Value type
  - Nullable value type
  - Boxing / Unboxing
- Reference type
- Các kiểu dữ liệu có sẵn (built-in types)
- Kiểu `object`
- Kiểu `string`
- Kiểu `dynamic`
- Kiểu `void`
- `enum`
- `struct`
- `ref struct`
- `null`
- Unmanaged types
- Khai báo biến
  - Từ khóa `var`
  - Giá trị mặc nhiên (default value)
- Kiểu generic
  - Ràng buộc `new()`
  - Ràng buộc `where`
- Namespace
  - Khai báo namespace
  - `using`
  - `extern alias`
- Ép kiểu (casting)
- Sơ đồ tổng quan hệ thống kiểu dữ liệu

---

## 1. Value type

**Value type** là các kiểu mà **giá trị được chứa trực tiếp** trong biến.  
Khi gán hoặc truyền tham số, giá trị được **copy** (trừ khi dùng `ref`, `in`, `out`).

Ví dụ: `int`, `double`, `bool`, `char`, `struct`, `enum`, `DateTime`…

Đặc điểm:

- Thường nhỏ, “nhẹ”, lưu trữ dữ liệu đơn giản.
- Mặc định **không thể null** (trừ khi dùng dạng nullable).
- Khi gán biến này cho biến khác → tạo **bản sao giá trị**, không dùng chung vùng nhớ.

```csharp
int a = 10;
int b = a;
b = 20;

Console.WriteLine(a); // 10
Console.WriteLine(b); // 20
```

---

### 1.1 Nullable value type (các kiểu dữ liệu nullable)

Mặc định, value type **không thể mang giá trị null**. C# cung cấp dạng `T?` (ví dụ `int?`, `bool?`) để cho phép:

- `T?` là shorthand của `Nullable<T>` với `T` là value type.
- Giá trị có thể là **một giá trị hợp lệ của T** hoặc **null**.

```csharp
int? age = null;

if (age.HasValue)
{
    Console.WriteLine(age.Value);
}
else
{
    Console.WriteLine("Chưa có tuổi");
}

// Null-coalescing
int ageValue = age ?? 0;
```

**Null-lifting**: Các toán tử trên nullable hoạt động theo logic:

- Nếu bất kỳ toán hạng nào là `null` → kết quả `null`.

```csharp
int? a = 5;
int? b = null;

int? sum = a + b; // null
```

---

### 1.2 Boxing và Unboxing

**Boxing**: chuyển một **value type → reference type** (thường là `object` hoặc interface). CLR sẽ:

- Cấp phát một object trên heap.
- Copy giá trị value type vào object đó.

```csharp
int x = 42;
object o = x; // boxing
```

**Unboxing**: lấy giá trị value type từ object đã boxing (cast).

```csharp
object o = 42;
int x = (int)o; // unboxing
```

Lưu ý:

- Unboxing sai kiểu sẽ ném `InvalidCastException`.
- Boxing/unboxing tốn chi phí (cấp phát + copy) → tránh trong code hiệu năng cao.

Ví dụ dễ bị boxing:

```csharp
int x = 42;
object o1 = x;          // boxing
IComparable c = x;      // boxing (x vào interface)
Console.WriteLine(o1);  // boxing nếu truyền value type trực tiếp cho object param
```

---

## 2. Reference type

**Reference type** là các kiểu mà biến **chứa tham chiếu (reference)** tới object nằm trên heap.

Ví dụ: `class`, `string`, `array`, `delegate`, `interface`…

Đặc điểm:

- Biến có thể chứa **`null`** (trừ khi bật nullable reference types và chú thích khác).
- Khi gán biến này cho biến khác → **cùng trỏ tới một object** (chia sẻ dữ liệu).

```csharp
class Person
{
    public string Name { get; set; } = "";
}

var p1 = new Person { Name = "Alice" };
var p2 = p1;
p2.Name = "Bob";

Console.WriteLine(p1.Name); // "Bob" - cùng tham chiếu
```

- Vòng đời object do **Garbage Collector (GC)** quản lý.

---

## 3. Các kiểu dữ liệu có sẵn (built-in types)

C# cung cấp các **alias** cho các kiểu .NET:

| C#       | .NET type          | Ghi chú                      |
|----------|--------------------|-----------------------------|
| `bool`   | `System.Boolean`   | true/false                  |
| `byte`   | `System.Byte`      | 0..255                      |
| `sbyte`  | `System.SByte`     | -128..127                   |
| `short`  | `System.Int16`     | 16-bit có dấu               |
| `ushort` | `System.UInt16`    | 16-bit không dấu            |
| `int`    | `System.Int32`     | 32-bit có dấu               |
| `uint`   | `System.UInt32`    | 32-bit không dấu            |
| `long`   | `System.Int64`     | 64-bit có dấu               |
| `ulong`  | `System.UInt64`    | 64-bit không dấu            |
| `float`  | `System.Single`    | số thực 32-bit              |
| `double` | `System.Double`    | số thực 64-bit              |
| `decimal`| `System.Decimal`   | số thập phân (tài chính)    |
| `char`   | `System.Char`      | ký tự Unicode 16-bit        |
| `string` | `System.String`    | chuỗi Unicode               |
| `object` | `System.Object`    | gốc mọi reference type      |

Tất cả chúng đều là **type bình thường trong .NET** – alias chỉ để code C# ngắn hơn.

---

## 4. Kiểu `object`

`object` (alias của `System.Object`) là:

- **Gốc (root)** của mọi reference type.
- Mọi type trong .NET (kể cả value type) “kế thừa” từ `object`  
  (value type thì boxing mới thực sự trở thành object).

Các method quan trọng:

- `ToString()` – chuỗi biểu diễn của object.
- `Equals(object? obj)` – so sánh equality.
- `GetHashCode()` – mã băm (dùng trong dictionary).
- `GetType()` – lấy type runtime (`System.Type`).

```csharp
object o = 42;
Console.WriteLine(o.GetType().Name); // Int32
Console.WriteLine(o.ToString());     // "42"
```

Khi thiết kế type:

- Nên override `ToString()` để debug/log dễ hơn.
- Nên override `Equals` + `GetHashCode` nếu type có equality theo giá trị.

---

## 5. Kiểu `string`

`string` là **chuỗi Unicode immutable**:

- Mọi thao tác “sửa” chuỗi thực chất là tạo **chuỗi mới**.
- Là reference type đặc biệt:
  - Có **string interning**.
  - Hỗ trợ **operator `+`**, **interpolation `$""`**, v.v.

```csharp
string s1 = "Hello";
string s2 = s1;
s1 += " world";

Console.WriteLine(s1); // "Hello world"
Console.WriteLine(s2); // "Hello"
```

Vì immutable nên:

- Nối chuỗi nhiều lần trong loop → nên dùng `StringBuilder` để tối ưu.

```csharp
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i).Append(", ");
}
string result = sb.ToString();
```

---

## 6. Kiểu `dynamic`

`dynamic` được thêm ở C# 4.0:

- Biến `dynamic` **được kiểm tra kiểu tại runtime**, không phải compile-time.
- Gần giống `object`, nhưng mọi gọi member được defer tới runtime (DLR – Dynamic Language Runtime).

```csharp
dynamic d = "hello";
Console.WriteLine(d.ToUpper()); // OK

d = 42;
Console.WriteLine(d + 1);       // 43

// Nếu gọi phương thức không tồn tại → lỗi runtime, không phải compile-time.
```

**Dùng khi:**

- Làm việc với **COM**, **Interop**, các API dynamic, JSON động, v.v.
- Nhưng: **rất dễ lỗi runtime** & chậm hơn do binding động → hạn chế dùng trong core logic.

---

## 7. Kiểu `void`

`void` biểu diễn **“không có kiểu / không trả về gì”**.

- Dùng làm kiểu trả về của method:

```csharp
void Log(string message) => Console.WriteLine(message);
```

- Không thể tạo biến kiểu `void`.
- Xuất hiện trong con trỏ hàm (function pointer) / delegate: `Action` tương đương `void` với tham số.

---

## 8. `enum`

`enum` là **value type** dùng để biểu diễn tập giá trị liệt kê.

- Underlying type mặc định: `int`, có thể đổi: `: byte`, `: short`…

```csharp
public enum OrderStatus : byte
{
    Pending = 0,
    Processing = 1,
    Completed = 2,
    Cancelled = 3
}
```

**Flags enum**:

```csharp
[Flags]
public enum FileAccess
{
    Read  = 1,
    Write = 2,
    ReadWrite = Read | Write
}
```

Enum vẫn là số ⇒ có thể cast giá trị “lạ”:

```csharp
OrderStatus s = (OrderStatus)123; // compile OK, nhưng giá trị "bẩn"
```

Khi nhận từ ngoài vào, nên validate.

---

## 9. `struct`

`struct` là **value type do người dùng định nghĩa**.

- Thường dùng cho type nhỏ, biểu diễn một “value” (Point, Range, Money…).
- Từ C# hiện đại có thể dùng `readonly struct`, `record struct`.

```csharp
public readonly struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);
}
```

Đặc điểm:

- Copy theo giá trị.
- Không có inheritance giữa struct; nhưng có thể implement interface.
- Nên nhỏ (vài field) và **immutable** để tránh lỗi.

---

## 10. `ref struct`

`ref struct` (C# 7.2) là **struct “stack-only”**:

- Không thể:

  - Boxing.
  - Là field của class/struct bình thường.
  - Dùng trong lambda/async (vì có thể sống lâu hơn stack).
  - Dùng làm type của `List<T>`…

- Điển hình: `Span<T>`, `ReadOnlySpan<T>`.

```csharp
public ref struct BufferSlice
{
    private Span<byte> _buffer;

    public BufferSlice(Span<byte> buffer) => _buffer = buffer;
}
```

Dùng khi cần thao tác trên dữ liệu vùng stack / unmanaged một cách an toàn, hiệu năng cao.

---

## 11. `null`

`null` là **giá trị đặc biệt** cho **reference type** & **nullable value type**:

```csharp
string? name = null;
int? age = null;
```

Từ C# 8.0:

- Có **nullable reference types**: `string?` vs `string`.
- Compiler giúp cảnh báo khả năng bị `NullReferenceException`.

Toán tử hữu ích:

- `?.` – null-conditional.
- `??` – null-coalescing.
- `??=` – null-coalescing assignment.
- `!` – null-forgiving.

```csharp
int? length = name?.Length;
string display = name ?? "Unknown";
```

---

## 12. Unmanaged types (các kiểu không được quản lý)

**Unmanaged type** là type **không chứa bất kỳ reference nào** bên trong:

- Tất cả primitive value type (`int`, `double`, v.v.).
- `enum`.
- Con trỏ (`T*` trong `unsafe`).
- `struct` mà **tất cả field** của nó cũng là unmanaged.

Lý do quan trọng:

- Dùng cho **interop** với native code, P/Invoke, memory layout xác định.
- Dùng cho generic constraint `where T : unmanaged`.

```csharp
public static unsafe void Fill<T>(T* destination, T value, int count)
    where T : unmanaged
{
    for (int i = 0; i < count; i++)
        destination[i] = value;
}
```

Unmanaged type **không thể chứa reference** (vd: `string`, `object`, `class`).

---

## 13. Khai báo biến

### 13.1 Khai báo với kiểu cụ thể

```csharp
int count = 10;
string name = "Alice";
Person customer = new Person();
```

- Biến local phải **được gán giá trị** trước khi sử dụng.
- Field của class/struct có **giá trị mặc định** nếu không gán.

---

### 13.2 Từ khóa `var`

`var` là **implicit typing** cho biến local:

- **Kiểu vẫn được xác định tại compile-time**, không phải dynamic.
- Phải được gán giá trị ngay khi khai báo để compiler suy ra kiểu.

```csharp
var x = 10;        // int
var list = new List<string>(); // List<string>
var person = new Person();     // Person
```

Không được:

```csharp
var x;        // lỗi: không suy được kiểu
var y = null; // lỗi: null không có kiểu
```

**Khi nên dùng `var`:**

- Khi kiểu bên phải **rõ ràng**: `var stream = File.OpenRead(...)`.
- Khi dùng LINQ: kiểu rất dài, khó viết tay.

Khi không nên:

- Khi mất ý nghĩa đọc code: `var x = Get();` (mà `Get` trả kiểu gì không rõ).

---

### 13.3 Giá trị mặc nhiên (default value)

- Field của class/struct, phần tử mảng, biến trong `new T()` luôn có **giá trị mặc định**:
  - `0`, `false`, `' '`, `null`, struct với field default.
- Biến local thì **bắt buộc phải gán** trước khi dùng.

`default(T)` / `default`:

```csharp
int x = default;         // 0
bool b = default;        // false
Person? p = default;     // null
Point p2 = default;      // struct với field 0
```

---

## 14. Kiểu generic

Generics cho phép **định nghĩa type/method với tham số kiểu**:

```csharp
public class Repository<T>
{
    public T? FindById(int id) => default;
}
```

Lợi ích:

- **Type-safe**: không cần cast từ/to `object`.
- Giảm boxing/unboxing.
- Tái sử dụng logic cho nhiều kiểu khác nhau.

---

### 14.1 Ràng buộc `new()` (constructor constraint)

`where T : new()` yêu cầu `T` phải có **public parameterless constructor**.

```csharp
public T CreateInstance<T>() where T : new()
{
    return new T();
}
```

Không dùng được nếu:

- Type không có ctor mặc định.
- Ctor mặc định là `private` hoặc `internal` không truy cập được.

---

### 14.2 Các ràng buộc `where` khác

Một số ràng buộc thường gặp:

- `where T : struct` – T là value type (nullable không được).
- `where T : class` – T là reference type.
- `where T : notnull` – T không được phép nhận null (C# 8+).
- `where T : unmanaged` – T là unmanaged type.
- `where T : SomeBaseClass` – T kế thừa `SomeBaseClass`.
- `where T : ISomeInterface` – T implement interface.
- `where T : U` – T kế thừa/implement U (type parameter khác).

Ví dụ:

```csharp
public interface IRepository<T> where T : Entity
{
    T? FindById(int id);
}

public class Pool<T> where T : unmanaged
{
    // quản lý vùng nhớ native cho T
}
```

---

## 15. Namespace

Namespace dùng để **tổ chức type theo “không gian tên”**.

### 15.1 Khai báo namespace

Cú pháp khối:

```csharp
namespace MyApp.Core.Services
{
    public class OrderService { }
}
```

Từ C# 10: **file-scoped namespace**:

```csharp
namespace MyApp.Core.Services;

public class OrderService { }
```

- Toàn bộ file nằm trong namespace đó, giảm một cấp indent.

---

### 15.2 `using`

`using` cho **hai mục đích khác nhau**:

1. **Import namespace** (compiler biết tìm type theo tên ngắn):

   ```csharp
   using System;
   using System.Collections.Generic;

   List<int> nums = new();
   ```

2. **Quản lý lifetime tài nguyên** (using statement):

   ```csharp
   using var stream = File.OpenRead("data.txt");
   // dùng stream...
   ```

Từ C# 10 có **global using**:

```csharp
global using System;
global using System.Collections.Generic;
```

– áp dụng cho mọi file trong project.

---

### 15.3 `extern alias`

Dùng khi bạn tham chiếu **2 assembly có cùng namespace/type**, cần phân biệt:

```csharp
extern alias LibA;
extern alias LibB;

using A = LibA::MyCompany.Library;
using B = LibB::MyCompany.Library;

class Program
{
    static void Main()
    {
        A::Service s1 = new A::Service();
        B::Service s2 = new B::Service();
    }
}
```

- `extern alias` được cấu hình từ project (alias cho assembly reference).

---

## 16. Ép kiểu (Casting)

C# có nhiều dạng ép kiểu khác nhau.

### 16.1 Ép kiểu tường minh (cast operator)

Dùng `(T)expr`:

```csharp
double d = 3.14;
int i = (int)d; // 3 - mất phần thập phân
```

- Có thể ném `InvalidCastException` nếu cast không hợp lệ (đặc biệt với reference type).

---

### 16.2 Ép kiểu ngầm định (implicit conversion)

Nếu an toàn, compiler cho phép cast ngầm:

```csharp
int i = 10;
double d = i; // implicit: int -> double
```

Các conversion user-defined với `implicit`/`explicit operator` cũng tham gia hệ thống này.

---

### 16.3 `as` / `is` / pattern matching

- `as`: ép kiểu reference/nullable, thất bại → `null`.

  ```csharp
  Stream? s = obj as Stream;
  ```

- `is`: từ C# 7, hỗ trợ pattern matching:

  ```csharp
  if (obj is string text && text.Length > 0)
  {
      Console.WriteLine(text);
  }
  ```

- Kết hợp với pattern `is T t`, `is null`, `is > 0`, `is Type { Prop: ... }`.

---

### 16.4 Kiểm soát overflow với numeric cast

- `checked` / `unchecked`:

```csharp
int x = int.MaxValue;
checked
{
    short s = (short)x; // OverflowException
}

unchecked
{
    short s = (short)x; // wrap, không exception
}
```

---

## 17. Sơ đồ tổng quan hệ thống kiểu dữ liệu

```text
System.Object (object)
│
├─ Value types (kiểu giá trị)
│  ├─ Primitive: bool, int, double, decimal, char, ...
│  ├─ enum
│  ├─ struct
│  │   ├─ user-defined struct
│  │   ├─ ref struct (Span<T>, ReadOnlySpan<T>, ...)
│  │   └─ record struct
│  └─ Nullable<T> (T?)  // chỉ áp dụng cho value type
│
└─ Reference types (kiểu tham chiếu)
   ├─ class
   │   ├─ class bình thường
   │   ├─ record (record class)
   │   └─ object (gốc của mọi reference type)
   ├─ interface
   ├─ delegate
   ├─ array (T[], T[,], ...)
   ├─ string (System.String – immutable)
   └─ dynamic* (xử lý kiểu ở runtime, dựa trên object)
```

Ghi chú:
- Primitive value types (Prim.): bool, byte, sbyte, short, ushort, int, uint, long, ulong,
  char, float, double, decimal.
- Nullable<T> (T?): chỉ áp dụng cho value type → tạo ra “value type có thể null”.
- struct: tất cả struct (bao gồm enum, user-defined struct, ref struct, record struct).
- ref struct: struct “stack-only”, không được boxing, không dùng làm field trong class…
- class: bao gồm các class bình thường, record class, v.v.
- dynamic: về thực chất là System.Object + thông tin động; được xử lý đặc biệt bởi runtime.
- string: reference type bất biến (immutable) nhưng được hỗ trợ đặc biệt bởi runtime + ngôn ngữ.
- array: mọi mảng (T[], T[,], …) đều là reference type thừa kế từ System.Array.
```

---
