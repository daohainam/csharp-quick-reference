# Toán tử (Operators) trong C# — Bản chi tiết hiện đại

## Mục lục

- [Toán tử (Operators) trong C# — Bản chi tiết hiện đại](#toán-tử-operators-trong-c--bản-chi-tiết-hiện-đại)
  - [Mục lục](#mục-lục)
  - [1. Tổng quan \& nguyên tắc](#1-tổng-quan--nguyên-tắc)
  - [2. Bảng ưu tiên (precedence) \& kết hợp (associativity)](#2-bảng-ưu-tiên-precedence--kết-hợp-associativity)
  - [3. Toán tử số học](#3-toán-tử-số-học)
  - [4. Toán tử tăng/giảm `++`/`--`](#4-toán-tử-tănggiảm---)
  - [5. So sánh \& bằng/khác](#5-so-sánh--bằngkhác)
  - [6. Bit \& logic: `&` `|` `^` `~` `&&` `||` `!`](#6-bit--logic-------)
  - [7. Dịch bit: `<<` `>>` `>>>`](#7-dịch-bit---)
  - [8. Gán \& gán hợp (compound assignment)](#8-gán--gán-hợp-compound-assignment)
  - [9. Điều kiện: `?:` (ternary)](#9-điều-kiện--ternary)
  - [10. Null: `??`, `??=`, null-conditional `?.` `?[]`, null-forgiving `!`](#10-null---null-conditional---null-forgiving-)
  - [11. Pattern \& cast: `is`, `as`, cast `(<T>)`](#11-pattern--cast-is-as-cast-t)
    - [`is` (pattern matching)](#is-pattern-matching)
    - [`as`](#as)
    - [Cast tường minh `(T)`](#cast-tường-minh-t)
  - [12. Range \& Index: `..`, `^`](#12-range--index--)
  - [13. Toán tử ngữ nghĩa đặc biệt](#13-toán-tử-ngữ-nghĩa-đặc-biệt)
  - [14. Operator overloading](#14-operator-overloading)
  - [15. User-defined conversions `implicit`/`explicit`](#15-user-defined-conversions-implicitexplicit)
  - [16. Lifted operators \& nullable](#16-lifted-operators--nullable)
  - [17. Unsafe/pointer operators: `*` `&` `->` `[]` `fixed`](#17-unsafepointer-operators------fixed)
  - [18. Best practices \& cảnh báo thường gặp](#18-best-practices--cảnh-báo-thường-gặp)

---

## 1. Tổng quan & nguyên tắc

- **Biểu thức** được đánh giá theo *precedence* và *associativity* → hiểu đúng để tránh bug.  
- Nhiều toán tử hỗ trợ **overload** trên struct/class; với **nullable** (`T?`), đa số toán tử được “nâng” (*lifted*) để hoạt động tự nhiên.  
- Toán tử **logic ngắn mạch**: `&&`, `||` — chỉ đánh giá vế phải khi cần.  
- Cẩn trọng **overflow** trong số học nguyên → dùng `checked` khi cần.

---

## 2. Bảng ưu tiên (precedence) & kết hợp (associativity)

Từ **cao** → **thấp** (tóm tắt nhóm chính):

1. **Postfix**: `x++` `x--` `x!` (null-forgiving) `a[b]` `a.b` `a?.b` `a?[^i]` `a()` `new T()` `typeof` `checked` `unchecked` `default` `nameof` `stackalloc` (phụ thuộc ngữ cảnh) — *trái → phải*  
2. **Unary**: `+x` `-x` `!x` `~x` `++x` `--x` `&x` `*x` `await x` `^i` (index) — *phải → trái*  
3. **Multiplicative**: `*` `/` `%` — *trái → phải*  
4. **Additive**: `+` `-` — *trái → phải*  
5. **Shift**: `<<` `>>` `>>>` — *trái → phải*  
6. **Relational & type test**: `<` `>` `<=` `>=` `is` `as` — *trái → phải*  
7. **Equality**: `==` `!=` — *trái → phải*  
8. **Bitwise AND/XOR/OR**: `&` `^` `|` — *trái → phải*  
9. **Conditional AND/OR**: `&&` `||` — *trái → phải*  
10. **Null-coalescing**: `??` — *trái → phải*  
11. **Conditional**: `?:` — *phải → trái*  
12. **Assignment**: `=` `+=` `-=` `*=` `/=` `%=` `&=` `|=` `^=` `<<=` `>>=` `>>>=` `??=` — *phải → trái*  
13. **Lambda**: `=>` (ràng buộc riêng, thường thấp)  

> Khi nghi ngờ, **dùng ngoặc** để làm rõ ý đồ.

---

## 3. Toán tử số học

- Nhị phân: `+` `-` `*` `/` `%`  
- Đơn ngôi: `+` `-`

```csharp
int a = 7 / 2;      // 3 (chia nguyên)
double b = 7 / 2.0; // 3.5
int c = -(-5);      // 5
```

**Overflow**: dùng `checked` để ném `OverflowException`:

```csharp
checked { int x = int.MaxValue + 1; } // ném
```

`decimal` phù hợp tài chính; `%` hoạt động trên số nguyên và `decimal`.

---

## 4. Toán tử tăng/giảm `++`/`--`

- **Prefix**: `++x` trả giá trị **sau khi tăng**.  
- **Postfix**: `x++` trả giá trị **trước khi tăng**.

```csharp
int x = 1;
int a = ++x; // x=2, a=2
int b = x++; // x=3, b=2
```

Tránh dùng trong biểu thức phức tạp gây khó đọc.

---

## 5. So sánh & bằng/khác

`<` `>` `<=` `>=` `==` `!=`

```csharp
string s1 = "a", s2 = new string("a");
bool eq = s1 == s2; // true: string so sánh theo **nội dung** (ordinal)
```

- Với reference type khác, mặc định `==` là **reference equality**, trừ khi overload (như `string`, `record`).  
- Khi so sánh chuỗi “ngôn ngữ dân gian”, cân nhắc `Compare(..., StringComparison.CurrentCultureIgnoreCase)`; còn đa số logic app dùng `Ordinal/OrdinalIgnoreCase`.

---

## 6. Bit & logic: `&` `|` `^` `~` `&&` `||` `!`

- Với **số nguyên**: `&` `|` `^` `~` là bitwise.  
- Với **bool**: `&` `|` là **logic không ngắn mạch**; `&&` `||` là **ngắn mạch**.

```csharp
bool b = Expensive() & Cheap();  // cả hai chạy
bool c = Expensive() && Cheap(); // nếu Expensive() == false → bỏ qua Cheap()
```

`^` với bool = XOR logic.

---

## 7. Dịch bit: `<<` `>>` `>>>`

- `<<` dịch trái (điền 0).  
- `>>` dịch phải **theo dấu** (arithmetic shift) cho kiểu có dấu (`int`, `long`).  
- `>>>` **dịch phải không dấu** (logical shift) — **C# 11**.

```csharp
int  a = -8;        // 111..1000
int  b = a >> 1;    // 111..1100  (giữ bit dấu)
uint c = 0xF0u >>> 2; // 0x3C (C# 11)
```

---

## 8. Gán & gán hợp (compound assignment)

`=`, `+=`, `-=`, `*=`, `/=`, `%=` `&=`, `|=`, `^=`, `<<=`, `>>=`, `>>>=` (C# 11), `??=`

```csharp
int x = 1;
x += 2; // x = x + 2
dict[key] ??= new List<int>(); // khởi tạo nếu null
```

Gán hợp được “nâng” để bảo toàn semantics với indexer/phép truy cập phức tạp.

---

## 9. Điều kiện: `?:` (ternary)

```csharp
var label = (age >= 18) ? "Adult" : "Minor";
```

- Kết quả phải có kiểu suy ra được; có thể tham gia **overload resolution**.  
- Kết hợp `?:` và `??` cẩn trọng **thứ tự ưu tiên** (`??` cao hơn `?:`).

---

## 10. Null: `??`, `??=`, null-conditional `?.` `?[]`, null-forgiving `!`

```csharp
string? name = GetNameOrNull();
int len = name?.Length ?? 0; // ?. tránh NRE, ?? cung cấp mặc định

dict?["key"]?.ToString();    // ?[] với indexer

obj!.Property // null-forgiving: cam kết với compiler là không null (cẩn thận)
```

- `??` và `??=` chỉ kiểm `== null`.  
- `?.` trả **null** nếu vế trái null, không ném NRE. Chuỗi `a?.B?.C` rất hữu dụng cho truy cập sâu.

---

## 11. Pattern & cast: `is`, `as`, cast `(<T>)`

### `is` (pattern matching)

```csharp
if (x is Person { Age: >= 18 } p) { /* ... */ }
```

- Hỗ trợ **type pattern**, **property/relational/list patterns**, kết hợp `and/or/not` (C# 9+).

### `as`

- Cast **an toàn**: trả `null` nếu không chuyển được (reference/nullable).

```csharp
var p = obj as Person;
if (p != null) Console.WriteLine(p.Name);
```

### Cast tường minh `(T)`

- Có thể ném `InvalidCastException` nếu không tương thích.

```csharp
var p2 = (Person)obj; // nếu obj không phải Person → ném
```

---

## 12. Range & Index: `..`, `^`

- `^i` → chỉ số **từ cuối** (1 = phần tử cuối).  
- `a[start..end]` → **slice** từ `start` (inclusive) tới `end` (exclusive).

```csharp
int[] arr = {0,1,2,3,4};
var last = arr[^1];    // 4
var mid  = arr[1..^1]; // {1,2,3}
```

Hoạt động với `string`, `Span<T>`, `Index`, `Range`, và các type tự cài indexer `this[Index]`/`this[Range]`.

---

## 13. Toán tử ngữ nghĩa đặc biệt

- **`await`**: tạm ngưng method async cho đến khi awaitable hoàn thành. (*Xem chương Async*)  
- **`nameof(x)`**: lấy **tên** định danh dạng chuỗi, không bị rename runtime (an toàn refactor).  
- **`sizeof(T)`**: kích thước byte của kiểu unmanaged (với managed struct, thường cần `unsafe`).  
- **`typeof(T)`**: trả `System.Type` của `T`.  
- **`checked` / `unchecked`**: bật/tắt kiểm tra overflow số học.  
- **`default`**: literal/expr tạo giá trị mặc định của `T`.  
- **`new`**: tạo instance; cũng là **operator** ở mức ngữ nghĩa.  
- **`stackalloc`**: cấp phát trên stack (unsafe) cho buffer `Span<T>`/con trỏ.

```csharp
var s = nameof(Person.Name);     // "Name"
var t = typeof(List<string>);    // System.Type
Span<byte> buf = stackalloc byte[256]; // stack buffer
```

---

## 14. Operator overloading

- Cho phép định nghĩa lại nghĩa của toán tử trên **class/struct** (không phải interface).  
- Khai báo `public static <ret> operator +(T a, T b) { ... }`.

**Overload được** (tiêu biểu): `+ - ! ~ ++ -- true false * / % & | ^ << >> == != < > <= >=`  
**Không overload được**: `=` `&&` `||` `?:` `??` `?.` `?[]` `=>` `.` `[]` `()` v.v.

Quy tắc quan trọng:

- Nếu overload `==` → nên override `Equals`/`GetHashCode` và overload `!=`.  
- `true`/`false` giúp type dùng trong `if`, kết hợp với `&`/`|`.  
- Tôn trọng kỳ vọng của người dùng (tính giao hoán/bất biến).

Ví dụ:

```csharp
public readonly struct Vector2(double x, double y)
{
    public double X { get; } = x;
    public double Y { get; } = y;

    public static Vector2 operator +(Vector2 a, Vector2 b)
        => new(a.X + b.X, a.Y + b.Y);

    public static bool operator ==(Vector2 a, Vector2 b)
        => a.X == b.X && a.Y == b.Y;
    public static bool operator !=(Vector2 a, Vector2 b) => !(a == b);
    public override bool Equals(object? o) => o is Vector2 v && this == v;
    public override int GetHashCode() => HashCode.Combine(X, Y);
}
```

---

## 15. User-defined conversions `implicit`/`explicit`

Cho phép chuyển đổi giữa type của bạn và type khác:

```csharp
public readonly struct Dollars
{
    public decimal Amount { get; }
    public Dollars(decimal amount) => Amount = amount;

    public static implicit operator Dollars(decimal a) => new(a);
    public static explicit operator decimal(Dollars d) => d.Amount;
}

Dollars d = 10.5m;           // implicit
decimal v = (decimal)d;      // explicit
```

- Dùng `implicit` cho **an toàn** (không mất dữ liệu); `explicit` khi có thể mất thông tin/chi phí.  
- Tránh tạo chuyển đổi **mơ hồ** gây lỗi overload resolution.

---

## 16. Lifted operators & nullable

Với `T?` (nullable value type), hầu hết toán tử nhị phân được “nâng”:

```csharp
int? a = null, b = 10;
int? c = a + b;   // => null (nếu bất kỳ toán hạng null)
bool d = a == b;  // => false (trừ khi cả hai null → true)
```

- So sánh `==`/`!=` giữa `T?` tuân quy tắc đặc biệt (cả hai `null` → `true` cho `==`).  
- Dùng `??` để thay mặc định trước khi tính nếu cần kết quả non-null.

---

## 17. Unsafe/pointer operators: `*` `&` `->` `[]` `fixed`

Trong **unsafe context**:

- `*p` truy cập giá trị, `&x` lấy địa chỉ, `p->Member` truy cập member qua con trỏ struct, `p[i]` index pointer arithmetic.  
- `fixed` cố định object/array để lấy địa chỉ ổn định cho GC.

```csharp
unsafe
{
    int x = 10;
    int* p = &x;
    *p = 20;
}
```

> Chỉ dùng khi thật cần (interop/hiệu năng đặc biệt).

---

## 18. Best practices & cảnh báo thường gặp

1. **Ưu tiên biểu thức rõ ràng**: thêm ngoặc khi tổ hợp `&&`, `||`, `??`, `?:`.  
2. **Chuỗi &**: với chuỗi, `+` cấp phát; trong vòng lặp dùng `StringBuilder`.  
3. **So sánh chuỗi**: rõ ràng `StringComparison`/`StringComparer` thay vì mặc định.  
4. **Nullable**: tận dụng `?.` + `??` để tránh NRE; tránh lạm dụng `!` (null-forgiving).  
5. **Overflow**: trong tính toán tài chính, dùng `decimal`; bật `checked` khi cần chính xác.  
6. **Overload toán tử**: nhất quán với trực giác; luôn đi đôi `==`/`!=`; không phá vỡ luật toán học (ví dụ `a + b` = `b + a` nếu kỳ vọng).  
7. **Shift mới `>>>`**: đảm bảo target .NET/C# hỗ trợ (C# 11+).  
8. **Range/Index**: coi chừng `IndexOutOfRangeException`; nhớ `end` là **exclusive**.  
9. **`as` vs cast**: dùng `as` khi chấp nhận `null`; cast khi chắc chắn đúng (hoặc muốn ném lỗi).  
10. **IQueryable**: một số toán tử ở LINQ có semantics khác (dịch SQL) — xem chương LINQ.
