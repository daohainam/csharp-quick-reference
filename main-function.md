# Hàm Main

Mỗi một chương trình C# phải chứa một hàm Main, và đây sẽ là hàm đầu tiên được thực thi.
Nếu chương trình có nhiều hơn một hàm Main, bạn sẽ phải chỉ ra đâu là hàm được chọn thực thi đầu tiên khi biên dịch 
(sử dụng `MainEntryPoint` hoặc `StartupObject`).

## Khai báo hàm Main
```csharp
class HelloWorld
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello World!");
    }
}
```

Các chương trình C#10 có thể dùng [Top-level statements](top-level-statements.md):
```csharp
		Console.WriteLine("Hello World!");
}
```

## Những cách khai báo hàm Main hợp lệ
```csharp
static void Main() { }
static int Main() { }
static void Main(string[] args) { }
static int Main(string[] args) { }
static async Task Main() { }
static async Task<int> Main() { }
static async Task Main(string[] args) { }
static async Task<int> Main(string[] args) { }
```

- Hàm Main luôn được khai báo là `static`, `class` hoặc `struct` chứa Main và bản thân Main không bắt buộc phải là public.
- Hàm Main có thể trả về một trong những kiểu sau: `void`, `int`, `Task`, or `Task<int>`. Trường hợp trả về int hoặc `Task<int>`, 
Main có thể được khai báo là async và giá trị trả về sẽ là giá trị trả về cho chương trình gọi nó, ngược lại tương đương 
với việc Main trả về 0.
- Hàm Main có thể được khai báo không có tham số `string[]` như trong ví dụ trên.
- Khi khai báo Main là async, trình biên dịch sẽ đổi tên hàm Main của bạn thành MainAsync và tạo ra một hàm Main gọi hàm
MainAsync.

