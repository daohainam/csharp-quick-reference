# Delegate & Lambda

> Lưu ý: đọc thêm chương *Events*, *LINQ*, *Type System*, *Methods*, *Async*.

---

## Mục lục

1. [Delegate là gì?](#1-delegate-là-gì)  
2. [Khai báo & sử dụng delegate](#2-khai-báo--sử-dụng-delegate)  
3. [Delegate dựng sẵn: `Action<>`, `Func<>`, `Predicate<T>`](#3-delegate-dựng-sẵn-action-func-predicatet)  
4. [Multicast delegate & Invocation List](#4-multicast-delegate--invocation-list)  
5. [Method group & overload resolution](#5-method-group--overload-resolution)  
6. [Lambda expressions](#6-lambda-expressions)  
   6.1 [Expression lambda vs Statement lambda](#61-expression-lambda-vs-statement-lambda) · 6.2 [Anonymous methods](#62-anonymous-methods) · 6.3 [Async lambda](#63-async-lambda)  
7. [Closure & Capturing](#7-closure--capturing)  
8. [Variance trong delegate (`in`/`out`)](#8-variance-trong-delegate-inout)  
9. [Ref/Out/In trong delegate & ref return](#9-refoutin-trong-delegate--ref-return)  
10. [Expression Trees (`Expression<TDelegate>`)](#10-expression-trees-expressiontdelegate)  
11. [Local functions vs Lambda](#11-local-functions-vs-lambda)  
12. [Delegates & Events (ôn nhanh sự khác biệt)](#12-delegates--events-ôn-nhanh-sự-khác-biệt)  
13. [Interop & Function Pointers](#13-interop--function-pointers)  
14. [Hiệu năng & Best Practices](#14-hiệu-năng--best-practices)  
15. [Cheat sheet & ví dụ tổng hợp](#15-cheat-sheet--ví-dụ-tổng-hợp)

---

## 1. Delegate là gì?

- **Delegate** là *kiểu tham chiếu tới phương thức* (type-safe function pointer).  
- Một biến delegate trỏ tới **phương thức** (static/instance) có **chữ ký** tương ứng.  
- Dùng làm *callback*, *event handler*, *pipeline*…

```csharp
public delegate int Transformer(int x);

int Square(int x) => x * x;

Transformer t = Square;
int r = t(5); // hoặc t.Invoke(5) → 25
```

---

## 2. Khai báo & sử dụng delegate

### 2.1 Khai báo custom delegate

```csharp
public delegate bool Filter<in T>(T item); // có thể khai báo variance
```

- Delegate là **sealed class** kế thừa `System.MulticastDelegate`.  
- Thuộc tính hữu ích: `.Target` (instance được capture), `.Method` (MethodInfo).

### 2.2 Gán & gọi

```csharp
Filter<string> f = s => !string.IsNullOrWhiteSpace(s);
bool ok = f("hello");
```

### 2.3 So sánh & bằng nhau

- Hai delegate bằng nhau khi **cùng Target + Method** (với multicast: cùng chuỗi invocation).  
- `delegateA == delegateB` kiểm tra semantically equal.

---

## 3. Delegate dựng sẵn: `Action<>`, `Func<>`, `Predicate<T>`

- `Action` không trả về: `Action`, `Action<T1>`, …, `Action<T1,...,T16>`
- `Func<..., TResult>`: tham số bất kỳ + **kết quả là phần tử cuối**.
- `Predicate<T>` tương đương `Func<T, bool>` (di sản, dùng trong `List<T>.Find`…)

```csharp
Action<string> log = Console.WriteLine;
Func<int,int,int> add = (a,b) => a + b;
Predicate<string> nonEmpty = s => !string.IsNullOrEmpty(s);
```

> Khuyên dùng `Func`/`Action` thay vì tạo delegate mới, trừ khi bạn cần **ý nghĩa tên** rõ ràng hoặc **variance** khác.  

---

## 4. Multicast delegate & Invocation List

- Delegate có thể **multicast** (kết hợp nhiều handler): sử dụng `+`, `-` hoặc `+=`, `-=`.
- Khi gọi, **thứ tự thực thi** theo thứ tự thêm vào (FIFO).  
- Nếu một handler ném exception → lời gọi **dừng tại đó**. Cách an toàn: iterate `GetInvocationList()` và `try/catch` từng handler.

```csharp
Action all = null!;
all += () => Console.WriteLine("A");
all += () => throw new Exception("Boom");
all += () => Console.WriteLine("B"); // không tới nếu không catch

foreach (Action h in all.GetInvocationList())
{
    try { h(); }
    catch (Exception ex) { Console.Error.WriteLine(ex.Message); }
}
```

---

## 5. Method group & overload resolution

**Method group** có thể gán trực tiếp cho delegate nếu chữ ký phù hợp:

```csharp
int Parse(string s) => int.Parse(s);
Func<string,int> f = Parse; // method group conversion
```

- Khi có **nhiều overload**, compiler chọn theo *overload resolution*.  
- Nếu mơ hồ, cần **cast** đích rõ ràng: `(Func<string,int>)Parse`.

---

## 6. Lambda expressions

```csharp
// Lambda cơ bản
(int x) => x * x
x => x * x              // suy kiểu
(string s) => { Console.WriteLine(s); return s.Length; } // statement lambda
```

> **C# 9**: *static lambda* — `static x => ...` không capture được biến ngoài.  
> **C# 10**: *Lambda improvements* — **type tự nhiên** (natural type), có thể **chỉ định kiểu trả về** và **gán attribute** cho lambda.  
> **C# 12**: hỗ trợ **default parameter** cho lambda (phiên bản mới).

### 6.1 Expression lambda vs Statement lambda

- **Expression lambda**: thân là *biểu thức* → trả về kết quả expression.  
- **Statement lambda**: thân là *khối lệnh* `{ ... }` → có `return`/`await`/`try-catch`.

### 6.2 Anonymous methods

Cú pháp cũ (C# 2.0), vẫn hữu dụng khi cần `goto`, nhiều `return`, hoặc không muốn khai báo tham số:

```csharp
delegate(int x) { return x * x; }
```

### 6.3 Async lambda

- Gắn từ khóa `async`: `async x => { await ...; }`  
- Kiểu trả về: `Task` / `Task<T>` / **hiếm khi** `void` (chỉ cho event handler).  
- Không thể vừa `async` vừa `yield` (iterator).

```csharp
Func<Task<int>> f = async () => { await Task.Delay(10); return 42; };
```

---

## 7. Closure & Capturing

- Lambda/anonymous method có thể **capture biến ngoài phạm vi** (tạo *closure*).  
- Capture **biến**, không capture **giá trị** → thay đổi sau đó sẽ phản ánh trong lambda.  
- Compiler sinh **class closure ẩn**, lưu các biến captured trên **heap** → có thể gây **cấp phát**.

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++)
{
    int copy = i;                 // ✅ dùng bản sao để tránh bẫy
    actions.Add(() => Console.WriteLine(copy));
}
foreach (var a in actions) a();   // 0 1 2
```

**Pitfalls & lưu ý**:

- `for` vẫn dùng **một biến** lặp → cần `copy` như trên.  
- `foreach` hiện đại **mỗi vòng có biến riêng** (C# 5+), nhưng khi nghi ngờ, vẫn có thể `var local = item;` rồi capture `local`.  
- Không thể capture `ref struct`/`ref` local.  
- **Static lambda**: `static () => ...` **không capture** gì → không cấp phát closure.

---

## 8. Variance trong delegate (`in`/`out`)

Bạn có thể khai báo delegate generic **covariant/contravariant**:

```csharp
public delegate TResult Factory<out TResult>();          // covariant kết quả
public delegate void Consumer<in T>(T item);             // contravariant tham số
```

- Tương tự như `Func<in ..., out TResult>` và `Action<in ...>`.  
- Cho phép *thay thế kiểu* thuận tiện khi kế thừa (ví dụ `IAnimal` ↔ `Cat`).

---

## 9. Ref/Out/In trong delegate & ref return

- Delegate có thể nhận **`ref`/`out`/`in`** parameters, nhưng phải **khớp đúng** ở method gán vào.  
- Delegate có thể **trả về `ref`**:

```csharp
public delegate ref int RefPicker(int[] data);

static ref int First(ref int a) => ref a; // ví dụ ref return

RefPicker pick = (int[] arr) => ref arr[0];
ref int r = ref pick(new[] { 10, 20 });
r = 99;  // thay đổi phần tử mảng
```

- **Không** thể `async` với `ref` return.  
- Cần thận trọng vòng đời đối tượng được trả `ref` (không trả ref tới local).

---

## 10. Expression Trees (`Expression<TDelegate>`)

- **Lambda** có thể “nâng” thành **biểu thức** dùng trong metaprogramming/ORM:

```csharp
using System.Linq.Expressions;

Expression<Func<int,int>> expr = x => x * x;
Func<int,int> compiled = expr.Compile();
int r = compiled(5); // 25
```

- **EF Core** & providers dịch `Expression` sang SQL/DSL → **chỉ một tập con** biểu thức được hỗ trợ.  
- Nếu gọi method không dịch được → fail hoặc client-eval (tránh).  
- Sửa/chắp biểu thức động: dùng `ExpressionVisitor` hoặc thư viện hỗ trợ.

---

## 11. Local functions vs Lambda

**Local function**: hàm cục bộ *đặt tên được*, nằm trong thân method (C# 7+).

```csharp
int Sum(int[] a)
{
    int Impl(int i, int acc)
        => i == a.Length ? acc : Impl(i+1, acc + a[i]);
    return Impl(0, 0);
}
```

So với lambda:

- **Hiệu năng**: local function thường **ít cấp phát** hơn khi không capture; hỗ trợ `ref/out`, `in`, iterator `yield`.  
- **Đọc/Debug**: tên rõ ràng, gọi đệ quy dễ.  
- Lambda **ngắn gọn** cho callback/simple mapping; có thể gắn `async` trực tiếp.

> Nhiều trường hợp local function là lựa chọn **tối ưu** thay vì lambda capture.

---

## 12. Delegates & Events (ôn nhanh sự khác biệt)

- **Delegate**: kiểu tham chiếu phương thức, *có thể* gọi trực tiếp.  
- **Event**: **bao bọc** delegate, **giới hạn** truy cập từ ngoài chỉ `+=`/`-=`; *chỉ* publisher mới được `Invoke`.

```csharp
public class Notifier
{
    public event EventHandler<string>? Message;
    protected virtual void OnMessage(string m)
        => Message?.Invoke(this, m);
}
```

- Tránh rò rỉ bộ nhớ: luôn **hủy đăng ký** khi không còn dùng; hoặc dùng *weak event pattern*.

---

## 13. Interop & Function Pointers

- Interop P/Invoke: khai báo delegate phù hợp **gọi ngược (callback)**.  
- **Marshal**: `Marshal.GetDelegateForFunctionPointer` / `GetFunctionPointerForDelegate`.  
- **Function pointers** (*unsafe*, C# 9+): `delegate*<int, void>` — nhanh hơn, nhưng mất nhiều an toàn.

```csharp
unsafe delegate*<int, void> fp;
```

> Dùng khi thật cần hiệu năng/interop thấp-level; còn lại cứ dùng delegate .NET.

---

## 14. Hiệu năng & Best Practices

1. **Ưu tiên method group** khi không cần capture: `DoSomethingAsync` thay vì `x => DoSomethingAsync(x)` để tránh closure.  
2. **Static lambda** nếu có thể: `static x => Transform(x)` — không capture, không cấp phát.  
3. **Cache delegate dùng lặp lại** (đặc biệt trong loop/hot-path).  
4. **Tránh lạm dụng LINQ/lambda** trong hot-path → cân nhắc `for`/`Span<T>`.  
5. **Unsubscribe** event để tránh rò rỉ.  
6. **Expression Trees** chỉ cho nơi cần dịch/biến đổi; không thay thế runtime delegate thông thường.  
7. **`async void`** chỉ dùng cho event; mọi async lambda còn lại trả `Task`/`Task<T>`.  
8. **Ref safety**: cẩn thận với `ref/out/in` trong delegate; không trả `ref` tới local.  
9. **Recursion**: lambda recursion cần tự tham chiếu (khó); local function thuận tiện hơn.

---

## 15. Cheat sheet & ví dụ tổng hợp

### 15.1 Callback đơn giản

```csharp
void Process<T>(IEnumerable<T> src, Func<T,bool> predicate, Action<T> onHit)
{
    foreach (var x in src)
        if (predicate(x)) onHit(x);
}

Process(new[]{1,2,3,4}, x => x%2==0, Console.WriteLine);
// output: 2 4
```

### 15.2 Ghép pipeline không cấp phát (static lambda)

```csharp
var data = Enumerable.Range(1, 1_000_000);
var sum = data.Where(static x => (x & 1) == 0)     // static: không capture
              .Select(static x => x * 2)
              .Sum();
```

### 15.3 Xử lý multicast an toàn

```csharp
public static void SafeInvoke<T>(this EventHandler<T>? evt, object sender, T args)
{
    if (evt is null) return;
    foreach (EventHandler<T> h in evt.GetInvocationList())
        try { h(sender, args); } catch (Exception ex) { /* log */ }
}
```

### 15.4 Biểu thức động cho EF (Expression Tree)

```csharp
Expression<Func<User,bool>> Build(string? q)
{
    q ??= "";
    return u => u.Name.Contains(q) || u.Email.Contains(q);
}
var expr = Build("alice");
var users = await db.Users.Where(expr).ToListAsync();
```

### 15.5 Local function vs Lambda capture

```csharp
int SumSquares(ReadOnlySpan<int> a)
{
    // Local function: không capture → không cấp phát
    static int Sqr(int x) => x * x;

    var s = 0;
    foreach (var x in a) s += Sqr(x);
    return s;
}
```

