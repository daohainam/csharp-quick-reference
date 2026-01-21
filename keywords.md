# Keywords

---

## 1. `abstract`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:**  
  - Đánh dấu class/member trừu tượng, không có triển khai đầy đủ.  
  - Class `abstract` không thể `new`.  
  - Member `abstract` bắt buộc phải được `override` trong lớp con.

**Ví dụ:**

```csharp
public abstract class Shape
{
    public abstract double Area();

    public virtual void Print()
        => Console.WriteLine($"Area = {Area()}");
}

public sealed class Circle : Shape
{
    public double Radius { get; }

    public Circle(double radius) => Radius = radius;

    public override double Area() => Math.PI * Radius * Radius;
}
```

**Ghi chú:**  
Thường dùng khi muốn định nghĩa “hợp đồng + một phần behavior chung” cho một nhóm class.

---

## 2. `as`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Cast an toàn giữa reference type / nullable value type. Thất bại ⇒ `null`, không ném exception.

**Ví dụ:**

```csharp
object obj = "hello";

string? s = obj as string;        // "hello"
FileStream? fs = obj as FileStream; // null

if (s != null)
{
    Console.WriteLine(s.ToUpper());
}
```

**Ghi chú:**  
Luôn nhớ kiểm tra `null` sau khi dùng `as`. Nếu muốn lỗi rõ ràng hơn, dùng cast thường `(T)obj`.

---

## 3. `base`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Dùng trong class dẫn xuất để gọi ctor hoặc member của base class (đặc biệt trong `override`).

**Ví dụ:**

```csharp
public class Animal
{
    public string Name { get; }

    public Animal(string name) => Name = name;

    public virtual void Speak()
        => Console.WriteLine($"{Name} makes a sound");
}

public class Dog : Animal
{
    public Dog(string name) : base(name) { }

    public override void Speak()
    {
        base.Speak(); // gọi behavior chung
        Console.WriteLine($"{Name} barks");
    }
}
```

**Ghi chú:**  
Không dùng được trong `struct`. Nếu base không có ctor mặc định, lớp con phải gọi `base(...)`.

---

## 4. `bool`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Kiểu Boolean với hai giá trị `true` / `false`. Không cho dùng int thay bool như C/C++.

**Ví dụ:**

```csharp
bool isActive = true;

if (isActive)
{
    Console.WriteLine("Enabled");
}
else
{
    Console.WriteLine("Disabled");
}
```

---

## 5. `break`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Thoát khỏi vòng lặp (`for`, `foreach`, `while`, `do`) hoặc `switch` ngay lập tức.

**Ví dụ:**

```csharp
for (int i = 0; i < 100; i++)
{
    if (i == 10)
        break;
}

switch (statusCode)
{
    case 200:
        Console.WriteLine("OK");
        break;
    default:
        Console.WriteLine("Other");
        break;
}
```

**Ghi chú:**  
Quá nhiều `break`/`continue` trong cùng một vòng lặp có thể làm flow khó đọc.

---

## 6. `byte`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số nguyên không dấu 8-bit (`0..255`), thường dùng cho buffer, stream, dữ liệu nhị phân.

**Ví dụ:**

```csharp
byte b = 255;
byte[] buffer = new byte[1024];
```

**Ghi chú:**  
Phép toán trên `byte` trả về `int`, cần cast ngược nếu muốn gán lại vào `byte`.

---

## 7. `case`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Định nghĩa nhánh trong `switch` (statement).

**Ví dụ:**

```csharp
switch (status)
{
    case 200:
        Console.WriteLine("OK");
        break;
    case 404:
        Console.WriteLine("Not Found");
        break;
    default:
        Console.WriteLine("Unknown");
        break;
}
```

**Ghi chú:**  
Trong `switch` cũ, mỗi `case` phải kết thúc bằng `break`/`return`/`goto`… Trên `switch expression` (C# 8+) không dùng `case` kiểu này nữa.

---

## 8. `catch`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Bắt exception được ném từ khối `try`.

**Ví dụ:**

```csharp
try
{
    File.ReadAllText("data.txt");
}
catch (FileNotFoundException ex)
{
    Console.WriteLine("File không tồn tại");
}
catch (IOException ex)
{
    Console.WriteLine("Lỗi IO khác");
}
catch (Exception ex)
{
    Console.WriteLine("Lỗi không xác định");
}
```

**Ghi chú:**  
Tránh `catch (Exception) { }` bỏ trống – rất khó debug. Nên log hoặc wrap thành exception có ý nghĩa cụ thể.

---

## 9. `char`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Ký tự Unicode 16-bit (`System.Char`). Một số ký tự (emoji, ký hiệu phức tạp) cần 2 `char` (surrogate pair).

**Ví dụ:**

```csharp
char c = 'A';
bool isLetter = char.IsLetter(c); // true
```

**Ghi chú:**  
Đừng giả định “1 ký tự người dùng = 1 `char`”; với Unicode phức tạp nên dùng API trên `string` hoặc `System.Text.Rune`.

---

## 10. `checked`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Bật kiểm tra overflow cho toán tử số học/ép kiểu integral. Overflow ⇒ ném `OverflowException`.

**Ví dụ:**

```csharp
int x = int.MaxValue;

checked
{
    int y = x + 1; // OverflowException
}
```

**Ghi chú:**  
Dùng ở chỗ cần đảm bảo không overflow (tài chính, số quan trọng). Có thể bật mặc định trong project và dùng `unchecked` cho các chỗ đặc biệt.

---

## 11. `class`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Khai báo **reference type** (class).

**Ví dụ:**

```csharp
public class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }

    public void SayHello()
        => Console.WriteLine($"Hi, I'm {Name} ({Age})");
}
```

**Ghi chú:**  
Class là reference type → được cấp phát trên heap, truyền qua reference. Với type nhỏ, immutable, nhạy hiệu năng, cân nhắc `struct` hoặc `record struct`.

---

## 12. `const`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Khai báo hằng compile-time. Giá trị được inline vào IL của caller.

**Ví dụ:**

```csharp
public const double Pi = 3.14159265358979;
private const string AppName = "MyApp";
```

**Ghi chú:**  
Thay đổi giá trị `public const` trong library không tự update cho code client đã build; cần rebuild client. Với giá trị có thể thay đổi, dùng `static readonly`.

---

## 13. `continue`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Bỏ phần còn lại của vòng lặp hiện tại, nhảy tới lần lặp tiếp theo.

**Ví dụ:**

```csharp
foreach (var item in items)
{
    if (item == null) continue;
    Process(item);
}
```

---

## 14. `decimal`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số thập phân 128-bit, độ chính xác cao, rất phù hợp tài chính / tiền tệ.

**Ví dụ:**

```csharp
decimal price = 19.99m;
decimal qty   = 3;
decimal total = price * qty; // 59.97m
```

**Ghi chú:**  
Chậm hơn `double`; không lý tưởng cho tính toán khoa học nặng.

---

## 15. `default`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:**  
  - Trong `switch`: nhánh mặc định.  
  - Trong expression: `default(T)` hoặc `default` (C# 7.1+) ⇒ giá trị mặc định của kiểu (`0`, `false`, `null`, struct rỗng…).

**Ví dụ:**

```csharp
int x = default;      // 0
string? s = default;  // null

T Create<T>() => default!;
```

---

## 16. `delegate`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Khai báo kiểu đại diện cho method (function pointer an toàn). Dùng cho callback, event handler, v.v.

**Ví dụ:**

```csharp
public delegate void LogHandler(string message);

public class Worker
{
    public LogHandler? Logger { get; set; }

    public void DoWork()
    {
        Logger?.Invoke("Working...");
    }
}
```

**Ghi chú:**  
Trong code hiện đại, thường dùng `Action<>`, `Func<>` thay vì tự khai báo delegate, trừ khi cần type public có tên rõ ràng.

---

## 17. `do`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Bắt đầu vòng lặp `do { ... } while (cond)` – thân vòng lặp **chạy ít nhất 1 lần**.

**Ví dụ:**

```csharp
int i = 0;

do
{
    Console.WriteLine(i);
    i++;
} while (i < 3);
```

---

## 18. `double`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số thực 64-bit theo chuẩn IEEE 754; dùng nhiều cho tính toán khoa học, đồ họa, đo đạc.

**Ví dụ:**

```csharp
double x = 0.1 + 0.2;
Console.WriteLine(x); // 0.30000000000000004
```

**Ghi chú:**  
Luôn tồn tại sai số floating-point; khi so sánh nên dùng epsilon, không so sánh trực tiếp bằng `==` cho số thực.

---

## 19. `else`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Nhánh “ngược lại” của `if`.

**Ví dụ:**

```csharp
if (score >= 90)
    grade = "A";
else if (score >= 80)
    grade = "B";
else
    grade = "C";
```

---

## 20. `enum`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Định nghĩa kiểu liệt kê (enum) với underlying integral type (mặc định `int`).

**Ví dụ:**

```csharp
public enum OrderStatus
{
    Pending = 0,
    Processing = 1,
    Completed = 2,
    Cancelled = 3
}
```

**Ghi chú:**  
Enum vẫn là số bên dưới ⇒ cast được giá trị không hợp lệ; nếu nhận từ bên ngoài cần validate.

---

## 21. `event`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Khai báo event .NET, cho phép subscribe/unsubscribe handler.

**Ví dụ:**

```csharp
public class Button
{
    public event EventHandler? Click;

    protected virtual void OnClick()
        => Click?.Invoke(this, EventArgs.Empty);

    public void SimulateClick() => OnClick();
}
```

**Ghi chú:**  
Cẩn thận memory leak nếu subscriber không hủy đăng ký ở các scenario sống lâu (winforms, WPF, v.v.).

---

## 22. `explicit`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Toán tử chuyển kiểu tường minh, bắt buộc dùng cast.

**Ví dụ:**

```csharp
public readonly struct Meter
{
    public double Value { get; }
    public Meter(double value) => Value = value;

    public static explicit operator Meter(double v) => new(v);
    public static explicit operator double(Meter m) => m.Value;
}

Meter m = (Meter)5.0;
double d = (double)m;
```

---

## 23. `extern`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Chỉ ra method được implement bên ngoài (thường là native DLL, dùng với `DllImport`).

**Ví dụ:**

```csharp
using System.Runtime.InteropServices;

class NativeMethods
{
    [DllImport("kernel32.dll")]
    public static extern void Sleep(uint milliseconds);
}
```

---

## 24. `false`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Hằng Boolean `false`.

**Ví dụ:**

```csharp
bool ok = false;
if (!ok)
{
    Console.WriteLine("Not OK");
}
```

---

## 25. `field`

- **Loại:** reserved · **C#:** 14.0  
- **Mục đích:** Cho phép truy cập vào backing field.

**Ví dụ:**

```csharp
public string Message
{
    get;
    set => field = value ?? throw new ArgumentNullException(nameof(value));
}
```

---

## 26. `finally`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Khối cleanup luôn chạy sau `try`/`catch` (dù có exception hay không).

**Ví dụ:**

```csharp
FileStream? stream = null;
try
{
    stream = File.OpenRead("data.txt");
    // xử lý
}
finally
{
    stream?.Dispose();
}
```

**Ghi chú:**  
Cẩn thận không ném exception mới từ `finally` (dễ che mất exception gốc). Hiện đại hơn là dùng `using` / `using var`.

---

## 27. `fixed`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Trong `unsafe`, pin object để lấy địa chỉ cố định (pointer) cho interop/native.

**Ví dụ:**

```csharp
unsafe
{
    int[] arr = { 1, 2, 3 };
    fixed (int* p = &arr[0])
    {
        // p trỏ tới arr[0], GC không được di chuyển arr trong khối fixed
        Console.WriteLine(*p);
    }
}
```

---

## 28. `float`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số thực 32-bit (nhẹ hơn nhưng kém chính xác hơn `double`).

**Ví dụ:**

```csharp
float f = 1.23f;
```

---

## 29. `for`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Vòng lặp có phần khởi tạo, điều kiện, bước tăng/giảm rõ ràng.

**Ví dụ:**

```csharp
for (int i = 0; i < items.Length; i++)
{
    Console.WriteLine(items[i]);
}
```

---

## 30. `foreach`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Vòng lặp tiện dụng trên mọi `IEnumerable` / `IEnumerable<T>`.

**Ví dụ:**

```csharp
foreach (var item in items)
{
    Console.WriteLine(item);
}
```

---

## 31. `goto`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Nhảy tới label hoặc `case`/`default` trong `switch`.

**Ví dụ:**

```csharp
switch (option)
{
    case 0:
        Console.WriteLine("Zero");
        goto case 1;
    case 1:
        Console.WriteLine("One");
        break;
}
```

**Ghi chú:**  
Thường được xem là “code smell”, trừ vài pattern rất hiếm (ví dụ thoát lồng nhiều vòng).

---

## 32. `if`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Câu lệnh rẽ nhánh cơ bản.

**Ví dụ:**

```csharp
if (user is null)
    throw new ArgumentNullException(nameof(user));
```

---

## 33. `implicit`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Toán tử chuyển kiểu ngầm định (không cần cast).

**Ví dụ:**

```csharp
public readonly struct Meter
{
    public double Value { get; }
    public Meter(double value) => Value = value;

    public static implicit operator Meter(double v) => new(v);
}

Meter m = 5.0; // implicit
```

---

## 34. `in`

- **Loại:** reserved · **C#:** 1.0 (thêm ý nghĩa mới ở C# 7.2)  
- **Mục đích:**  
  - Trong tham số: `in` ⇒ readonly by-ref.  
  - Trong `foreach`: một phần cú pháp (`foreach (var x in xs)`).

**Ví dụ (C# 7.2+):**

```csharp
public static double Distance(in Point a, in Point b)
{
    // a, b truyền by-ref nhưng không gán lại được
    // tránh copy struct lớn
}
```

---

## 35. `int`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số nguyên 32-bit có dấu, kiểu integer phổ biến nhất.

**Ví dụ:**

```csharp
int count = 42;
```

---

## 36. `interface`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Định nghĩa hợp đồng (method, property, event…) mà class/struct phải thực hiện.

**Ví dụ:**

```csharp
public interface ILogger
{
    void Log(string message);
}
```

---

## 37. `internal`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Access modifier – chỉ thấy được trong cùng assembly.

**Ví dụ:**

```csharp
internal class InternalHelper { }
```

---

## 38. `is`

- **Loại:** reserved · **C#:** 1.0 (pattern matching từ C# 7.0)  
- **Mục đích:**  
  - Kiểm tra kiểu (`obj is T`).  
  - Từ C# 7: pattern matching (`is T v`, `is null`, `is > 0`, v.v.).

**Ví dụ:**

```csharp
if (obj is string s && s.Length > 0)
{
    Console.WriteLine(s);
}
```

---

## 39. `lock`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Đồng bộ truy cập giữa các thread (`Monitor.Enter/Exit`).

**Ví dụ:**

```csharp
private readonly object _sync = new();
private int _counter;

public void Increment()
{
    lock (_sync)
    {
        _counter++;
    }
}
```

---

## 40. `long`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số nguyên 64-bit có dấu (`System.Int64`).

**Ví dụ:**

```csharp
long big = 1_000_000_000_000L;
```

---

## 41. `namespace`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Tổ chức không gian tên cho type.

**Ví dụ:**

```csharp
namespace MyApp.Core
{
    public class Service { }
}
```

---

## 42. `new`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:**  
  - Tạo instance: `new Type(...)`.  
  - Hide member base: `new void Foo()`.  
  - Constraint generic: `where T : new()` - yêu cầu kiểu T phải có constructor không tham số (public parameterless constructor).

**Ví dụ:**

```csharp
var p = new Person("Alice");

public class Base
{
    public void Print() => Console.WriteLine("Base");
}

public class Derived : Base
{
    public new void Print() => Console.WriteLine("Derived");
}

// Generic constraint với new():
public class Factory<T> where T : new()
{
    public T Create() => new T(); // Đảm bảo T có constructor không tham số
}
```

---

## 43. `null`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Giá trị “không tham chiếu tới object nào” cho reference type & nullable value type.

**Ví dụ:**

```csharp
string? name = null;
if (name is null)
{
    // ...
}
```

---

## 44. `object`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Kiểu gốc của mọi reference type (alias cho `System.Object`).

**Ví dụ:**

```csharp
object o = 42;   // boxing
int x = (int)o;  // unboxing
```

---

## 45. `operator`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Định nghĩa toán tử overload (`+`, `-`, `==`, conversion…) cho type custom.

**Ví dụ:**

```csharp
public readonly struct Money
{
    public decimal Value { get; }

    public Money(decimal value) => Value = value;

    public static Money operator +(Money a, Money b)
        => new(a.Value + b.Value);
}
```

---

## 46. `out`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Tham số output; method phải gán trước khi return.

**Ví dụ:**

```csharp
if (int.TryParse("123", out int value))
{
    Console.WriteLine(value);
}
```

---

## 47. `params`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Cho phép truyền số lượng đối số biến đổi (varargs).

**Ví dụ:**

```csharp
void Log(params string[] messages)
{
    foreach (var m in messages)
        Console.WriteLine(m);
}

Log("A", "B", "C");
```

---

## 48. `private`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Access modifier – chỉ trong cùng type.

**Ví dụ:**

```csharp
public class User
{
    private string _passwordHash = "";
}
```

---

## 49. `protected`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Access modifier – trong type và lớp dẫn xuất.

```csharp
public class Base
{
    protected void OnChanged() { }
}
```

---

## 50. `public`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Access modifier – public ở khắp nơi nếu nhìn thấy type/assembly.

```csharp
public class ApiClient { }
```

---

## 51. `readonly`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Field chỉ gán trong ctor hoặc tại điểm khai báo.

```csharp
public class Config
{
    public readonly string ConnectionString;

    public Config(string conn) => ConnectionString = conn;
}
```

---

## 52. `ref`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Tham số by-ref (đọc/ghi), C# 7+ có `ref local`, `ref return`.

```csharp
void Swap(ref int a, ref int b)
{
    int t = a; a = b; b = t;
}
```

---

## 53. `return`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Trả giá trị (nếu có) và kết thúc method/local function.

```csharp
int Double(int x) => x * 2;
```

---

## 54. `sbyte`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số nguyên 8-bit có dấu; không CLS-compliant, hiếm dùng.

```csharp
sbyte x = -5;
```

---

## 55. `sealed`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:**  
  - `sealed class`: cấm kế thừa.  
  - `sealed override`: không cho override tiếp.

Khi được áp dụng cho một lớp (class), từ khóa sealed sẽ ngăn các lớp khác kế thừa từ lớp đó. Trong ví dụ sau, lớp FinalType kế thừa từ lớp BaseType, nhưng không có lớp nào có thể kế thừa từ lớp FinalType.

```csharp
class BaseType {}
sealed class FinalType : BaseType {}
```

- Bạn cũng có thể dùng từ khóa `sealed` cho một phương thức (method) hoặc thuộc tính (property) đang override một phương thức/thuộc tính `virtual` trong lớp cơ sở (base class). Điều này cho phép bạn vẫn cho phép các lớp khác kế thừa từ lớp của bạn, nhưng ngăn chúng override một số phương thức/thuộc tính `virtual` cụ thể.
- Bạn không thể áp dụng `sealed` chung với `abstract` khi khai báo lớp, vì bạn buộc phải cho phép thừa kế từ lớp `abstract` mới có thể dùng được.

---

## 56. `short`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số nguyên 16-bit có dấu.

```csharp
short s = 10;
```

---

## 57. `sizeof`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Trả kích thước (byte) của kiểu.

```csharp
int size = sizeof(int); // 4
```

---

## 58. `stackalloc`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Cấp phát mảng trên stack, thường dùng với `Span<T>` hoặc pointer.

```csharp
Span<int> span = stackalloc int[100];
```

---

## 59. `static`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Thành viên/kiểu thuộc về type, không thuộc instance.

```csharp
public static class MathHelper
{
    public static int Square(int x) => x * x;
}
```

---

## 60. `string`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Chuỗi Unicode immutable (alias `System.String`).

```csharp
string s = "Hello";
s += " world"; // tạo string mới
```

---

## 61. `struct`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Khai báo value type tùy biến.

```csharp
public readonly struct Point
{
    public int X { get; }
    public int Y { get; }
    public Point(int x, int y) => (X, Y) = (x, y);
}
```

---

## 62. `switch`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Câu lệnh rẽ nhánh nhiều nhánh; C# 8+ có thêm switch expression.

```csharp
switch (day)
{
    case DayOfWeek.Monday:
        Console.WriteLine("Start");
        break;
    default:
        Console.WriteLine("Other");
        break;
}
```

---

## 63. `this`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Tham chiếu tới instance hiện tại; trong extension method đứng trước tham số đầu tiên.

```csharp
public class Person
{
    public string Name { get; set; } = "";
    public void Introduce()
        => Console.WriteLine($"I'm {this.Name}");
}

public static class StringExtensions
{
    public static bool IsEmpty(this string? value)
        => string.IsNullOrEmpty(value);
}
```

---

## 64. `throw`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Ném exception.

```csharp
if (id <= 0)
    throw new ArgumentOutOfRangeException(nameof(id));
```

---

## 65. `true`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Hằng Boolean `true`.

```csharp
while (true)
{
    if (ShouldStop()) break;
}
```

---

## 66. `try`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Bắt đầu khối có xử lý ngoại lệ (`catch`, `finally`).

```csharp
try
{
    DoWork();
}
catch (Exception ex)
{
    Log(ex);
}
```

---

## 67. `typeof`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Lấy `System.Type` của một kiểu.

```csharp
Type t1 = typeof(string);
Type t2 = typeof(List<int>);
```

---

## 68. `uint`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số nguyên 32-bit không dấu; không CLS-compliant.

```csharp
uint u = 10u;
```

---

## 69. `ulong`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số nguyên 64-bit không dấu.

```csharp
ulong u = 10UL;
```

---

## 70. `unchecked`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Tắt kiểm tra overflow.

```csharp
unchecked
{
    int x = int.MaxValue + 1; // wrap, không ném exception
}
```

---

## 71. `unsafe`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Cho phép dùng pointer, `stackalloc`, `fixed` – giống C/C++ style.

```csharp
unsafe void Foo(int* p)
{
    *p = 42;
}
```

---

## 72. `ushort`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Số nguyên 16-bit không dấu.

```csharp
ushort u = 10;
```

---

## 73. `using`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:**  
  - Import namespace (`using System;`).  
  - Quản lý lifetime tài nguyên (`using var` hoặc `using (...) { ... }`).

```csharp
using System;

using var stream = File.OpenRead("data.txt");
// dùng stream...
```

---

## 74. `virtual`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Cho phép member được override trong lớp con.

```csharp
public class Base
{
    public virtual void Do() { }
}
```

---

## 75. `void`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Kiểu trả về “không có gì”.

```csharp
void Log(string message) => Console.WriteLine(message);
```

---

## 76. `volatile`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Field `volatile` đảm bảo read/write luôn đi thẳng bộ nhớ, cải thiện visibility giữa thread.

```csharp
public volatile bool _stopped;
```

---

## 77. `while`

- **Loại:** reserved · **C#:** 1.0  
- **Mục đích:** Vòng lặp kiểm tra điều kiện trước mỗi lần lặp.

```csharp
int i = 0;
while (i < 10)
{
    Console.WriteLine(i++);
}
```

---

