# Hệ thống kiểu dữ liệu trong C# (Common Type Sytem)

## Mục lục

- [Hệ thống kiểu dữ liệu trong C# (Common Type Sytem)](#hệ-thống-kiểu-dữ-liệu-trong-c-common-type-sytem)
  - [Mục lục](#mục-lục)
  - [1. Tổng quan CTS/CLS \& Runtime](#1-tổng-quan-ctscls--runtime)
  - [2. Bức tranh bộ nhớ: Stack/Managed Heap \& GC](#2-bức-tranh-bộ-nhớ-stackmanaged-heap--gc)
  - [3. Phân loại kiểu dữ liệu](#3-phân-loại-kiểu-dữ-liệu)
    - [3.1 Value types](#31-value-types)
      - [Nullable Value Types](#nullable-value-types)
      - [Boxing \& Unboxing](#boxing--unboxing)
    - [3.2 Reference types](#32-reference-types)
    - [3.3 Built-in \& Primitive](#33-built-in--primitive)
  - [4. Kiểu đặc biệt: `object`, `string`, `dynamic`, `void`, `null`](#4-kiểu-đặc-biệt-object-string-dynamic-void-null)
  - [5. Struct, `readonly struct`, `ref struct` (byref-like)](#5-struct-readonly-struct-ref-struct-byref-like)
  - [6. Enum \& `[Flags]`](#6-enum--flags)
  - [7. Tuples \& `ValueTuple`, deconstruction](#7-tuples--valuetuple-deconstruction)
  - [8. Records (record class / record struct)](#8-records-record-class--record-struct)
  - [9. Mảng (arrays): 1D, nhiều chiều, jagged, `Span<T>` (overview)](#9-mảng-arrays-1d-nhiều-chiều-jagged-spant-overview)
  - [10. Nullable Reference Types (NRT)](#10-nullable-reference-types-nrt)
  - [11. Khai báo biến \& suy luận kiểu (`var`, target-typed, default literal)](#11-khai-báo-biến--suy-luận-kiểu-var-target-typed-default-literal)
  - [12. Giá trị mặc định (default values)](#12-giá-trị-mặc-định-default-values)
  - [13. Generics \& ràng buộc (`where`, `new()`, `struct/class/unmanaged/notnull`), phương sai (variance)](#13-generics--ràng-buộc-where-new-structclassunmanagednotnull-phương-sai-variance)
    - [13.1 Ràng buộc (`where`)](#131-ràng-buộc-where)
    - [13.2 Phương sai (variance)](#132-phương-sai-variance)
  - [14. Namespace, `using`, `global using`, `extern alias`](#14-namespace-using-global-using-extern-alias)
  - [15. Chuyển đổi \& ép kiểu: implicit/explicit, user-defined, pattern matching](#15-chuyển-đổi--ép-kiểu-implicitexplicit-user-defined-pattern-matching)
    - [15.1 Chuyển đổi chuẩn](#151-chuyển-đổi-chuẩn)
    - [15.2 User-defined conversion](#152-user-defined-conversion)
    - [15.3 Pattern matching](#153-pattern-matching)
  - [16. Unsafe \& unmanaged types (overview), function pointers](#16-unsafe--unmanaged-types-overview-function-pointers)
  - [17. Sơ đồ “type tree” (ASCII)](#17-sơ-đồ-type-tree-ascii)

---

## 1. Tổng quan CTS/CLS & Runtime

- **CTS (Common Type System)**: chuẩn định nghĩa tất cả kiểu trong .NET (giá trị/tham chiếu, kế thừa, generic…).  
- **CLS (Common Language Specification)**: tập con quy tắc để ngôn ngữ khác nhau tương tác (interop) thuận lợi.  
- **CLR**: runtime thực thi IL, quản lý **GC**, **JIT**, kiểm tra an toàn kiểu (type-safety).  
- **C#** biên dịch → **IL** (MSIL/CIL), chứa **metadata** (thông tin type, thuộc tính, method…).

Hiểu CTS/CLS giúp bạn thiết kế API tương thích, đặc biệt khi publish library.

---

## 2. Bức tranh bộ nhớ: Stack/Managed Heap & GC

- **Stack**: lưu biến local/value type (có thể “nằm trong” stack frame), tham chiếu tới object, con trỏ return… vòng đời theo scope call.  
- **Managed Heap**: nơi **object/reference type** (và **boxed value**) sống; GC thu hồi khi không còn tham chiếu.  
- **Value type trong object**: tồn tại trong heap **bên trong** object chứa (ví dụ field của class).  
- **Large Object Heap (LOH)** cho object lớn (≈ ≥ 85KB).  
- **Generations** (Gen 0/1/2): tối ưu chi phí thu gom.

> Không phải “value type = nằm trên stack” luôn đúng; vị trí phụ thuộc **ngữ cảnh lưu trữ**.

---

## 3. Phân loại kiểu dữ liệu

### 3.1 Value types

- Thừa kế **ngầm** từ `System.ValueType` (cuối vẫn từ `object`) nhưng **không** hỗ trợ kế thừa tuỳ ý.  
- Gồm: các số nguyên/thực/decimal/bool/char, `enum`, `struct`, `DateTime`, `Guid`, `ValueTuple`,…  
- **Copy-by-value** khi gán/tham số (nếu không dùng `ref/in/out`).  
- **Hiệu năng**: tốt cho dữ liệu nhỏ, bất biến; nhưng copy struct lớn tốn chi phí → cân nhắc `in`, `ref readonly`.

#### Nullable Value Types

- `T?` với `T` là value type ⇒ `Nullable<T>`. Ví dụ: `int?`, `DateTime?`.  
- Thuộc tính: `.HasValue`, `.Value`, hoặc dùng `??`, `??=`, pattern matching `is null`.  
- Toán tử nâng (lifted operators) hoạt động với nullable (`int? a + int? b`).

#### Boxing & Unboxing

- **Boxing**: value → object (heap).  
- **Unboxing**: object → value (copy từ box).  
- Tốn cấp phát/copy → tránh trong hot-path; lưu ý khi dùng `ArrayList`, `object`, “params object[]”.

```csharp
int x = 42;
object o = x;         // boxing
int y = (int)o;       // unboxing (InvalidCastException nếu sai kiểu)
```

### 3.2 Reference types

- Gồm: `class`, `interface`, `delegate`, `string`, **array**, `record class`.  
- **Copy-by-reference**: gán truyền địa chỉ object; 2 biến trỏ cùng object.  
- Quản lý bởi GC; có **nullable reference types (NRT)** ở mức ngôn ngữ để an toàn null (phần 10).

### 3.3 Built-in & Primitive

- **Numeric**: sbyte/byte, short/ushort, int/uint, long/ulong, nint/nuint, float, double, decimal, Half (System.Half).  
- **Others**: bool, char, string, object.  
- **BigInteger** (System.Numerics) cho số “vô hạn”.  
- Chú ý **decimal** (base-10) cho tiền tệ/chính xác, **double/float** (IEEE754) cho khoa học/hiệu năng.

---

## 4. Kiểu đặc biệt: `object`, `string`, `dynamic`, `void`, `null`

- **`object`**: gốc của mọi kiểu; có `ToString()`, `Equals()`, `GetHashCode()`. Bạn có thể override ở class/struct.  
- **`string`**: immutable, interning một phần (literal), thao tác nhiều → dùng `StringBuilder`. So sánh: `StringComparison`.  
- **`dynamic`**: defer binding qua DLR (runtime); linh hoạt nhưng mất an toàn kiểu/IntelliSense, có chi phí.  
- **`void`**: chỉ dùng làm **kiểu trả về**; trong IL là `System.Void`.  
- **`null`**: literal biểu diễn “không tham chiếu/không giá trị”; với value type chỉ có trong `Nullable<T>`.

---

## 5. Struct, `readonly struct`, `ref struct` (byref-like)

- **`struct`**: value type do người dùng định nghĩa; phù hợp dữ liệu nhỏ, bất biến, nhiều instance. Không nên vượt ~16–24 byte nếu dùng nhiều.  
- **`readonly struct`**: mọi field readonly; tối ưu copy/defensive-copy; an toàn bất biến.  
- **`ref struct`**: *byref-like* (ví dụ `Span<T>`, `ReadOnlySpan<T>`) với **ràng buộc nghiêm**:  
  - Không boxed, không dùng làm field của class, không dùng trong async/iterator, không capture lambda, không trong `IEnumerable<T>`.  
  - Mục tiêu: truy cập bộ nhớ hiệu quả, an toàn (stack-only).

```csharp
public readonly struct Point(int x, int y)
{
    public int X { get; } = x;
    public int Y { get; } = y;
}
```

---

## 6. Enum & `[Flags]`

- `enum` là value type, **nền tảng** là kiểu số (mặc định `int`).  
- Dùng `[Flags]` cho bitmask; giá trị nên là mũ 2 (1,2,4,8…).

```csharp
[Flags]
public enum FileAccess { None=0, Read=1, Write=2, Execute=4 }
var rights = FileAccess.Read | FileAccess.Write;
bool canWrite = rights.HasFlag(FileAccess.Write);
```

---

## 7. Tuples & `ValueTuple`, deconstruction

- `ValueTuple<T1,...>` là **value type** (khác `Tuple<>` tham chiếu). Hỗ trợ **deconstruction**:

```csharp
(int x, int y) Get() => (10, 20);
var (a, b) = Get();
```

- Đặt tên phần tử: `(int X, int Y)` → `p.X`. Dùng cho trả nhiều giá trị, nhưng **API công khai** lớn nên cân nhắc type rõ nghĩa.

---

## 8. Records (record class / record struct)

- **Record** cung cấp **equality theo giá trị**, `with`-expression, phù hợp mô hình dữ liệu bất biến.  
- `record class User(string Id, string Name);`  
- `record struct` là **value type** có semantics tương tự (từ C# 10).

```csharp
var u1 = new User("1","Alice");
var u2 = u1 with { Name = "Bob" };
Console.WriteLine(u1 == u2); // false (so sánh theo giá trị)
```

---

## 9. Mảng (arrays): 1D, nhiều chiều, jagged, `Span<T>` (overview)

- **1D**: `T[]` phổ biến nhất.  
- **Nhiều chiều**: `T[,]` (rectangular).  
- **Jagged**: `T[][]` (mảng các mảng, linh hoạt).  
- Array là **reference type**, covariant (có rủi ro `ArrayTypeMismatchException`).

```csharp
object[] arr = new string[2];   // hợp lệ compile-time
arr[0] = 123;                   // runtime: ArrayTypeMismatchException
```

**`Span<T>`/`ReadOnlySpan<T>`** (byref-like, `ref struct`): lát cắt bộ nhớ hiệu quả, không cấp phát; dùng cho xử lý buffer, text. Không lưu trữ lâu dài, không dùng qua async/iterator.

---

## 10. Nullable Reference Types (NRT)

- Bật bằng `#nullable enable` hoặc trong project (`<Nullable>enable</Nullable>`).  
- Phân biệt `string` (non-null) và `string?` (có thể null); compiler sinh *warnings* giúp tránh `NullReferenceException`.  
- Chú ý các **annotation attributes** trong `System.Diagnostics.CodeAnalysis` như `NotNull`, `MaybeNull`, `MemberNotNull`, `NotNullWhen(bool)`,… để mô tả hợp đồng nullability cho API phức tạp.

```csharp
#nullable enable
string? name = GetNameOrNull();
if (name is not null)
{
    Console.WriteLine(name.Length); // an toàn
}
```

---

## 11. Khai báo biến & suy luận kiểu (`var`, target-typed, default literal)

- **`var`**: suy luận tại compile-time từ vế phải (vẫn *strongly-typed*). 
- **Target-typed `new`** (C# 9): `List<int> list = new();`  
- **Default literal** (C# 7.1): `T x = default;` (tự suy kiểu T).  
- **Anonymous types**: `new { Name = "A", Age = 1 }` (chỉ dùng nội bộ).

---

## 12. Giá trị mặc định (default values)

- `default(T)`:
  - Value type: tất cả bit 0 (`0`, `false`, `\0`, struct với field default).  
  - Reference type: `null`.  
- Nullable: `default(int?)` == `null`.

```csharp
int x = default;        // 0
string? s = default;    // null
DateTime dt = default;  // 01/01/0001 ...
```

---

## 13. Generics & ràng buộc (`where`, `new()`, `struct/class/unmanaged/notnull`), phương sai (variance)

### 13.1 Ràng buộc (`where`)

- `where T : class` / `struct` / `unmanaged` / `notnull`  
- `where T : new()` (có ctor không tham số)  
- `where T : SomeBase, ISomeInterface` (đa ràng buộc)

```csharp
T Create<T>() where T : new() => new T();
bool Equal<T>(T a, T b) where T : IEquatable<T> => a.Equals(b);
```

**`unmanaged`**: chỉ chứa các field unmanaged (không tham chiếu) → phù hợp interop/unsafe/fixed size.

### 13.2 Phương sai (variance)

- `out` (covariant — chỉ *xuất* `T`) cho **interfaces/delegates**: `IEnumerable<out T>`.  
- `in` (contravariant — chỉ *nhập* `T`): `IComparer<in T>`.  
- Giúp tái sử dụng kiểu generic giữa kế thừa: `IEnumerable<string>` có thể dùng nơi yêu cầu `IEnumerable<object>`.

> Lưu ý: **Array covariance** tồn tại nhưng *nguy hiểm* (mục 9).

---

## 14. Namespace, `using`, `global using`, `extern alias`

- **Namespace**: tổ chức type theo không gian tên; từ C# 10 có **file-scoped namespace**:

```csharp
namespace MyApp.Core; // file-scoped
```

- **`using` directive**: import namespace; **`global using`** (C# 10) áp dụng cho toàn project.  
- **`extern alias`**: phân biệt 2 assembly có cùng namespace/type:

```csharp
extern alias LibA;
extern alias LibB;
using A = LibA::Company.Product;
using B = LibB::Company.Product;
```

---

## 15. Chuyển đổi & ép kiểu: implicit/explicit, user-defined, pattern matching

### 15.1 Chuyển đổi chuẩn

- **Implicit**: an toàn, không mất dữ liệu (`int -> long`).  
- **Explicit**: có thể mất dữ liệu, cần cast (`double -> int`).  
- **Parse/TryParse** cho string → số/ngày…

### 15.2 User-defined conversion

Trong class/struct, bạn có thể định nghĩa:

```csharp
public readonly struct Dollars
{
    public decimal Amount { get; }
    public Dollars(decimal a) => Amount = a;
    public static implicit operator Dollars(decimal a) => new(a);
    public static explicit operator decimal(Dollars d) => d.Amount;
}
```

### 15.3 Pattern matching

- `is` pattern, **`switch` expression**, property/relational/list patterns.  
- Vừa kiểm tra kiểu, vừa “rút” giá trị một cách an toàn, ngắn gọn.

```csharp
object x = Get();
if (x is Person { Age: >= 18 } p)
    Console.WriteLine($"{p.Name} is adult");
```

---

## 16. Unsafe & unmanaged types (overview), function pointers

- **Unsafe context** (`unsafe { ... }`): dùng con trỏ (`T*`), `stackalloc`, `fixed`. Chỉ dùng khi **thật cần** (interop/hiệu năng đặc biệt).  
- **Unmanaged types**: không chứa reference; có thể dùng trong `sizeof`, `stackalloc`, `unmanaged` constraint.  
- **Function pointers** (C# 9, unsafe): `delegate*<int, void>` — hiệu năng cao khi interop/native, nhưng mất an toàn kiểu ở C# mức cao; đa phần nên dùng **delegate**.

---

## 17. Sơ đồ “type tree” (ASCII)

```
object
├─ Value types (System.ValueType)
│  ├─ Primitives: bool, char, sbyte/byte, short/ushort, int/uint, long/ulong, nint/nuint, float, double, decimal, Half
│  ├─ Enums: enum E : int { ... }
│  ├─ Structs: DateTime, Guid, ValueTuple<...>, custom struct/readonly struct
│  └─ Byref-like: ref struct (Span<T>, ReadOnlySpan<T>)
└─ Reference types
   ├─ class (String, Exception, Stream, List<T>, ...)
   │  ├─ record class
   │  └─ arrays: T[], T[,], T[][] (covariant, ref type)
   ├─ interface (IDisposable, IEnumerable<T>, ...)
   ├─ delegate (Action, Func<...>, custom delegates)
   └─ dynamic (runtime-bound)
```
