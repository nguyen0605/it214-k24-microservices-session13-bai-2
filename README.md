# Báo cáo cấu hình Circuit Breaker cho EWallet-Service

## 1. Giải quyết bài toán nghiệp vụ
Trong hệ thống thanh toán, việc phân định rõ loại lỗi là cực kỳ quan trọng:
- **InsufficientBalanceException (Lỗi nghiệp vụ - HTTP 400):** Do người dùng không đủ tiền trong ví. Đây không phải là lỗi hệ thống nên **không được** tính vào tỷ lệ lỗi của Circuit Breaker. Thuộc tính `ignoreExceptions` được sử dụng để phớt lờ hoàn toàn các ngoại lệ này.
- **TimeoutException (Lỗi hệ thống - HTTP 504):** Do EWallet-Service bị treo mạng. Đây là sự cố hạ tầng/hệ thống thực sự, cần được ghi nhận để ngắt mạch kịp thời nhằm bảo vệ hệ thống khỏi sập dây chuyền. Thuộc tính `recordExceptions` chịu trách nhiệm ghi nhận các lỗi này.

## 2. Kiểm tra checklist tự đánh giá
- **Gửi liên tiếp 8 request ném lỗi InsufficientBalanceException:** Cầu dao vẫn giữ trạng thái **CLOSED** bình thường vì các ngoại lệ này đã bị cấu hình `ignoreExceptions` bỏ qua hoàn toàn, không làm tăng tỷ lệ lỗi.
- **Gửi liên tiếp 5 request ném lỗi TimeoutException:** Cầu dao sẽ lập tức chuyển sang trạng thái **OPEN**. Lý do: 5/5 request đều lỗi (đạt 100% tỷ lệ lỗi, vượt ngưỡng 50%) và số lượng request đã đạt mức tối thiểu `minimumNumberOfCalls = 5`.
- **Khi mạch đang OPEN, gửi 1 request hợp lệ:** Hệ thống sẽ ngay lập tức ném ra ngoại lệ `CallNotPermittedException` mà không thực hiện gọi sang EWallet-Service, giúp tiết kiệm tài nguyên và bảo vệ hệ thống đang gặp sự cố.