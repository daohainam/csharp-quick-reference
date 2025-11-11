# Preprocessor directives

Preprocessor directives ảnh hưởng tới **quá trình biên dịch**: bật/tắt code, cảnh báo, nullable, v.v.  
Phần lớn có từ C# 1.0; riêng `#nullable` từ C# 8.0.

---

## Mục lục

- [Preprocessor directives](#preprocessor-directives)
  - [Mục lục](#mục-lục)
    - [1. `#define` – Định nghĩa symbol điều kiện](#1-define--định-nghĩa-symbol-điều-kiện)
    - [2. `#undef` – Hủy symbol](#2-undef--hủy-symbol)
    - [3. `#if`, `#elif`, `#else`, `#endif` – Biên dịch có điều kiện](#3-if-elif-else-endif--biên-dịch-có-điều-kiện)
    - [4. `#warning` – Sinh cảnh báo](#4-warning--sinh-cảnh-báo)
    - [5. `#error` – Sinh lỗi biên dịch](#5-error--sinh-lỗi-biên-dịch)
    - [6. `#line` – Điều khiển số dòng \& file name](#6-line--điều-khiển-số-dòng--file-name)
    - [7. `#region` / `#endregion` – Gom nhóm code](#7-region--endregion--gom-nhóm-code)
    - [8. `#pragma` (thường dùng `#pragma warning`)](#8-pragma-thường-dùng-pragma-warning)
    - [9. `#nullable` – Nullable reference types](#9-nullable--nullable-reference-types)

---

### 1. `#define` – Định nghĩa symbol điều kiện

**Mục đích:** Định nghĩa symbol dùng trong `#if`.  
**Phiên bản C#:** 1.0  

```csharp
#define DEBUG

class Program
{
    static void Main()
    {
#if DEBUG
        Console.WriteLine("Running in DEBUG mode");
#endif
    }
}
```

**Ghi chú:**

- Thường nên định nghĩa qua cấu hình project/build hơn là viết trực tiếp trong code.

---

### 2. `#undef` – Hủy symbol

**Mục đích:** Hủy định nghĩa symbol trước đó.  
**Phiên bản C#:** 1.0  

```csharp
#define FEATURE_X
#undef FEATURE_X

#if FEATURE_X
    // không biên dịch
#endif
```

---

### 3. `#if`, `#elif`, `#else`, `#endif` – Biên dịch có điều kiện

**Mục đích:** Bao/bỏ đoạn code dựa trên symbol.  
**Phiên bản C#:** 1.0  

```csharp
#define DEBUG

class Program
{
    static void Main()
    {
#if DEBUG
        Console.WriteLine("Debug logging enabled");
#else
        Console.WriteLine("Production mode");
#endif
    }
}
```

**Ghi chú:**

- Chỉ sử dụng symbol + toán tử `&&`, `||`, `!`, `==`, `!=`.  
- Không dùng biến runtime trong `#if`.

---

### 4. `#warning` – Sinh cảnh báo

**Mục đích:** Tạo warning custom khi build.  
**Phiên bản C#:** 1.0  

```csharp
#warning TODO: Remove this legacy code after migrating to v2 API
```

---

### 5. `#error` – Sinh lỗi biên dịch

**Mục đích:** Chặn build trong cấu hình không hợp lệ.  
**Phiên bản C#:** 1.0  

```csharp
#if NETFRAMEWORK
# error This project must target .NET 8, not .NET Framework.
#endif
```

---

### 6. `#line` – Điều khiển số dòng & file name

**Mục đích:** Điều khiển line/file trong error/warning (dùng cho code generator).  
**Phiên bản C#:** 1.0  

```csharp
#line 200 "GeneratedFile.cs"
int x = "not int"; // lỗi sẽ báo ở dòng 200, file "GeneratedFile.cs"
#line default
```

---

### 7. `#region` / `#endregion` – Gom nhóm code

**Mục đích:** Tạo vùng có thể collapse/expand trong IDE.  
**Phiên bản C#:** 1.0  

```csharp
#region Public API

public class MyService
{
    public void Start() { }
    public void Stop() { }
}

#endregion
```

---

### 8. `#pragma` (thường dùng `#pragma warning`)

**Mục đích:** Điều khiển behavior compiler (đặc biệt warning).  
**Phiên bản C#:** 1.0  

```csharp
#pragma warning disable CS0168 // Biến khai báo mà không dùng
void Foo()
{
    int x;
}
#pragma warning restore CS0168
```

**Ghi chú:**

- Chỉ nên disable warning trong phạm vi nhỏ và có lý do.

---

### 9. `#nullable` – Nullable reference types

**Mục đích:** Bật/tắt **nullable context** cho file/vùng code.  
**Phiên bản C#:** 8.0  

```csharp
#nullable enable

string? maybeNull = null;   // OK
string notNull = null;      // Warning

#nullable disable
string oldStyle = null;     // Không warning
```

**Các mode phổ biến:**

- `#nullable enable` / `disable` / `restore`  
- Có thể dùng `annotations` / `warnings` để điều chỉnh chi tiết.

