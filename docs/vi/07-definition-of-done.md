# Tiêu chí hoàn thành

[English](../07-definition-of-done.md) | **Tiếng Việt** · [Trang chủ](../../README.vi.md)

Danh sách này xác định tiêu chí đề xuất cho bản phát hành đầu. Tất cả mục vẫn chưa hoàn thành cho đến khi có bằng chứng triển khai và kiểm chứng.

## Phạm vi chức năng

- [ ] Có thể tạo và liệt kê nhiều tài khoản với số dư chính xác.
- [ ] Danh mục thu nhập và chi tiêu tuân theo quy tắc kiểm tra đã mô tả.
- [ ] Thu nhập và chi tiêu được kiểm tra và ghi nhận chính xác.
- [ ] Chuyển tiền nguyên tử và không làm tăng sai báo cáo thu nhập hoặc chi tiêu.
- [ ] Có thể tạo và cập nhật ngân sách, hiển thị số đã chi và còn lại.
- [ ] Báo cáo tháng tuân theo múi giờ cấu hình và quy tắc ranh giới.
- [ ] Nhập/xuất JSON tuân theo hợp đồng trao đổi đã công bố.
- [ ] Nhập/xuất CSV tuân theo hợp đồng trao đổi đã công bố.
- [ ] Dữ liệu đã commit vẫn còn sau khi khởi động lại.

## Kiến trúc và tính đúng đắn

- [ ] Bộ xử lý CLI và main.go không chứa logic nghiệp vụ.
- [ ] Interface repository tách service khỏi bộ điều hợp lưu trữ cụ thể.
- [ ] Số tiền không bao giờ đi qua biểu diễn dấu phẩy động.
- [ ] Kiểm tra số tiền, số dư, tham chiếu, loại danh mục, thời điểm và tràn số.
- [ ] Thất bại trước commit không làm lộ thay đổi dở dang.
- [ ] Lô nhập không hợp lệ không commit bản ghi bổ sung nào.
- [ ] Có mô tả hành vi đối với ID trùng, thử lại và kết quả chưa chắc chắn.
- [ ] Việc truyền context và hành vi rollback được xác định rõ.
- [ ] Chính sách ghi đồng thời đã chọn được cưỡng chế.

## Kiểm thử

- [ ] Ít nhất 30 ca kiểm thử đơn vị riêng biệt, có ý nghĩa và kết quả mong đợi có tên rõ ràng.
- [ ] Kiểm thử theo bảng bao phủ tính hợp lệ và ranh giới nghiệp vụ.
- [ ] Chuyển tiền được kiểm thử thành công, thiếu tiền, tràn số và lỗi commit.
- [ ] Kiểm thử phân tích loại trừ chuyển tiền và số dư đầu kỳ khỏi thu chi.
- [ ] Kiểm thử tích hợp bao phủ mở lại kho và vòng xuất–nhập của cả hai định dạng.
- [ ] Kiểm thử chạy trên dữ liệu giả lập độc lập, không có đường dẫn riêng của máy.

## Yêu cầu GitHub Actions

Hiện chưa có quy trình tự động. Ở mốc triển khai, cần thêm quy trình:

1. Chạy khi push và tạo pull request.
2. Checkout kho mã và chọn phiên bản Go tương thích với khai báo mô-đun.
3. Tải và giải quyết các phụ thuộc đã khai báo.
4. Kiểm tra định dạng, chạy phân tích tĩnh phù hợp và thực thi kiểm thử đơn vị.
5. Chạy kiểm thử tích hợp cho bộ điều hợp đã chọn, nêu rõ yêu cầu nền tảng/driver.
6. Thất bại khi một bước kiểm tra bắt buộc thất bại và cung cấp log hữu ích.

### Công việc CI

- [ ] Thêm quy trình trong .github/workflows khi bắt đầu triển khai.
- [ ] Kiểm chứng một lượt chạy thành công trên GitHub.
- [ ] Xác nhận kiểm thử cố ý thất bại khiến quy trình thất bại, sau đó khôi phục kiểm thử.
- [ ] Kiểm thử các nền tảng được hỗ trợ khi hành vi hệ thống tệp hoặc driver khác nhau, bao gồm Windows nếu công bố hỗ trợ.

## Tài liệu cài đặt và sử dụng

Chưa cung cấp lệnh cài đặt vì ứng dụng chưa được triển khai. Trước khi phát hành, cần mô tả và kiểm chứng:

- [ ] Phiên bản Go yêu cầu, hệ điều hành được hỗ trợ và phụ thuộc thực tế.
- [ ] Các bước checkout, giải quyết phụ thuộc, build và chạy trên môi trường sạch.
- [ ] Lựa chọn lưu trữ, vị trí dữ liệu mặc định, cấu hình và hành vi múi giờ.
- [ ] Ví dụ đầy đủ dùng dữ liệu tài chính giả lập.
- [ ] Lệnh, đối số, định dạng tiền/thời gian và mã thoát.
- [ ] Yêu cầu tệp nhập/xuất và chính sách bản ghi trùng.
- [ ] Sao lưu, khôi phục và xử lý dữ liệu hỏng.
- [ ] Giới hạn hỗ trợ tiền tệ, ghi lùi ngày, thao tác đồng thời, sửa giao dịch và độ bền dữ liệu đã được kiểm chứng khi có sự cố.
- [ ] Hướng dẫn kiểm thử đơn vị và tích hợp.

Chỉ giới thiệu hành vi đã triển khai và kiểm chứng.

## Kịch bản nghiệm thu thủ công

Dùng kho kiểm thử mới:

1. Tạo Ví A với 1.000.000 VND và Ngân hàng B với 0 VND.
2. Tạo Lương là danh mục thu nhập và Ăn uống là danh mục chi tiêu.
3. Ghi nhận thu nhập 2.000.000 VND vào B và chi ăn uống 100.000 VND từ A trong cùng tháng.
4. Chuyển 300.000 VND từ B sang A.
5. Kiểm chứng A = 1.200.000; B = 1.700.000; thu nhập tháng = 2.000.000; chi tiêu = 100.000.
6. Đặt ngân sách Ăn uống trong tháng là 500.000 VND và kiểm chứng còn lại 400.000 VND.
7. Thử chuyển số tiền vượt số dư B; kiểm chứng số dư và lịch sử giữ nguyên.
8. Khởi động lại và kiểm chứng kết quả không đổi.
9. Xuất/nhập JSON vào kho trống; lặp lại với CSV và một kho trống khác.
10. Nhập lại dữ liệu giống hệt và kiểm chứng không tạo trùng. Nhập lô không hợp lệ và kiểm chứng không có bản ghi bổ sung nào được commit.

## Danh sách công bố

- [ ] Chọn giấy phép nguồn mở và thêm toàn văn vào tệp LICENSE.
- [ ] Bảo đảm README phân biệt chính xác tính năng có sẵn và dự kiến.
- [ ] Giữ hướng dẫn đóng góp phù hợp với công cụ thực tế của kho mã.
- [ ] Chỉ công bố ví dụ và dữ liệu kiểm thử giả lập; loại trừ dữ liệu tài chính thật, thông tin xác thực và cấu hình cục bộ.
- [ ] Giải quyết đặc tả mâu thuẫn và ghi rõ giới hạn còn lại trước khi phát hành.

## Rà soát khả năng bảo trì

- [ ] Có thể giải thích biểu diễn tiền và giới hạn số học từ tài liệu.
- [ ] Có thể xác định ranh giới commit và rollback trong luồng chuyển tiền và nhập dữ liệu.
- [ ] Có thể thay quy tắc kiểm tra bằng các kiểm thử service tập trung.
- [ ] Thêm bộ điều hợp lưu trữ không đòi hỏi viết lại quy tắc nghiệp vụ.
- [ ] Vẫn phân biệt được lỗi phân tích đầu vào, nghiệp vụ, hủy thao tác và lưu trữ.
- [ ] Báo cáo mới có thể dùng hợp đồng đọc hiện có mà không vượt ranh giới các lớp.
