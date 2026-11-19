# Kiến trúc

[English](../03-architecture.md) | **Tiếng Việt** · [Trang chủ](../../README.vi.md)

## Ranh giới các lớp

Luồng yêu cầu: đầu vào terminal → bộ xử lý CLI → service ứng dụng → hợp đồng repository → bộ điều hợp lưu trữ.

Service chịu trách nhiệm về quy tắc nghiệp vụ và phụ thuộc vào các hợp đồng mà nó sử dụng. Bộ điều hợp triển khai các hợp đồng đó. Điểm khởi chạy chọn và kết nối các thành phần phụ thuộc. Thay JSON bằng SQLite không được làm thay đổi việc kiểm tra chuyển tiền hoặc tính toán ngân sách.

## Cấu trúc package đề xuất

| Đường dẫn | Trách nhiệm |
|---|---|
| cmd/termfin/main.go | Khởi động, cấu hình, context gốc, kết nối phụ thuộc và mã thoát |
| internal/cli/ | Phân tích đối số, chuyển đổi đầu vào, gọi service và trình bày |
| internal/account/ | Kiểu tài khoản, kiểm tra hợp lệ và ca sử dụng |
| internal/category/ | Kiểu danh mục, kiểm tra hợp lệ và quy tắc tham chiếu |
| internal/transaction/ | Thu nhập, chi tiêu, chuyển tiền và hợp đồng thao tác nguyên tử |
| internal/budget/ | Hạn mức ngân sách và đánh giá chi tiêu |
| internal/analytics/ | Tổng hợp theo tháng và ranh giới báo cáo |
| internal/transferio/ | Điều phối nhập/xuất, tách biệt với chuyển tiền giữa tài khoản |
| internal/storage/ | Bộ điều hợp trong bộ nhớ và bền vững, giao dịch lưu trữ và phiên bản dữ liệu |
| internal/domain/ | Kiểu nghiệp vụ dùng chung khi cần, như tiền tệ và định danh |
| pkg/ | Package tùy chọn được chủ động hỗ trợ cho bên ngoài sử dụng |
| migrations/ | Thay đổi lược đồ theo thứ tự nếu chọn SQLite |
| test/ | Kiểm thử tích hợp và nghiệm thu CLI; kiểm thử đơn vị nằm cạnh package tương ứng |
| go.mod | Định danh mô-đun và khai báo phụ thuộc |

Đây là cấu trúc đề xuất, chưa phải cây thư mục hiện có. Thêm package khi trách nhiệm xuất hiện rõ. Không cần pkg hoặc migrations nếu chưa có mục đích sử dụng.

## Trách nhiệm của CLI

Bộ xử lý CLI có thể phân tích đối số, chuyển đổi đầu vào dạng văn bản, báo lỗi cú pháp lệnh, gọi service và định dạng kết quả.

Service xác định tính hợp lệ của tham chiếu, sự tương thích của danh mục, số dư đủ dùng, an toàn số học, ranh giới cập nhật nguyên tử và điều kiện tính vào ngân sách/báo cáo. Các quy tắc này phải được bảo đảm cả khi gọi service trực tiếp mà không qua terminal.

## Hợp đồng repository

Xác định hành vi trước khi viết interface:

1. Mỗi service cần đọc và ghi dữ liệu nào?
2. Bên gọi cần phân biệt những lỗi nào?
3. Giá trị trả về là bản sao độc lập hay tham chiếu có thể thay đổi?
4. Bản ghi không tồn tại và danh sách rỗng có ý nghĩa gì?
5. Thứ tự kết quả có xác định không?
6. Ai tạo ID và bảo đảm tính duy nhất?
7. Những thao tác đọc và ghi nào phải cùng nằm trong một phạm vi nguyên tử?

Ưu tiên interface nhỏ do bên sử dụng sở hữu, hoặc package hợp đồng dùng chung khi nhiều bên thực sự cần. Không đưa kiểu SQL, đường dẫn tệp và chi tiết tuần tự hóa vào hợp đồng service nghiệp vụ. Đường dẫn nhập/xuất thuộc ranh giới I/O.

## Thao tác nguyên tử

Hợp đồng unit-of-work được đề xuất cho phép service thực hiện các thao tác liên quan trong cùng một phạm vi nguyên tử. Service đọc tài khoản, kiểm tra số dư và ghi giao dịch bằng các repository gắn với phạm vi đó. Bộ điều hợp xử lý cô lập, khóa, commit và rollback.

Đọc số dư ngoài phạm vi này rồi chỉ bắt đầu giao dịch khi ghi sẽ tạo khoảng trống có thể dùng dữ liệu đã lỗi thời. Cập nhật nguồn và đích không được là hai lần lưu độc lập. Lô nhập dữ liệu nên dùng lại cùng cơ chế nguyên tử.

## Chiều phụ thuộc

- Các kiểu nghiệp vụ dùng chung nên có ít phụ thuộc.
- Service không được import bộ điều hợp lưu trữ cụ thể.
- Lớp lưu trữ có thể dùng kiểu nghiệp vụ và triển khai hợp đồng phục vụ service.
- Phân tích dữ liệu đọc qua hợp đồng, không truy cập tệp hoặc bộ xử lý CLI trực tiếp.
- Khi gặp vòng lặp import, rà soát quyền sở hữu và ranh giới trước khi thêm lớp trừu tượng.

## Truyền context

Truyền context tường minh từ điểm khởi chạy qua service và các thao tác I/O. Không dùng context để chứa tham số nghiệp vụ thông thường hoặc lưu context của một yêu cầu lâu dài trong service.

Kiểm tra hủy thao tác hữu ích quanh các lần chờ và công việc kéo dài. Hủy thao tác không tự cung cấp rollback. Hủy trước commit cần loại bỏ thay đổi tạm thời. Khi đã biết commit thành công, việc hủy đến muộn không được báo như bằng chứng rằng chưa lưu gì. Kết quả commit chưa xác định cần lỗi riêng và chính sách thử lại/đối soát.

## Công việc thiết kế

- [ ] Mô tả luồng chi tiêu và chuyển tiền, gồm ranh giới nguyên tử.
- [ ] Đặc tả kết quả, lỗi, quyền sở hữu dữ liệu và thứ tự của repository.
- [ ] Xác nhận nguồn dữ liệu chuẩn để tính số dư.
- [ ] Kiểm chứng có thể kiểm thử service mà không cần CLI hoặc lưu trữ bền vững.
- [ ] Rà soát đồ thị phụ thuộc để phát hiện vòng lặp.
- [ ] Mô tả thành phần nào thay đổi khi thêm bộ điều hợp lưu trữ thứ hai.
