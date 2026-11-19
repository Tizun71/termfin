# CLI và trao đổi dữ liệu

[English](../05-cli-and-data-exchange.md) | **Tiếng Việt** · [Trang chủ](../../README.vi.md)

## Hành vi lệnh

Xác định đầu vào, đầu ra và trường hợp thất bại của lệnh trước khi chọn thư viện CLI. Bộ xử lý lệnh giao quyết định nghiệp vụ cho service.

| Thao tác | Đầu vào chính | Đầu ra mong đợi |
|---|---|---|
| Tạo/liệt kê tài khoản | Tên và số dư đầu kỳ | ID, tên, số dư và loại tiền |
| Tạo/liệt kê danh mục | Tên và loại | ID, tên và loại |
| Ghi nhận thu nhập | Tài khoản, danh mục, số tiền, thời điểm | ID giao dịch và chi tiết |
| Ghi nhận chi tiêu | Tài khoản, danh mục, số tiền, thời điểm | Chi tiết giao dịch và số dư sau giao dịch |
| Chuyển tiền | Nguồn, đích, số tiền, thời điểm | Một ID chuyển tiền và tham chiếu tới cả hai tài khoản |
| Liệt kê giao dịch | Bộ lọc tài khoản/tháng tùy chọn | Bản ghi được sắp xếp theo thứ tự xác định |
| Tạo/cập nhật ngân sách | Danh mục chi tiêu, tháng, hạn mức | Hạn mức, đã chi và còn lại |
| Báo cáo tháng | Tháng | Thu nhập, chi tiêu, chênh lệch và phân tích theo danh mục |
| Nhập/xuất | Định dạng và đường dẫn | Số bản ghi hoặc lỗi có hướng xử lý |

Quy ước đầu vào đề xuất: tháng dùng YYYY-MM; thời điểm gồm ngày, giờ và độ lệch UTC; số tiền VND dùng văn bản số nguyên không có ký tự phân nhóm chữ số.

Dùng đầu ra chuẩn cho kết quả được yêu cầu và đầu ra lỗi chuẩn cho thông tin chẩn đoán. Mã thoát 0 nghĩa là thành công; cần mô tả các mã khác 0. Sắp xếp danh sách theo thứ tự xác định, ví dụ theo thời điểm rồi đến ID. Cú pháp lệnh cụ thể sẽ được đặc tả sau khi chọn thiết kế CLI.

### Công việc

- [ ] Đặc tả tên lệnh, đối số bắt buộc và nội dung trợ giúp.
- [ ] Tách lỗi cú pháp khỏi lỗi nghiệp vụ.
- [ ] Cung cấp thông báo lỗi có hướng xử lý.
- [ ] Bảo đảm mọi lệnh giao quy tắc nghiệp vụ cho service có thể kiểm thử.

## Hợp đồng trao đổi dữ liệu chung

Phạm vi đề xuất gồm tài khoản, danh mục, giao dịch và ngân sách trong cả hai định dạng. Không được mô tả bản xuất chỉ có giao dịch là bản sao lưu đầy đủ.

Mỗi định dạng phải xác định phiên bản lược đồ, mã hóa UTF-8, đơn vị tiền, ID ổn định, tham chiếu, trường bắt buộc và tùy chọn, cách xử lý trường không biết, thời điểm, múi giờ, bản ghi trùng và tính nguyên tử khi nhập.

Chính sách ban đầu được đề xuất: từ chối trường hoặc cột không biết để phát hiện lỗi gõ tên. Thay đổi về tương thích sau này cần có phiên bản và kiểm thử.

## Định dạng JSON

Dùng tài liệu gốc có phiên bản, chứa các tập hợp tài khoản, danh mục, giao dịch và ngân sách, cùng múi giờ báo cáo và siêu dữ liệu tiền tệ. Bao gồm các trường dữ liệu chuẩn cần cho việc khôi phục; số dư hiển thị được tính toán không được trở thành nguồn dữ liệu chuẩn thứ hai.

Biểu diễn số tiền khi trao đổi được đề xuất là chuỗi số nguyên thập phân, tránh công cụ trung gian âm thầm làm tròn số tiền lớn. Cách này khác với kiểu số nguyên nội bộ. Giải mã không được chuyển số tiền qua giá trị dấu phẩy động.

Giữ mô hình trao đổi độc lập với cấu trúc lưu trữ riêng của bộ điều hợp để việc đổi lưu trữ không thay đổi hợp đồng tệp công khai.

## Định dạng CSV

Bản xuất đầy đủ được đề xuất là một thư mục chứa các tệp:

| Tệp | Cột |
|---|---|
| metadata.csv | schema_version, currency, timezone |
| accounts.csv | id, name, currency, opening_balance |
| categories.csv | id, name, kind |
| transactions.csv | id, kind, account_id, source_account_id, destination_account_id, category_id, amount, occurred_at, note |
| budgets.csv | id, category_id, month, currency, limit |

metadata.csv chứa một dòng dữ liệu. Tập hợp rỗng vẫn có dòng tiêu đề. Trường tiền chứa văn bản số nguyên chưa định dạng.

Thu nhập và chi tiêu dùng account_id và category_id. Chuyển tiền dùng source_account_id và destination_account_id. Trường không áp dụng phải để trống và được kiểm tra đúng như vậy.

Dùng bộ phân tích/mã hóa CSV thay vì tách dòng theo dấu phẩy. Ghi chú có thể chứa dấu phẩy, dấu ngoặc kép và xuống dòng. Chính sách mã hóa đề xuất: chấp nhận UTF-8 BOM tùy chọn khi đọc và không ghi BOM khi xuất.

## Quy trình nhập

1. Kiểm tra phiên bản lược đồ, mã hóa, tiêu đề và cấu trúc tài liệu.
2. Phân tích thành bản ghi trung gian; lỗi cần chỉ rõ tệp, bản ghi/dòng hoặc đường dẫn trường khi phù hợp.
3. Phát hiện ID trùng trong lô và đối chiếu với dữ liệu hiện có.
4. Xây dựng trạng thái dự kiến và giải quyết tham chiếu, kể cả tham chiếu được tạo trong cùng lô.
5. Áp dụng kiểm tra nghiệp vụ trong phạm vi nguyên tử.
6. Commit toàn bộ lô một lần.

### Chính sách xung đột và kiểm tra được đề xuất

- Nhập bổ sung dữ liệu, không âm thầm thay thế kho hiện có.
- ID đã tồn tại với nội dung nghiệp vụ tương đương: bỏ qua và báo riêng.
- ID đã tồn tại với nội dung khác: từ chối toàn bộ lô.
- ID lặp trong lô: từ chối lô.
- Không ghi đè số dư đầu kỳ hoặc giao dịch hiện có.
- Kiểm tra số dư trên trạng thái dự kiến đã kết hợp, không phụ thuộc thứ tự dòng. Điều này tuân theo chính sách số dư hiện tại và không bảo đảm số dư lịch sử không âm.
- Bản xuất chỉ có một phần lịch sử có thể không nhập độc lập được theo quy tắc số dư không âm. Bản sao lưu đầy đủ chứa toàn bộ dữ liệu chuẩn.
- Kiểm tra siêu dữ liệu múi giờ. Từ chối nhập vào kho không rỗng nếu múi giờ báo cáo xung đột; khi khôi phục vào kho trống, dùng múi giờ trong bản xuất.

Nội dung tương đương được xác định bằng giá trị nghiệp vụ đã chuẩn hóa, không phải so sánh byte hoặc thứ tự thuộc tính JSON. Lô thất bại phải báo số bản ghi thêm đã commit bằng không.

## Quy trình xuất

- Đọc một ảnh chụp dữ liệu nhất quán.
- Dùng thứ tự bản ghi xác định.
- Với CSV, chuẩn bị thư mục xuất mới và chỉ báo thành công khi mọi tệp bắt buộc đã hoàn tất. Nhập phải từ chối bộ tệp thiếu.
- Không âm thầm ghi đè đầu ra hiện có. Tùy chọn ghi đè, nếu có, phải tường minh và được triển khai an toàn.
- Kiểm chứng xuất → nhập vào kho trống bằng cách so sánh dữ liệu nghiệp vụ, tham chiếu, số dư và báo cáo.

## Công việc đặc tả còn lại

- [ ] Định nghĩa kiểu trường, giá trị enum được chấp nhận, giới hạn độ dài và yêu cầu có điều kiện.
- [ ] Định nghĩa giới hạn kích thước tệp và số bản ghi.
- [ ] Đặc tả tương thích phiên bản và hành vi khôi phục múi giờ.
- [ ] Kiểm thử tên Unicode, dấu ngoặc kép, dấu phẩy và ghi chú nhiều dòng.
- [ ] Đặc tả số lượng đã thêm/đã bỏ qua và chi tiết lỗi có cấu trúc.
