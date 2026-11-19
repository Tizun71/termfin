# Termfin

[English](README.md) | **Tiếng Việt**

Termfin là dự án ứng dụng quản lý tài chính cá nhân trên terminal, dự kiến được viết bằng Go. Thiết kế bao gồm nhiều tài khoản, thu chi, chuyển tiền, danh mục, ngân sách, báo cáo tháng và trao đổi dữ liệu CSV/JSON.

**Trạng thái: hiện chỉ có thiết kế và hướng dẫn triển khai.** Kho mã hiện chứa tài liệu. Mã ứng dụng, mô-đun Go, cơ chế lưu trữ bền vững và quy trình CI chưa được triển khai. Các danh sách công việc mô tả kế hoạch, chưa phải tính năng có sẵn.

## Tính năng dự kiến

| Mô-đun | Phạm vi |
|---|---|
| Tài khoản | Nhiều tài khoản, số dư đầu kỳ và số dư hiện tại |
| Giao dịch | Thu nhập, chi tiêu và chuyển tiền nguyên tử |
| Danh mục | Phân loại riêng cho thu nhập và chi tiêu |
| Ngân sách | Hạn mức theo danh mục trong tháng và tổng hợp chi tiêu |
| Phân tích | Thu nhập, chi tiêu hằng tháng và phân tích theo danh mục |
| Nhập/xuất | Định dạng CSV và JSON có phiên bản |
| Lưu trữ | Bộ điều hợp dùng tệp hoặc SQLite, thông qua các giao diện repository |

## Tài liệu

| Tài liệu | Nội dung |
|---|---|
| [Lộ trình triển khai](docs/vi/01-roadmap.md) | Quyết định về phạm vi, các mốc và kiến thức Go |
| [Quy tắc nghiệp vụ](docs/vi/02-domain-rules.md) | Kiểm tra hợp lệ, số dư, chuyển tiền, ngân sách và báo cáo |
| [Kiến trúc](docs/vi/03-architecture.md) | Trách nhiệm của từng package và hợp đồng repository |
| [Tiền tệ và lưu trữ](docs/vi/04-money-and-persistence.md) | Số tiền dạng số nguyên, ghi nguyên tử và các lựa chọn lưu trữ |
| [CLI và trao đổi dữ liệu](docs/vi/05-cli-and-data-exchange.md) | Hành vi lệnh và hợp đồng CSV/JSON |
| [Chiến lược kiểm thử](docs/vi/06-testing.md) | 40 tình huống kiểm thử đơn vị đề xuất và phạm vi kiểm thử tích hợp |
| [Tiêu chí hoàn thành](docs/vi/07-definition-of-done.md) | Tiêu chí nghiệm thu, yêu cầu CI và tài liệu phát hành |

Đọc quy tắc nghiệp vụ và kiến trúc trước khi triển khai tính năng. Các đề xuất thiết kế được ghi rõ; quyết định chưa chốt được thể hiện bằng công việc chưa đánh dấu hoàn thành. Cập nhật đặc tả và kiểm thử liên quan mỗi khi quy tắc thay đổi. Khi sửa tài liệu, cập nhật cả bản tiếng Anh và tiếng Việt để hai bản nhất quán.

## Mục tiêu thiết kế

- Giữ quy tắc nghiệp vụ độc lập với đầu vào terminal và công nghệ lưu trữ.
- Biểu diễn tiền chính xác; từ chối số tiền không hợp lệ hoặc gây tràn số.
- Hoàn tất toàn bộ giao dịch chuyển tiền và lô nhập dữ liệu, hoặc giữ nguyên trạng thái đã được ghi nhận.
- Cung cấp lỗi có hành vi rõ ràng, kiểm thử có thể tái lập và định dạng dữ liệu được mô tả trong tài liệu.
- Giữ phần triển khai đủ nhỏ để dễ hiểu và mở rộng.

Dự án cũng là môi trường thực hành kiến thức Go cơ bản: hàm, struct, phương thức, con trỏ, tập hợp, interface, composition, xử lý lỗi, package, đọc/ghi tệp, mã hóa JSON, truyền context và kiểm thử theo bảng.

## Đóng góp

Ở giai đoạn này, cải thiện tài liệu, bổ sung trường hợp biên và góp ý thiết kế đều hữu ích. Khi đề xuất thay đổi, hãy mô tả vấn đề, quy tắc bị ảnh hưởng, hành vi mong đợi và tiêu chí nghiệm thu. Giữ đặc tả và ví dụ nhất quán giữa các tài liệu và cả hai ngôn ngữ.

Đóng góp mã nguồn cần tuân theo ranh giới các lớp đã được mô tả và có kiểm thử cho hành vi bị ảnh hưởng. Bổ sung hướng dẫn cài đặt và ví dụ có thể chạy khi ứng dụng đã hoạt động.

## Giấy phép

Dự án chưa chọn hoặc thêm giấy phép nguồn mở. Cần chọn giấy phép và bổ sung toàn văn trước khi công bố dự án dưới dạng bản phát hành nguồn mở có giấy phép.
