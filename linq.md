# LINQ (Language Integrated Query) trong C#/.NET

**LINQ** đem cú pháp truy vấn vào C#, thống nhất cách làm việc với **tập hợp đối tượng**, **XML**, **CSDL**, **JSON**, **in-memory** và cả **stream async** (thông qua `IAsyncEnumerable<T>` + gói mở rộng).  

---

## Mục lục

- [LINQ (Language Integrated Query) trong C#/.NET](#linq-language-integrated-query-trong-cnet)
  - [Mục lục](#mục-lục)
  - [1. Tư duy LINQ \& hai cú pháp](#1-tư-duy-linq--hai-cú-pháp)
  - [2. Deferred vs Immediate execution](#2-deferred-vs-immediate-execution)
  - [3. LINQ to Objects vs IQueryable (EF/LINQ Providers)](#3-linq-to-objects-vs-iqueryable-eflinq-providers)
  - [4. Nhóm toán tử chuẩn (Standard Query Operators)](#4-nhóm-toán-tử-chuẩn-standard-query-operators)
    - [4.1 Filtering: `Where`, `OfType`](#41-filtering-where-oftype)
    - [4.2 Projection: `Select`, `SelectMany`](#42-projection-select-selectmany)
    - [4.3 Sorting: `OrderBy`, `ThenBy`, `Reverse`](#43-sorting-orderby-thenby-reverse)
    - [4.4 Grouping: `GroupBy`, `ToLookup`](#44-grouping-groupby-tolookup)
    - [4.5 Joining: `Join`, `GroupJoin`, Left Join](#45-joining-join-groupjoin-left-join)
    - [4.6 Set: `Distinct`, `Union`, `Intersect`, `Except`](#46-set-distinct-union-intersect-except)
    - [4.7 Quantifiers: `Any`, `All`, `Contains`](#47-quantifiers-any-all-contains)
    - [4.8 Element: `First`, `Single`, `Last`, `ElementAt`](#48-element-first-single-last-elementat)
    - [4.9 Partitioning: `Skip`, `Take`, `SkipWhile`, `TakeWhile`](#49-partitioning-skip-take-skipwhile-takewhile)
    - [4.10 Aggregation: `Count`, `Sum`, `Min`, `Max`, `Average`, `Aggregate`](#410-aggregation-count-sum-min-max-average-aggregate)
    - [4.11 Generation/Conversion: `Range`, `Repeat`, `Empty`, `ToList`, `ToArray`, `ToDictionary`, `ToHashSet`…](#411-generationconversion-range-repeat-empty-tolist-toarray-todictionary-tohashset)
    - [4.12 `Zip`, `Chunk`, `Append/Prepend`, `SequenceEqual`, `DefaultIfEmpty`](#412-zip-chunk-appendprepend-sequenceequal-defaultifempty)
  - [5. Query syntax ↔ method syntax (bảng quy chiếu)](#5-query-syntax--method-syntax-bảng-quy-chiếu)
  - [6. IQueryable \& biểu thức (Expression)](#6-iqueryable--biểu-thức-expression)
  - [7. Async LINQ \& Streams](#7-async-linq--streams)
  - [8. PLINQ (Parallel LINQ)](#8-plinq-parallel-linq)
  - [9. Custom LINQ operators (viết toán tử riêng)](#9-custom-linq-operators-viết-toán-tử-riêng)
  - [10. Best practices \& Pitfalls](#10-best-practices--pitfalls)
  - [11. Cheat sheet nhanh](#11-cheat-sheet-nhanh)

---

## 1. Tư duy LINQ & hai cú pháp

LINQ có **hai cú pháp tương đương**:

- **Method syntax** (khuyên dùng): chuỗi extension methods trên `IEnumerable<T>`/`IQueryable<T>`.
- **Query syntax**: tựa SQL; compile-time dịch sang method syntax.

```csharp
// Method syntax
var q1 = people.Where(p => p.Age >= 18)
               .OrderBy(p => p.LastName)
               .Select(p => new { p.LastName, p.FirstName });

// Query syntax (dịch tương đương)
var q2 = from p in people
         where p.Age >= 18
         orderby p.LastName
         select new { p.LastName, p.FirstName };
```

Chọn cú pháp nào?  
- **Method syntax** nhất quán, đầy đủ toán tử.  
- **Query syntax** dễ đọc với `from/where/select/group/join`; nhưng không có sẵn cho một số toán tử nâng cao.

---

## 2. Deferred vs Immediate execution

- **Deferred execution**: `Where`, `Select`, `OrderBy`, `GroupBy`, `Join`… chỉ **xây dựng pipeline**; chưa chạy cho đến khi **enumerate** (`foreach`, `ToList`, `Count`, …).  
- **Immediate execution**: `ToList`, `ToArray`, `ToDictionary`, `Count`, `Sum`, `First`… **kích hoạt** thực thi.

```csharp
var q = numbers.Where(x => x % 2 == 0); // chưa chạy

// chạy tại đây (mỗi lần duyệt lại chạy lại):
foreach (var n in q) Console.WriteLine(n);

var list = q.ToList(); // chạy và materialize một lần
```

**Lưu ý**: deferred giúp **lazy**, nhưng dễ gây **multiple enumeration** nếu duyệt nhiều lần. Dùng `ToList()` khi cần cố định kết quả và tránh lặp lại tính toán.

---

## 3. LINQ to Objects vs IQueryable (EF/LINQ Providers)

- **LINQ to Objects**: chạy trên `IEnumerable<T>` (in-memory). Lambda là **delegate** chạy trong .NET.  
- **`IQueryable<T>`**: biểu diễn truy vấn **có thể dịch** sang hệ đích (SQL, OData…). Lambda là **expression tree**.
  - **EF Core**: chỉ dịch được **tập con** toán tử/method; nếu không dịch được → (tuỳ cấu hình) ném ngoại lệ hoặc chuyển sang client-eval (nên tránh).
  - Tránh gọi **method tuỳ ý** trong predicate/select vì **không thể dịch sang SQL**.

**Quy tắc vàng**: Với EF, cố gắng giữ toàn bộ truy vấn **trên server** trước khi materialize (`ToListAsync`). Dùng `AsNoTracking()` nếu chỉ đọc để tối ưu.

---

## 4. Nhóm toán tử chuẩn (Standard Query Operators)

### 4.1 Filtering: `Where`, `OfType`

```csharp
var adults = people.Where(p => p.Age >= 18);
var stringsOnly = objects.OfType<string>(); // bỏ phần tử không đúng kiểu
```

### 4.2 Projection: `Select`, `SelectMany`

```csharp
var names = people.Select(p => $"{p.FirstName} {p.LastName}");

var allTags = posts.SelectMany(p => p.Tags); // flatten
// cross join:
var pairs = from x in xs
            from y in ys
            select (x, y);
```

### 4.3 Sorting: `OrderBy`, `ThenBy`, `Reverse`

```csharp
var ordered = people.OrderBy(p => p.LastName)
                    .ThenBy(p => p.FirstName);

var desc = people.OrderByDescending(p => p.Age);
```

**`OrderBy` ổn định**: phần tử bằng nhau giữ thứ tự ban đầu.

### 4.4 Grouping: `GroupBy`, `ToLookup`

```csharp
var groups = people.GroupBy(p => p.City); // IEnumerable<IGrouping<string,Person>>

ILookup<string, Person> byCity = people.ToLookup(p => p.City);
var inHanoi = byCity["Hanoi"]; // fast lookups theo key
```

**`GroupBy`** deferred; **`ToLookup`** materialize ngay.

### 4.5 Joining: `Join`, `GroupJoin`, Left Join

```csharp
// Inner join
var q = from c in customers
        join o in orders on c.Id equals o.CustomerId
        select new { c.Name, o.Id };

// Group join (customers + collection orders)
var q2 = from c in customers
         join o in orders on c.Id equals o.CustomerId into g
         select new { c, Orders = g };

// Left join = group join + DefaultIfEmpty
var left = from c in customers
           join o in orders on c.Id equals o.CustomerId into g
           from o in g.DefaultIfEmpty()
           select new { c, o }; // o có thể null
```

**Composite key**: dùng **anonymous type** hoặc **tuple**:

```csharp
join o in orders on new { c.Id, c.Region } equals new { o.CustomerId, o.Region }
```

### 4.6 Set: `Distinct`, `Union`, `Intersect`, `Except`

```csharp
var unique = items.Distinct(comparer); // truyền comparer nếu cần
var union = a.Union(b);
var inter = a.Intersect(b);
var diff  = a.Except(b);
```

### 4.7 Quantifiers: `Any`, `All`, `Contains`

```csharp
bool anyAdult = people.Any(p => p.Age >= 18);
bool allAdult = people.All(p => p.Age >= 18);
bool has42 = numbers.Contains(42);
```

### 4.8 Element: `First`, `Single`, `Last`, `ElementAt`

```csharp
var first = numbers.First();               // throw nếu rỗng
var firstOr = numbers.FirstOrDefault();    // default(T) nếu rỗng

var only = numbers.Single(n => n == 5);    // throw nếu != 1 phần tử phù hợp
var onlyOr = numbers.SingleOrDefault();
```

### 4.9 Partitioning: `Skip`, `Take`, `SkipWhile`, `TakeWhile`

```csharp
var page = items.Skip((pageIndex-1)*pageSize).Take(pageSize);
var untilNeg = numbers.TakeWhile(n => n >= 0);
```

### 4.10 Aggregation: `Count`, `Sum`, `Min`, `Max`, `Average`, `Aggregate`

```csharp
int count = items.Count();
int sum   = numbers.Sum();
var total = numbers.Aggregate(0, (acc, x) => acc + x);
```

### 4.11 Generation/Conversion: `Range`, `Repeat`, `Empty`, `ToList`, `ToArray`, `ToDictionary`, `ToHashSet`…

```csharp
var r = Enumerable.Range(1, 5);      // 1..5
var rep = Enumerable.Repeat("A", 3); // A A A
var empty = Enumerable.Empty<int>();

var list = q.ToList();
var dict = people.ToDictionary(p => p.Id); // chú ý key trùng
var set  = items.ToHashSet(StringComparer.OrdinalIgnoreCase);
```

### 4.12 `Zip`, `Chunk`, `Append/Prepend`, `SequenceEqual`, `DefaultIfEmpty`

```csharp
var zipped = xs.Zip(ys, (x,y) => (x,y)); // ghép 2 dãy theo vị trí
var chunks = numbers.Chunk(100);         // tách khối 100 phần tử
var withHead = seq.Prepend(head);
bool same = seq1.SequenceEqual(seq2);
var withDefault = seq.DefaultIfEmpty(0); // nếu rỗng → có 1 phần tử 0
```

---

## 5. Query syntax ↔ method syntax (bảng quy chiếu)

| Query syntax | Method syntax |
|---|---|
| `from x in xs select x` | `xs.Select(x => x)` |
| `from x in xs where P(x) select x` | `xs.Where(x => P(x))` |
| `from x in xs orderby x.Key select x` | `xs.OrderBy(x => x.Key)` |
| `from x in xs orderby x.A, x.B descending select x` | `xs.OrderBy(x => x.A).ThenByDescending(x => x.B)` |
| `from x in xs group x by k` | `xs.GroupBy(x => k)` |
| `from x in xs join y in ys on x.K equals y.K select ...` | `xs.Join(ys, x => x.K, y => y.K, (x,y) => ...)` |
| `from x in xs from y in ys select ...` | `xs.SelectMany(x => ys, (x,y) => ...)` |
| `let t = expr select ...` | (giới thiệu biến trung gian → lồng `Select`) |
| `into g ...` | tiếp tục với kết quả `GroupJoin`/`group` |

---

## 6. IQueryable & biểu thức (Expression)

- `IQueryable<T>` mở rộng `IEnumerable<T>` bằng **`Expression`** để provider dịch sang ngôn ngữ đích.  
- `AsQueryable()` biến `IEnumerable` thành `IQueryable` (nhưng không tự có provider).  
- Trong EF Core:
  - Tránh phương thức **không thể dịch** (`Regex.IsMatch`, custom helpers…) trong query.  
  - Tránh **materialize sớm** (ví dụ `ToList()` giữa chừng) nếu muốn server lọc/sort.  
  - Dùng `AsNoTracking()` cho truy vấn chỉ-đọc; `AsSplitQuery()` (khi cần tách include lớn).

---

## 7. Async LINQ & Streams

- **Enumerable async**: dùng `IAsyncEnumerable<T>` + `await foreach`.  
- **Toán tử async cho `IAsyncEnumerable<T>`**: cần gói mở rộng (ví dụ `System.Linq.Async`) để có `WhereAwait`, `SelectAwait`, `ToListAsync`, `FirstOrDefaultAsync` trên async streams.  
- **EF Core**: cung cấp **`ToListAsync`, `SingleAsync`, `AnyAsync`…** trên `IQueryable` để thực thi async với DB.

```csharp
await foreach (var line in ReadLinesAsync(path).Where(x => x.Length > 0))
    Console.WriteLine(line);

// EF Core
var users = await db.Users.Where(u => u.Active).ToListAsync();
```

---

## 8. PLINQ (Parallel LINQ)

- `AsParallel()` chạy LINQ trên nhiều core. Dùng cho **tác vụ CPU-bound** thuần (không I/O).

```csharp
var result = data.AsParallel()
                 .WithDegreeOfParallelism(Environment.ProcessorCount)
                 .AsOrdered()                 // nếu cần giữ thứ tự
                 .Where(ComputeHeavy)         // thuần, không side-effects
                 .Select(Transform)
                 .ToList();
```

**Cảnh báo**: Side-effects, cập nhật shared state → cần đồng bộ hoặc tránh. Không áp dụng cho LINQ to SQL/EF.

---

## 9. Custom LINQ operators (viết toán tử riêng)

- LINQ to Objects dựa vào **extension methods** trả `IEnumerable<T>` (iterator + `yield return`).  
- Bạn có thể viết toán tử tuỳ biến:

```csharp
public static class LinqEx
{
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T?> source)
        where T : struct
    {
        foreach (var x in source)
            if (x.HasValue) yield return x.Value;
    }

    public static IEnumerable<T> DistinctBy<T, TKey>(this IEnumerable<T> src, Func<T,TKey> keySelector)
    {
        var seen = new HashSet<TKey>();
        foreach (var x in src)
            if (seen.Add(keySelector(x)))
                yield return x;
    }
}
```

- Gợi ý: nhiều operator hữu ích đã có sẵn trong .NET hiện đại (`DistinctBy`, `MaxBy`, `MinBy`, `Chunk`, …).

---

## 10. Best practices & Pitfalls

1. **Hiểu deferred execution**: tránh multiple enumeration vô tình; materialize (`ToList`) khi cần snapshot.  
2. **Null-safety & rỗng**: `First()` ném lỗi nếu rỗng; dùng `FirstOrDefault()` + xử lý `default`.  
3. **Distinct/Dictionary với đối tượng phức tạp**: truyền `IEqualityComparer<T>` hoặc override equality.  
4. **Chuẩn so sánh chuỗi**: dùng `StringComparer.Ordinal/OrdinalIgnoreCase` rõ ràng culture.  
5. **Hiệu năng**: LINQ rõ ràng nhưng có overhead; trong hot-path cân nhắc `for`/`Span<T>`.  
6. **EF/IQueryable**: đừng chèn method không dịch được; tránh client-eval; ghép `Where`/`Select` trước khi `ToListAsync`.  
7. **Vòng lặp và biến captured**: cẩn thận **closure** trong lambda (đặc biệt khi tạo delegates trong vòng `for`).  
8. **Exceptions**: `Single()` rất nghiêm – chỉ dùng khi chắc chắn có **đ đúng 1** phần tử; nếu nghi ngờ dùng `FirstOrDefault()` + kiểm tra.  
9. **PLINQ**: chỉ cho CPU-bound, không I/O, tránh side-effects.  
10. **Compose nhỏ, test dễ**: chia truy vấn thành biến trung gian đặt tên bằng `var step1 = ...; var step2 = step1.Where(...);`.

---

## 11. Cheat sheet nhanh

```csharp
// Top N theo điểm giảm dần, cùng điểm thì theo tên tăng dần
var top = students
    .OrderByDescending(s => s.Score)
    .ThenBy(s => s.Name, StringComparer.Ordinal)
    .Take(10)
    .ToList();

// Left join customers - orders và tính tổng
var totals = from c in customers
             join o in orders on c.Id equals o.CustomerId into grp
             from o in grp.DefaultIfEmpty()
             group o by c into g
             select new {
                Customer = g.Key,
                Total = g.Where(x => x != null).Sum(x => x!.Amount)
             };

// Grouping tháng-năm, đếm số đơn
var monthly = orders
    .GroupBy(o => new { o.Date.Year, o.Date.Month })
    .Select(g => new { g.Key.Year, g.Key.Month, Count = g.Count() })
    .OrderBy(x => x.Year).ThenBy(x => x.Month);

// Distinct theo khoá tuỳ biến
var distinct = users.DistinctBy(u => u.Email.ToLowerInvariant());

// Chia trang
IEnumerable<T> Page<T>(IEnumerable<T> src, int page, int size)
    => src.Skip((page-1)*size).Take(size);
```

---

**Kết luận**: LINQ giúp code **khai báo, ngắn gọn, dễ đọc**, đồng thời đủ mạnh để **kết hợp** với EF/Providers, **song song** (PLINQ), và **async streams**. Nắm chắc **toán tử chuẩn**, hiểu **deferred vs immediate**, phân biệt **IEnumerable/IQueryable**, và áp dụng **best practices** là chìa khoá để có truy vấn vừa **đúng** vừa **nhanh**.
