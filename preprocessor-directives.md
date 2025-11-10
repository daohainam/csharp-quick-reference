# Tham khảo nhanh C# – Preprocessor directives

## 1. Preprocessor directives trong C#

> Preprocessor directives ảnh hưởng tới **quá trình biên dịch**: bật/tắt code, cảnh báo, nullable, v.v.  
> Phần lớn có từ C# 1.0; riêng `#nullable` từ C# 8.0.

- `#define`
- `#undef`
- `#if`, `#elif`, `#else`, `#endif`
- `#warning`
- `#error`
- `#line`
- `#region`, `#endregion`
- `#pragma` (thường dùng `#pragma warning`)
- `#nullable`

---

### 2. `#define` – Định nghĩa symbol điều kiện

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

### 3. `#undef` – Hủy symbol

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

### 4. `#if`, `#elif`, `#else`, `#endif` – Biên dịch có điều kiện

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

### 5. `#warning` – Sinh cảnh báo

**Mục đích:** Tạo warning custom khi build.  
**Phiên bản C#:** 1.0  

```csharp
#warning TODO: Remove this legacy code after migrating to v2 API
```

---

### 6. `#error` – Sinh lỗi biên dịch

**Mục đích:** Chặn build trong cấu hình không hợp lệ.  
**Phiên bản C#:** 1.0  

```csharp
#if NETFRAMEWORK
# error This project must target .NET 8, not .NET Framework.
#endif
```

---

### 7. `#line` – Điều khiển số dòng & file name

**Mục đích:** Điều khiển line/file trong error/warning (dùng cho code generator).  
**Phiên bản C#:** 1.0  

```csharp
#line 200 "GeneratedFile.cs"
int x = "not int"; // lỗi sẽ báo ở dòng 200, file "GeneratedFile.cs"
#line default
```

---

### 8. `#region` / `#endregion` – Gom nhóm code

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

### 9. `#pragma` (thường dùng `#pragma warning`)

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

### 10. `#nullable` – Nullable reference types

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

---

### 11. Bảng tóm tắt preprocessor directives

| Directive                            | C#   | Mục đích chính                                  |
| ------------------------------------ | ---- | ----------------------------------------------- |
| `#define` / `#undef`                 | 1.0  | Định nghĩa / hủy symbol cho biên dịch điều kiện |
| `#if` / `#elif` / `#else` / `#endif` | 1.0  | Bao/bỏ code theo symbol                         |
| `#warning`                           | 1.0  | Sinh cảnh báo custom                            |
| `#error`                             | 1.0  | Sinh lỗi biên dịch custom                       |
| `#line`                              | 1.0  | Điều khiển số dòng & tên file                   |
| `#region` / `#endregion`             | 1.0  | Gom nhóm code để IDE collapse/expand            |
| `#pragma warning`                    | 1.0+ | Bật/tắt/điều chỉnh warning                      |
| `#nullable`                          | 8.0  | Bật/tắt phân tích nullable reference types      |
