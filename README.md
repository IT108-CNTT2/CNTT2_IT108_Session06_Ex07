## Phần 1: Phân tích lỗi

Khi trace code dòng lệnh xử lý log trong script, nguyên nhân khiến file `daily_critical.txt` gần như trống rỗng được xác định như sau:

1. Khớp chữ hoa/thường (`-CaseSensitive`): Lệnh `Select-String` được cấu hình với tham số `-CaseSensitive` và tìm kiếm từ khóa `"critical"` (chữ thường).  
2. Sai lệch dữ liệu log: Trong tệp `server.log`, các dòng lỗi thực tế được ghi bằng chữ hoa hoàn toàn: `CRITICAL`.  
3. Hậu quả: Do bật chế độ phân biệt chữ hoa/thường, PowerShell không tìm thấy dòng nào khớp với `"critical"`. Kết quả là không có dòng lỗi nào được nối thêm (`-Append`) vào file, khiến file chỉ còn lại các dòng tiêu đề trắng.

## Phần 2: Sửa lỗi và Kiểm thử

### 1\. Đoạn script `deploy_checker.ps1` đã được sửa đúng

Sửa bằng cách đổi pattern thành `"CRITICAL"` hoặc bỏ hẳn tham số `-CaseSensitive` (vì mặc định `Select-String` không phân biệt hoa/thường):

PowerShell  
\# Script: deploy\_checker.ps1 (Đã sửa lỗi)  
\# Mục đích: Trích xuất lỗi CRITICAL từ server.log vào daily\_critical.txt

"=== Bat dau quet log: \$(Get-Date) \===" | Out-File daily\_critical.txt  
Select-String \-Path server.log \-Pattern "CRITICAL" | Out-File \-Append daily\_critical.txt  
"=== Hoan tat. So dong loi: \===" | Out-File \-Append daily\_critical.txt  
(Get-Content daily\_critical.txt).Count | Out-File \-Append daily\_critical.txt

### 2\. Kết quả mong đợi của file `daily_critical.txt`

Sau khi chạy script với dữ liệu log mẫu, tệp kết quả sẽ chứa nội dung sau:

Plaintext  
\=== Bat dau quet log: 09/18/2026 02:30:00 \===  
2026-09-18 02:30:11 CRITICAL Database connection pool exhausted  
2026-09-18 02:30:45 CRITICAL Retry limit exceeded for payment-service  
\=== Hoan tat. So dong loi: \===  
4

### 3\. Hiện tượng khi file `server.log` không tồn tại

Nếu tệp `server.log` bị mất hoặc chưa được tạo trên hệ thống:

* Lệnh `Select-String` sẽ sinh ra một lỗi ngoại lệ (Exception / Error Stream) thông báo không tìm thấy đường dẫn file (`Path not found`).  
* PowerShell sẽ hiển thị thông báo lỗi màu đỏ trên màn hình console, nhưng script vẫn sẽ chạy tiếp các dòng lệnh phía sau (ghi dòng footer và đếm số dòng hiện có trong file báo cáo).

"# CNTT2_IT108_Session06_Ex07" 
