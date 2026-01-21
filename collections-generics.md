# Collections & Generics

---

## Mục lục

- [Collections \& Generics](#collections--generics)
  - [Mục lục](#mục-lục)
  - [1. Các interface cốt lõi của Collections](#1-các-interface-cốt-lõi-của-collections)
  - [2. Nhóm Collections thường dùng (mutable)](#2-nhóm-collections-thường-dùng-mutable)
    - [2.1 `List<T>`](#21-listt)
    - [2.2 `LinkedList<T>`](#22-linkedlistt)
    - [2.3 `Queue<T>`](#23-queuet)
    - [2.4 `Stack<T>`](#24-stackt)
    - [2.5 `Dictionary<TKey,TValue>`](#25-dictionarytkeytvalue)
    - [2.6 `SortedDictionary<TKey,TValue>` vs `SortedList<TKey,TValue>`](#26-sorteddictionarytkeytvalue-vs-sortedlisttkeytvalue)
    - [2.7 `HashSet<T>` / `SortedSet<T>`](#27-hashsett--sortedsett)
  - [3. Collections bất biến (`System.Collections.Immutable`)](#3-collections-bất-biến-systemcollectionsimmutable)
  - [4. Collections đồng thời (thread-safe)](#4-collections-đồng-thời-thread-safe)
  - [5. Readonly \& View: `ReadOnlyCollection<T>`, `IReadOnlyList<T>`…](#5-readonly--view-readonlycollectiont-ireadonlylistt)
  - [6. Mảng \& các tiện ích hiệu năng: `Array`, `ArrayPool<T>`, `Span<T>`, `Memory<T>`](#6-mảng--các-tiện-ích-hiệu-năng-array-arraypoolt-spant-memoryt)
  - [7. So sánh \& băm: `IEquatable<T>`, `IComparable<T>`, `IEqualityComparer<T>`…](#7-so-sánh--băm-iequatablet-icomparablet-iequalitycomparert)
  - [8. Hiệu năng \& best practices khi dùng collections](#8-hiệu-năng--best-practices-khi-dùng-collections)
  - [9. Generics nâng cao](#9-generics-nâng-cao)
    - [9.1 Ràng buộc (`where`) \& mẫu thiết kế](#91-ràng-buộc-where--mẫu-thiết-kế)
    - [9.2 Phương sai (variance): `out`/`in`](#92-phương-sai-variance-outin)
    - [9.3 Generic math \& `static abstract` members](#93-generic-math--static-abstract-members)
    - [9.4 Comparer/Equality custom cho collections](#94-comparerequality-custom-cho-collections)
  - [10. Cheat sheet chọn cấu trúc dữ liệu](#10-cheat-sheet-chọn-cấu-trúc-dữ-liệu)
    - [Ví dụ tổng hợp](#ví-dụ-tổng-hợp)

---

## 1. Các interface cốt lõi của Collections

```
IEnumerable<T>
└─ IEnumerator<T> (GetEnumerator())
ICollection<T> : IEnumerable<T> (Count, Add/Remove/Contains, CopyTo)
└─ IList<T> (indexer, Insert/RemoveAt)          // list dạng mảng
└─ ISet<T>  (hợp, giao, hiệu)                   // tập hợp
IDictionary<TKey,TValue> (Add, TryGetValue, Keys, Values)
IReadOnlyCollection<T> / IReadOnlyList<T> / IReadOnlyDictionary<TKey,TValue>
```

**Nguyên tắc API**:  

- **Expose tối thiểu** cần thiết (ví dụ trả `IReadOnlyList<T>` thay vì `List<T>`).  
- **Duyệt**: mọi collection nên hỗ trợ `foreach` qua `IEnumerable<T>`.  
- **Try-pattern**: `bool TryGetValue(TKey key, out TValue value)` để tránh ném exception trong luồng thường.

---

## 2. Nhóm Collections thường dùng (mutable)

### 2.1 `List<T>`

- Mảng động, tiếp giáp bộ nhớ → **O(1)** truy cập ngẫu nhiên, **Append amortized O(1)**.
- **Insert/Remove ở giữa**: O(n) do dồn phần tử.
- API đáng chú ý: `Capacity`, `EnsureCapacity`, `AddRange`, `InsertRange`, `RemoveAll`, `BinarySearch`, `Sort`, `AsReadOnly`.

```csharp
var list = new List<int>(capacity: 1024);
list.AddRange(new[] {1,2,3});
list.Sort(); // O(n log n)
int idx = list.BinarySearch(2); // yêu cầu list đã Sort
```

**Mẹo**: biết trước kích thước? → set `Capacity` để tránh **realloc** nhiều lần.

---

### 2.2 `LinkedList<T>`

- Danh sách liên kết đôi. **Insert/Remove O(1)** khi đã có node. **Tìm kiếm O(n)**.
- Không tiếp giáp bộ nhớ → cache kém; hiếm khi nhanh hơn `List<T>` trừ khi **add/remove nội bộ cực nhiều** và đã có node.

```csharp
var ll = new LinkedList<int>();
var n2 = ll.AddLast(2);
ll.AddBefore(n2, 1); // O(1)
```

---

### 2.3 `Queue<T>`

- Hàng đợi FIFO. `Enqueue`/`Dequeue` amortized **O(1)**.

```csharp
var q = new Queue<string>();
q.Enqueue("a");
var x = q.Dequeue(); // "a"
```

---

### 2.4 `Stack<T>`

- Ngăn xếp LIFO. `Push`/`Pop` amortized **O(1)**.

```csharp
var st = new Stack<int>();
st.Push(10);
int top = st.Pop();
```

---

### 2.5 `Dictionary<TKey,TValue>`

- Bảng băm. **Lookup trung bình O(1)**.  
- **Khóa** cần equality/hashing tốt (`IEquatable<T>`, `GetHashCode`).  
- Tránh `dict.ContainsKey(k)` rồi `dict[k]`: dùng `TryGetValue` **một lần tra**.

```csharp
var dict = new Dictionary<string,int>(StringComparer.Ordinal);
if (dict.TryGetValue("key", out var value))
{
    // dùng value
}
```

**Mẹo**: chọn `StringComparer.Ordinal`/`OrdinalIgnoreCase` thay vì mặc định để rõ ràng *culture*.

---

### 2.6 `SortedDictionary<TKey,TValue>` vs `SortedList<TKey,TValue>`

- `SortedDictionary` dùng **cây đỏ-đen** (lookup **O(log n)**, thêm/xóa **O(log n)** ổn định).  
- `SortedList` dùng **mảng đã sort** (lookup **O(log n)**, thêm/xóa **O(n)** do dịch phần tử) nhưng **tiêu tốn ít bộ nhớ** hơn & truy cập theo **index**.

Chọn gì?

- Nhiều **insert/remove rải rác** → `SortedDictionary`.  
- Ít thay đổi, cần **truy cập theo index** hoặc bộ nhớ chặt → `SortedList`.

---

### 2.7 `HashSet<T>` / `SortedSet<T>`

- `HashSet<T>`: tập hợp không trùng; các phép **Union/Intersect/Except**.

```csharp
var a = new HashSet<int>{1,2,3};
var b = new HashSet<int>{3,4};
a.IntersectWith(b); // a = {3}
```

- `SortedSet<T>`: sắp xếp tự nhiên theo `IComparer<T>`; hỗ trợ **range view** (`GetViewBetween`).

---

## 3. Collections bất biến (`System.Collections.Immutable`)

> **Lưu ý**: `System.Collections.Immutable` là một thư viện riêng, cần cài đặt NuGet package `System.Collections.Immutable` để sử dụng.

- `ImmutableList<T>`, `ImmutableDictionary<TKey,TValue>`, `ImmutableHashSet<T>`…  
- **Mọi thao tác sinh cấu trúc mới**; bên trong dùng **persistent data structure** để chia sẻ nút → tiết kiệm bộ nhớ so với copy thô.
- Phù hợp: **đồng thời, chia sẻ giữa thread**, **state lịch sử** (time-travel), **functional style**.

```csharp
using System.Collections.Immutable;
var list = ImmutableList<int>.Empty;
var list2 = list.Add(1).Add(2); // list vẫn rỗng
```

**Builder**: `var b = list.ToBuilder(); ...; var newList = b.ToImmutable();` — tối ưu nhiều thao tác.

---

## 4. Collections đồng thời (thread-safe)

- `ConcurrentDictionary<TKey,TValue>`: tra cứu an toàn, API `GetOrAdd`, `AddOrUpdate`.
- `ConcurrentQueue<T>`, `ConcurrentStack<T>`, `ConcurrentBag<T>`: hàng đợi/ngăn xếp/túi thread-safe (không có thứ tự mạnh).
- `BlockingCollection<T>`: bọc trên concurrent collection với **bounded capacity** & blocking producers/consumers.
- `System.Threading.Channels` (thư viện riêng): **channel** tốc độ cao (producer/consumer) — tốt cho I/O pipeline.

```csharp
var cd = new ConcurrentDictionary<string,int>();
int v = cd.AddOrUpdate("k", 1, (_, old) => old + 1);
```

> **Lock thủ công** (`lock`) vẫn hữu ích cho thao tác phức tạp nhiều bước cần tính nguyên tử.

---

## 5. Readonly & View: `ReadOnlyCollection<T>`, `IReadOnlyList<T>`…

- **`ReadOnlyCollection<T>`**: *view* bất biến **trên** một `IList<T>` hiện hữu (thay đổi ở gốc sẽ **phản ánh** vào view).  
- **Interface `IReadOnlyList<T>`/`IReadOnlyDictionary<TKey,TValue>`**: hợp đồng chỉ-đọc; trả về từ API để **giấu** implement thật.

```csharp
IReadOnlyList<int> GetIds() => _ids; // _ids là List<int>
```

---

## 6. Mảng & các tiện ích hiệu năng: `Array`, `ArrayPool<T>`, `Span<T>`, `Memory<T>`

- **`Array`**: thao tác khối: `Array.Copy`, `Clear`, `BinarySearch`, `Sort`.  
- **`ArrayPool<T>`**: *rent/return* mảng để **giảm GC** trong luồng nóng.
```csharp
var pool = System.Buffers.ArrayPool<byte>.Shared;
byte[] buffer = pool.Rent(4096);
try
{
    // dùng buffer
}
finally
{
    pool.Return(buffer, clearArray: true);
}
```

- **`Span<T>` / `ReadOnlySpan<T>`** (byref-like): lát cắt không cấp phát; dùng với string (as `ReadOnlySpan<char>`), file I/O, parsing…  
- **`Memory<T>` / `ReadOnlyMemory<T>`**: tương tự `Span` nhưng **lưu trữ được** (dùng cho async, field).

**CollectionsMarshal** (nâng cao): `CollectionsMarshal.AsSpan(list)` để truy cập nội bộ `List<T>` *không an toàn phiên bản* → chỉ dùng khi hiểu rõ ràng buộc.

---

## 7. So sánh & băm: `IEquatable<T>`, `IComparable<T>`, `IEqualityComparer<T>`…

- **`IEquatable<T>`**: so sánh bằng nhau theo **giá trị** (tối ưu tránh boxing).  
- **`IComparable<T>`**: thứ tự sắp xếp.  
- **`IEqualityComparer<T>` / `IComparer<T>`**: truyền vào `Dictionary`/`HashSet`/`SortedSet` để **định nghĩa quy tắc**.

```csharp
public sealed class PersonIdComparer : IEqualityComparer<Person>
{
    public bool Equals(Person? x, Person? y) => x?.Id == y?.Id;
    public int GetHashCode(Person obj) => obj.Id.GetHashCode();
}

// Dùng
var set = new HashSet<Person>(new PersonIdComparer());
```

**`GetHashCode` chuẩn**: dùng `HashCode.Combine(a,b,...)`; đảm bảo: bằng nhau ⇒ hash bằng nhau.

**Tuples/records**: đã có equality/hashing theo giá trị; tận dụng cho key phức tạp: `Dictionary<(int,int),TValue>`.

---

## 8. Hiệu năng & best practices khi dùng collections

- **Chọn đúng cấu trúc** (xem *Cheat sheet*).  
- **Capacity**: biết trước kích thước? set `Capacity`/`EnsureCapacity` (`List<T>`, `Dictionary<,>`).  
- **Tránh cấp phát**: dùng `ArrayPool<T>`, `Span<T>`, tránh tạo iterator/closure trong hot-path.  
- **`foreach`** trên `List<T>` dùng **struct enumerator** (không cấp phát). Trên `IEnumerable<T>` “trừu tượng” có thể boxing; trong hot-path cân nhắc `for`/`Span`.  
- **`Dictionary`**: dùng `TryGetValue` thay vì `ContainsKey` + indexer; khai báo `StringComparer` phù hợp.  
- **`SortedList`** vs `SortedDictionary`**: xem mục 2.6.  
- **LINQ**: rõ ràng, ngắn; nhưng dễ **cấp phát**. Trong đường nóng → cân nhắc vòng `for`/`foreach`.  
- **Immutable**: dùng khi chia sẻ giữa thread nhiều đọc; biến đổi nhiều → dùng `Builder`.  
- **Concurrent**: thao tác đơn giản → concurrent collections; thao tác phức tạp nhiều bước → *lock*.

---

## 9. Generics nâng cao

### 9.1 Ràng buộc (`where`) & mẫu thiết kế

```csharp
public T Create<T>() where T : new() => new T();

public T Max<T>(T a, T b) where T : IComparable<T>
    => a.CompareTo(b) >= 0 ? a : b;

public interface IRepository<T> where T : class
{
    T? FindById(Guid id);
    void Add(T entity);
}
```

- Ràng buộc đặc biệt: `unmanaged`, `notnull`, `struct`, `class`, `new()`.  
- **Ràng buộc nhiều**: `where T : SomeBase, ISvc, new()`.

### 9.2 Phương sai (variance): `out`/`in`

- **Covariant `out`**: cho **output-only**. Ví dụ `IEnumerable<out T>` → `IEnumerable<string>` nạp vào nơi cần `IEnumerable<object>`.
- **Contravariant `in`**: cho **input-only**. Ví dụ `IComparer<in T>` có thể so sánh `object` cho `string`.

```csharp
IEnumerable<string> ss = new List<string>();
IEnumerable<object> oo = ss; // ok nhờ 'out T'
```

> Không áp dụng cho class/struct generic, chỉ **interface/delegate**.

### 9.3 Generic math & `static abstract` members

Từ .NET 7/C# 11: interface có **`static abstract`** cho toán học tổng quát (`System.Numerics`):

```csharp
using System.Numerics;

T Sum<T>(IEnumerable<T> xs) where T : INumber<T>
{
    T s = T.Zero;
    foreach (var x in xs) s += x;
    return s;
}
```

- Viết thuật toán số học generic **không cần** overloading thủ công từng kiểu.

### 9.4 Comparer/Equality custom cho collections

- `Dictionary<TKey,TValue>(IEqualityComparer<TKey>)`  
- `SortedSet<T>(IComparer<T>)`

```csharp
var dictCI = new Dictionary<string,int>(StringComparer.OrdinalIgnoreCase);
var setDesc = new SortedSet<int>(Comparer<int>.Create((a,b) => b.CompareTo(a)));
```

---

## 10. Cheat sheet chọn cấu trúc dữ liệu

| Bài toán | Gợi ý |
|---|---|
| Truy cập theo chỉ số, thêm cuối nhiều | `List<T>` (+ đặt `Capacity`) |
| Thêm/xóa nhiều ở giữa (đã có node) | `LinkedList<T>` |
| FIFO / LIFO | `Queue<T>` / `Stack<T>` |
| Tra cứu theo khóa | `Dictionary<TKey,TValue>` (+ `TryGetValue`, comparer phù hợp) |
| Tập hợp không trùng | `HashSet<T>` |
| Cần thứ tự sort & tra cứu | `SortedDictionary<TKey,TValue>` |
| Sort ổn định, ít cập nhật, cần index | `SortedList<TKey,TValue>` |
| Chia sẻ thread-safe, nhiều đọc | `Immutable*` collections |
| Producer/consumer tốc độ cao | `System.Threading.Channels` / `BlockingCollection<T>` |
| Giảm GC với buffer | `ArrayPool<T>`, `Span<T>`/`Memory<T>` |

---

### Ví dụ tổng hợp

```csharp
// Đếm số lần xuất hiện (case-insensitive) và xuất top N theo tần suất giảm dần
IReadOnlyList<(string Word, int Count)> TopNWords(IEnumerable<string> words, int n)
{
    var freq = new Dictionary<string,int>(StringComparer.OrdinalIgnoreCase);
    foreach (var w in words)
        freq[w] = (freq.TryGetValue(w, out var c) ? c : 0) + 1;

    // sort theo Count giảm dần, rồi theo Word tăng dần
    var list = freq.ToList();
    list.Sort((a,b) => b.Value.CompareTo(a.Value) != 0
        ? b.Value.CompareTo(a.Value)
        : StringComparer.OrdinalIgnoreCase.Compare(a.Key, b.Key));

    if (n < list.Count) list.RemoveRange(n, list.Count - n);
    return list.Select(p => (p.Key, p.Value)).ToList();
}
```

---

**Kết luận**: Nắm vững **interface cốt lõi**, chọn đúng **cấu trúc dữ liệu**, hiểu **equality/hashing**, và sử dụng **generics nâng cao** (ràng buộc, variance, generic math) sẽ giúp code C# của bạn **đúng, nhanh, và dễ bảo trì**.
