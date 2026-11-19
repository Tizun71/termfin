# Chiến lược kiểm thử

[English](../06-testing.md) | **Tiếng Việt** · [Trang chủ](../../README.vi.md)

## Mục tiêu kiểm thử đơn vị

Bản phát hành đầu yêu cầu ít nhất 30 ca kiểm thử đơn vị có ý nghĩa. 40 tình huống đề xuất dưới đây là kế hoạch khởi đầu; chưa tình huống nào được triển khai. Các ca liên quan có thể dùng kiểm thử con có tên trong kiểm thử theo bảng. Khi báo số lượng, cần nói rõ đang đếm hàm, kiểm thử con hay tình huống; tránh lặp ca chỉ để đủ số lượng.

Kiểm thử đơn vị không nên cần tệp thật, SQLite hoặc CLI đang chạy. Dùng repository trong bộ nhớ hoặc bản giả lập cho phép chủ động gây lỗi. Thành phần thay thế trong kiểm thử phải tuân thủ hợp đồng nguyên tử của repository; nếu không, kiểm thử đạt vẫn có thể che giấu lỗi rollback.

## Tình huống kiểm thử đơn vị đề xuất

| # | Phạm vi | Tình huống và kết quả mong đợi |
|---|---|---|
| 01 | Tiền tệ | Văn bản số nguyên hợp lệ được phân tích thành số tiền nguyên đồng chính xác |
| 02 | Tiền tệ | Từ chối đầu vào có phần lẻ trong phạm vi VND ban đầu |
| 03 | Tiền tệ | Từ chối đầu vào không phải số |
| 04 | Tiền tệ | Từ chối giá trị ngoài phạm vi số nguyên |
| 05 | Tài khoản | Tạo hợp lệ sẽ gán ID và giữ nguyên số dư đầu kỳ |
| 06 | Tài khoản | Từ chối tên chỉ có khoảng trắng |
| 07 | Tài khoản | Từ chối tên trùng sau chuẩn hóa |
| 08 | Tài khoản | Từ chối số dư đầu kỳ âm |
| 09 | Danh mục | Có thể tạo danh mục thu nhập và chi tiêu hợp lệ |
| 10 | Danh mục | Từ chối loại không được hỗ trợ |
| 11 | Danh mục | Từ chối tên trùng sau chuẩn hóa trong cùng loại |
| 12 | Thu nhập | Khoản thu hợp lệ tăng số dư và ghi một giao dịch |
| 13 | Thu nhập | Tài khoản không tồn tại gây lỗi và không ghi dữ liệu |
| 14 | Thu nhập | Từ chối danh mục chi tiêu và không ghi dữ liệu |
| 15 | Thu nhập | Từ chối tràn số dư và không ghi dữ liệu |
| 16 | Giao dịch | Từ chối số tiền bằng không và âm qua các đầu vào riêng |
| 17 | Chi tiêu | Khoản chi hợp lệ giảm số dư và ghi một giao dịch |
| 18 | Chi tiêu | Chi đúng toàn bộ số dư khả dụng thành công |
| 19 | Chi tiêu | Thiếu tiền giữ nguyên số dư và lịch sử |
| 20 | Chi tiêu | Danh mục không tồn tại hoặc không tương thích không gây ghi dữ liệu |
| 21 | Chuyển tiền | Thành công cập nhật cả hai số dư và bảo toàn tổng tiền |
| 22 | Chuyển tiền | Từ chối nguồn và đích giống nhau |
| 23 | Chuyển tiền | Tài khoản không tồn tại khiến toàn bộ trạng thái giữ nguyên |
| 24 | Chuyển tiền | Nguồn thiếu tiền giữ nguyên cả hai số dư và lịch sử |
| 25 | Chuyển tiền | Tràn số ở đích giữ nguyên nguồn |
| 26 | Tính nguyên tử | Lỗi commit được giả lập và xác nhận thất bại giữ nguyên toàn bộ trạng thái trước đó |
| 27 | Context | Hủy trước commit trả về lỗi hủy và không thay đổi dữ liệu |
| 28 | Ngân sách | Có thể tạo ngân sách hợp lệ cho danh mục chi tiêu/tháng |
| 29 | Ngân sách | Từ chối tạo trùng danh mục/tháng/loại tiền |
| 30 | Ngân sách | Từ chối hạn mức không dương và danh mục thu nhập |
| 31 | Ngân sách | Chi đúng hạn mức còn lại bằng không và không vượt hạn mức |
| 32 | Ngân sách | Chi trên hạn mức tạo số còn lại âm và trạng thái vượt hạn mức |
| 33 | Phân tích | Tháng trống trả về tổng bằng không và bảng phân tích rỗng |
| 34 | Phân tích | Chuyển tiền và số dư đầu kỳ không tính là thu nhập hoặc chi tiêu |
| 35 | Phân tích | Bao gồm giao dịch đúng thời điểm đầu tháng |
| 36 | Phân tích | Loại trừ giao dịch đúng thời điểm đầu tháng kế tiếp |
| 37 | Phân tích | Chuyển đổi múi giờ đưa giao dịch ở ranh giới vào đúng tháng |
| 38 | Nhập | Một bản ghi không hợp lệ ngăn toàn bộ lô commit |
| 39 | Nhập | Nhập lại ID đã có với nội dung tương đương không tạo trùng |
| 40 | Nhập | ID đã có với nội dung khác khiến toàn bộ lô bị từ chối |

Với trường hợp thất bại, kiểm tra nhóm lỗi và bảo đảm không có thay đổi trạng thái ngoài ý muốn. Chỉ kiểm tra có lỗi là chưa đủ.

## Cấu trúc kiểm thử theo bảng

Mỗi ca cần có tên mô tả, trạng thái ban đầu, đầu vào, đầu ra/lỗi mong đợi và trạng thái cuối mong đợi.

Ví dụ chi tiêu, đơn vị VND:

- Có 100.000; chi 40.000: còn 60.000 và thêm một giao dịch.
- Có 100.000; chi 100.000: còn 0 và thêm một giao dịch.
- Có 100.000; chi 100.001: lỗi thiếu tiền, giữ số dư ban đầu và không thêm giao dịch.

Mỗi kiểm thử con tạo trạng thái độc lập. Tránh dùng chung map, slice, repository hoặc bộ tạo ID có thể thay đổi. Kiểm soát đồng hồ và định danh để kết quả không phụ thuộc thời gian thực hoặc thứ tự chạy.

## Phạm vi bổ sung

- [ ] Từ chối thời điểm trong tương lai bằng đồng hồ được kiểm soát.
- [ ] Tổng hợp báo cáo phát hiện tràn số.
- [ ] Đổi tên danh mục giữ nguyên tham chiếu giao dịch.
- [ ] CSV xử lý dấu ngoặc kép, dấu phẩy, Unicode và ghi chú nhiều dòng.
- [ ] JSON giữ chính xác số tiền nguyên lớn.
- [ ] Thiếu cột, trường không biết và phiên bản không hỗ trợ tạo lỗi rõ ràng.
- [ ] Nhập giải quyết tham chiếu trong lô không phụ thuộc thứ tự dòng và từ chối ID lặp.
- [ ] Từ chối xung đột siêu dữ liệu múi giờ mà không thay đổi dữ liệu.
- [ ] Hủy đến muộn không báo sai rằng chưa lưu khi đã biết commit thành công.
- [ ] Kết quả commit chưa chắc chắn tuân theo chính sách đối soát đã mô tả.

## Phạm vi kiểm thử tích hợp

Các kiểm thử này dùng bộ điều hợp thật và không thay thế yêu cầu kiểm thử đơn vị:

- [ ] Lưu trạng thái, đóng kho, mở lại và kiểm chứng dữ liệu cùng báo cáo tương đương.
- [ ] JSON: đầu vào hỏng không bị xem là kho trống; lỗi ghi giữ nguyên dữ liệu đã commit.
- [ ] SQLite: lỗi trong giao dịch rollback các thay đổi; kiểm chứng thực thi khóa ngoại.
- [ ] Các khoản chi cạnh tranh không thể chi quá số dư; có thể từ chối bên ghi thứ hai nếu cưỡng chế chính sách chỉ một bên ghi.
- [ ] Xuất/nhập JSON vào kho trống bảo toàn tham chiếu, số dư và báo cáo.
- [ ] Xuất/nhập CSV vào kho trống bảo toàn tham chiếu, số dư và báo cáo.
- [ ] Nghiệm thu CLI bao gồm tài khoản, thu nhập, chi tiêu, chuyển tiền, ngân sách, báo cáo và khởi động lại.

Chỉ chạy kiểm thử riêng cho bộ điều hợp đã triển khai. Khi thêm bộ điều hợp thứ hai, chạy bộ kiểm thử hợp đồng repository dùng chung trên cả hai.

## Bằng chứng hoàn thành

- [ ] Ghi nhận số ca kiểm thử đơn vị có ý nghĩa và kết quả tích hợp.
- [ ] Bao gồm đầu vào không hợp lệ, lỗi lưu trữ, ranh giới tháng và trường hợp rollback.
- [ ] Kiểm chứng rằng cố ý làm sai một quy tắc khiến kiểm thử tương ứng thất bại.
- [ ] Dùng dữ liệu kiểm thử tạm, không chứa dữ liệu tài chính thật hoặc đường dẫn riêng của máy.
