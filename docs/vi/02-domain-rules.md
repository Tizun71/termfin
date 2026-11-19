# Quy tắc nghiệp vụ

[English](../02-domain-rules.md) | **Tiếng Việt** · [Trang chủ](../../README.vi.md)

Tài liệu đề xuất mô hình nghiệp vụ ban đầu nhất quán. Khi thay đổi quy tắc, cần cập nhật ví dụ và kết quả kiểm thử mong đợi trước khi triển khai.

## Tài khoản

Tài khoản có ID ổn định, tên, loại tiền và số dư đầu kỳ. Số dư hiện tại được tính từ số dư đầu kỳ và các giao dịch đã commit.

- Tên sau khi loại bỏ khoảng trắng ở hai đầu không được rỗng.
- Đề xuất về tính duy nhất: so sánh tên sau khi loại bỏ khoảng trắng ở hai đầu, không phân biệt chữ hoa và chữ thường. Giữ nguyên dấu và mô tả chính sách so sánh Unicode.
- Tham chiếu sử dụng ID ổn định thay vì tên. Đổi tên phải bảo toàn các mối liên hệ.
- Số dư đầu kỳ phải không âm và không được tính là thu nhập tháng.
- Trong phiên bản đầu, không thể xóa tài khoản đã được giao dịch tham chiếu.

## Danh mục

Danh mục có ID, tên và loại thu nhập hoặc chi tiêu.

- Tên phải hợp lệ và duy nhất trong cùng loại theo chính sách chuẩn hóa đã chọn.
- Khoản thu phải dùng danh mục thu nhập; khoản chi phải dùng danh mục chi tiêu.
- Chuyển tiền không có danh mục thu nhập hoặc chi tiêu.
- Không thể xóa danh mục đã được tham chiếu. Đổi tên phải bảo toàn tham chiếu hiện có.

## Giao dịch

Mỗi giao dịch có ID, loại, số tiền dương, thời điểm, ghi chú tùy chọn và các ID tài khoản/danh mục liên quan. Loại tiền ban đầu là VND.

| Loại | Tham chiếu bắt buộc | Tác động |
|---|---|---|
| Thu nhập | Tài khoản nhận và danh mục thu nhập | Tăng số dư tài khoản |
| Chi tiêu | Tài khoản chi trả và danh mục chi tiêu | Giảm số dư tài khoản |
| Chuyển tiền | Tài khoản nguồn và đích khác nhau | Giảm nguồn và tăng đích trong cùng một lần commit |

Quy tắc chung:

- Số tiền phải dương và nằm trong phạm vi biểu diễn của kiểu số nguyên đã chọn.
- Tài khoản và danh mục được tham chiếu phải tồn tại.
- Khoản chi và chuyển tiền không được khiến số dư hiện tại của tài khoản nguồn âm.
- Phép tính phải phát hiện tràn số, bao gồm khi cộng vào tài khoản đích.
- Thời điểm phải có thông tin múi giờ. Chính sách đề xuất: từ chối giao dịch trong tương lai, dùng đồng hồ có thể thay thế để kiểm thử cho kết quả xác định.
- Cho phép ghi nhận giao dịch lùi ngày theo chính sách số dư hiện tại được đề xuất. Điều này không bảo đảm mọi số dư trong lịch sử đều không âm.
- Muốn bảo đảm số dư lịch sử không âm, cần kiểm tra lại dòng thời gian giao dịch bị ảnh hưởng và bổ sung kiểm thử.

### Ví dụ minh họa

Ví A có số dư đầu kỳ 1.000.000 VND; Ngân hàng B có số dư đầu kỳ 0 VND.

1. Ghi nhận thu nhập 2.000.000 VND vào B: A = 1.000.000; B = 2.000.000.
2. Ghi nhận chi ăn uống 100.000 VND từ A: A = 900.000; B = 2.000.000.
3. Chuyển 300.000 VND từ B sang A: A = 1.200.000; B = 1.700.000.

Tổng tiền cuối cùng là 2.900.000 VND. Thu nhập báo cáo là 2.000.000 VND và chi tiêu là 100.000 VND. Chuyển tiền không làm thay đổi tổng thu nhập hoặc tổng chi tiêu.

## Ngân sách

Phạm vi ngân sách được đề xuất là một danh mục chi tiêu trong một tháng dương lịch, trên tất cả tài khoản.

- Khóa duy nhất: danh mục chi tiêu, tháng và loại tiền.
- Hạn mức phải dương. Tạo trùng trả về lỗi; thay đổi hạn mức là một thao tác cập nhật riêng.
- Đã chi bằng tổng các khoản chi đủ điều kiện trong danh mục và tháng.
- Còn lại bằng hạn mức trừ đã chi, có thể âm.
- Chi đúng hạn mức là chạm hạn mức; chi cao hơn là vượt hạn mức.
- Việc làm tròn khi hiển thị không được quyết định ngân sách có bị vượt hay không.
- Thu nhập, chuyển tiền và số dư đầu kỳ không tiêu hao ngân sách.

## Phân tích theo tháng

- Dùng một múi giờ được cấu hình để xác định giao dịch thuộc tháng nào.
- Bao gồm thời điểm đầu tháng được chọn và loại trừ thời điểm đầu tháng kế tiếp.
- Trả về tổng thu nhập, tổng chi tiêu, chênh lệch thu trừ chi và chi tiêu theo danh mục.
- Dòng tiền ròng trong tháng khác với số dư tài khoản hiện tại.
- Tháng không có dữ liệu trả về tổng bằng không và bảng phân tích danh mục rỗng.
- Tất cả giá trị trong một báo cáo phải lấy từ cùng một ảnh chụp dữ liệu nhất quán.

## Các nhóm lỗi

Phân biệt đầu vào không hợp lệ, tham chiếu không tồn tại, trùng lặp, thiếu tiền, tràn số, xung đột ghi, hủy thao tác, dữ liệu lưu trữ hỏng và lỗi I/O hoặc cơ sở dữ liệu.

Service trả về lỗi mà không in ra terminal. CLI chuyển lỗi thành thông báo hữu ích. Repository không được báo ghi thành công khi thao tác ghi thất bại.

## Các quyết định còn bỏ ngỏ

- [ ] Xác nhận cách chuẩn hóa tên và quy tắc xác định tên duy nhất.
- [ ] Xác nhận chính sách giao dịch trong tương lai và giao dịch lùi ngày.
- [ ] Đặc tả cấu hình múi giờ và định dạng nhập tháng.
- [ ] Xác nhận hạn chế sửa và xóa trong phiên bản đầu.
- [ ] Bổ sung tiêu chí nghiệm thu cho mọi phần mở rộng phạm vi.
