# Hệ thống kiểu dữ liệu trong C#

C# là một ngôn ngữ `strongly typed`, có nghĩa là các kiểu dữ liệu được sử dụng rất chặt chẽ, và bạn luôn phải 
xác định kiểu cụ thể của một biến, hằng hoặc biểu thức. Vì C# là ngôn ngữ được thiết kế cho .NET nên nó hỗ trợ 
đầy đủ hệ thống kiểu trong .NET, bao gồm các kiểu dữ liệu primitive và cả một tập rất lớn các kiểu phức tạp được
khai báo trong hệ thống thư viện của .NET.

Các kiểu dữ liệu trong .NET được chia làm hai loại: [value type](value-type.md) (kiểu giá trị) 
và [reference type](reference-type].md) (kiểu tham chiếu).

Khi nói về một kiểu dữ liệu, ta sẽ có các thông tin sau:
  - Kích cỡ không gian bộ nhớ mà kiểu dữ liệu đó chiếm.
  - Các giá trị nhỏ nhất và lớn nhất của kiểu dữ liệu.
  - Các thành phần bên trong kiểu dữ liệu (phương thức, các trường, thuộc tính...).
  - Kiểu dữ liệu cơ sở mà kiểu dữ liệu này thừa kế.
  - Các interface được implement.
  - Các toán tử mà kiểu dữ liệu hỗ trợ.
  
Trình biên dịch có những thông tin trên về tất cả các kiểu dữ liệu, nhờ đó nó hỗ trợ an toàn kiểu (type safe), 
bạn không thể gán các giá trị không tương thích vào cho các biến, ví dụ không thể gán một giá trị số thực vào
một biến kiểu số nguyên (nhưng ngược lại thì được).

Các thông tin kiểu dữ liệu trên cũng được lưu vào file thực thi (.exe hoặc .dll). Trình runtime của .NET (CLR) 
sẽ dùng các thông tin này để đảm bảo an toàn kiểu khi nó cấp phát và thu hồi bộ nhớ.
