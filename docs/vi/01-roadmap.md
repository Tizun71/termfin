# Lộ trình triển khai

[English](../01-roadmap.md) | **Tiếng Việt** · [Trang chủ](../../README.vi.md)

Lộ trình này chia kế hoạch triển khai thành các mốc có thể kiểm chứng độc lập. Tài liệu không có nghĩa là bất kỳ mốc nào đã hoàn thành.

## Phạm vi ban đầu được đề xuất

- Một người dùng cục bộ và một loại tiền: VND, biểu diễn theo đơn vị đồng nguyên.
- Số dư tài khoản không âm; không có nợ, tỷ giá, lãi suất hoặc tích hợp ngân hàng.
- Giao dịch đã ghi nhận không thể sửa trong phiên bản đầu. Cơ chế điều chỉnh cần thiết kế riêng.
- Ngân sách giúp người dùng theo dõi chi tiêu; không ngăn việc ghi nhận khoản chi.
- Repository trong bộ nhớ phục vụ phát triển nghiệp vụ và kiểm thử đơn vị. Bản phát hành đầu cần một bộ điều hợp lưu trữ bền vững, JSON hoặc SQLite.
- Các thao tác ghi được thực hiện tuần tự. Có thể áp dụng và cưỡng chế chính sách chỉ một bên ghi trong phiên bản đầu.

Đây là các ràng buộc đề xuất, chưa phải khả năng đã triển khai. Có thể xem xét hỗ trợ tiền tệ tổng quát riêng mà vẫn giữ yêu cầu biểu diễn tiền bằng số nguyên.

### Các quyết định về phạm vi

- [ ] Xác nhận loại tiền ban đầu và chính sách sửa giao dịch.
- [ ] Chọn bộ điều hợp lưu trữ bền vững đầu tiên.
- [ ] Chọn múi giờ báo cáo và cơ chế cấu hình.
- [ ] Xác nhận quy tắc nghiệp vụ trước khi triển khai service.
- [ ] Mô tả cách ngăn chặn hoặc điều phối các thao tác ghi đồng thời.

## Mốc 1: Nền tảng nghiệp vụ

- [ ] Thiết lập mô-đun Go và cấu trúc package tối thiểu.
- [ ] Định nghĩa kiểu tiền tệ, tài khoản và định danh.
- [ ] Định nghĩa hợp đồng repository và các nhóm lỗi.
- [ ] Triển khai tạo tài khoản với repository trong bộ nhớ.
- [ ] Thêm điểm khởi chạy CLI đơn giản và kiểm thử tính hợp lệ của tài khoản.

Nghiệm thu: có thể tạo tài khoản qua service độc lập với CLI. Chưa cần lưu trữ bền vững ở mốc này.

## Mốc 2: Giao dịch cốt lõi

- [ ] Triển khai danh mục thu nhập và chi tiêu.
- [ ] Triển khai ghi nhận thu nhập và chi tiêu.
- [ ] Triển khai chuyển tiền trong một thao tác lưu trữ nguyên tử.
- [ ] Kiểm thử tính hợp lệ, thiếu tiền, tràn số và lỗi commit.

Nghiệm thu: chuyển tiền thành công bảo toàn tổng số tiền; chuyển tiền thất bại giữ nguyên số dư và lịch sử giao dịch.

## Mốc 3: Lưu trữ bền vững

- [ ] Triển khai bộ điều hợp JSON hoặc SQLite theo hợp đồng hiện có.
- [ ] Khôi phục trạng thái đã commit sau khi khởi động lại.
- [ ] Xử lý rõ ràng lỗi lưu trữ và dữ liệu hỏng hoặc không được hỗ trợ.
- [ ] Cưỡng chế chính sách ghi đồng thời đã chọn.

Nghiệm thu: việc lưu trữ không đòi hỏi thay đổi quy tắc nghiệp vụ; ghi thất bại không làm lộ cập nhật dở dang.

## Mốc 4: Báo cáo và trao đổi dữ liệu

- [ ] Triển khai ngân sách tháng và tổng hợp chi tiêu.
- [ ] Triển khai báo cáo thu chi hằng tháng.
- [ ] Triển khai nhập/xuất JSON và CSV có phiên bản.
- [ ] Mô tả đầu vào, đầu ra và hành vi lỗi của CLI.

Nghiệm thu: chuyển tiền không được tính vào tổng thu chi; dữ liệu xuất có thể khôi phục dữ liệu nghiệp vụ tương đương vào kho trống.

## Mốc 5: Sẵn sàng phát hành

- [ ] Hoàn thành ít nhất 30 ca kiểm thử đơn vị có ý nghĩa.
- [ ] Hoàn thành kiểm thử tích hợp lưu trữ và một kịch bản nghiệm thu CLI.
- [ ] Thêm quy trình GitHub Actions chạy kiểm thử.
- [ ] Kiểm chứng hướng dẫn cài đặt và sử dụng trên môi trường sạch.
- [ ] Hoàn thành danh sách tiêu chí hoàn thành và chọn giấy phép.

## Kiến thức Go được thực hành

| Khái niệm | Ứng dụng |
|---|---|
| Biến, hằng, hàm | Chuẩn hóa và kiểm tra đầu vào |
| Struct và phương thức | Tài khoản, giao dịch, ngân sách và hành vi nghiệp vụ |
| Con trỏ | Ranh giới thay đổi dữ liệu và quyền sở hữu |
| Array, slice, map | Giá trị có kích thước cố định, danh sách giao dịch và tra cứu định danh |
| Interface và composition | Hợp đồng repository và bộ điều hợp có thể thay thế |
| Xử lý lỗi | Phân biệt lỗi nghiệp vụ, phân tích dữ liệu, hủy thao tác và lưu trữ |
| Quản lý package | Package có trách nhiệm gắn kết và phụ thuộc tường minh |
| Đọc/ghi tệp và JSON | Ảnh chụp trạng thái bền vững và định dạng trao đổi |
| context.Context | Hủy thao tác qua ranh giới service và I/O |
| Kiểm thử theo bảng | Đầu vào hợp lệ, không hợp lệ và trường hợp biên |

Mỗi mốc cần nêu rõ đầu vào, đầu ra, hành vi khi thất bại và chuyển đổi trạng thái. Rà soát phụ thuộc giữa các package và ranh giới nguyên tử trước khi mở rộng phạm vi.
