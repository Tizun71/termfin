# Tiền tệ và lưu trữ

[English](../04-money-and-persistence.md) | **Tiếng Việt** · [Trang chủ](../../README.vi.md)

## Biểu diễn tiền chính xác

Biểu diễn nội bộ được đề xuất là số nguyên có dấu 64 bit theo đơn vị tiền nhỏ nhất được hỗ trợ. Với phạm vi VND nguyên đồng ban đầu, 150000 nghĩa là 150.000 VND. Muốn hỗ trợ tiền tệ có phần lẻ về sau, cần quy tắc rõ ràng về loại tiền và số chữ số thập phân.

Phân tích trực tiếp văn bản số thập phân thành số tiền nguyên. Không chuyển qua float64. Đầu vào nguyên đồng phải từ chối phần lẻ và ký tự phân nhóm chữ số không được hỗ trợ; việc phân nhóm thuộc khâu hiển thị.

Phân biệt đầu vào sai định dạng, giá trị ngoài phạm vi biểu diễn và số tiền bị quy tắc nghiệp vụ từ chối. Kiểm tra tràn số cho phép cộng, trừ và tổng hợp báo cáo trước khi thực hiện phép tính không an toàn. Lấy giá trị tuyệt đối của số nguyên có dấu nhỏ nhất cũng cần thận trọng.

### Công việc

- [ ] Định nghĩa tập trung đơn vị lưu trữ và hiển thị.
- [ ] Đặc tả định dạng văn bản số tiền được chấp nhận.
- [ ] Kiểm thử phân tích đầu vào, giới hạn biểu diễn, tràn số và số tiền giao dịch không hợp lệ.

## Nguồn dữ liệu chuẩn để tính số dư

Nguồn dữ liệu chuẩn ban đầu được đề xuất là số dư đầu kỳ cộng với các bản ghi giao dịch đã commit. Số dư hiện tại được tính từ nguồn này. Một lần chuyển tiền là một bản ghi chứa cả nguồn và đích.

Cách này tránh duy trì độc lập số dư nguồn, số dư đích và lịch sử giao dịch. Số dư được tính toán vẫn cần thao tác đọc–kiểm tra–ghi nguyên tử: các khoản chi đồng thời không được cùng tiêu một số tiền khả dụng.

Nếu bổ sung số dư lưu đệm về sau, hãy cập nhật chúng cùng lịch sử giao dịch trong một lần commit và cung cấp cơ chế đối soát với bản ghi chuẩn.

## Trình tự chuyển tiền nguyên tử

1. Bắt đầu phạm vi nguyên tử với cơ chế điều phối ghi cần thiết.
2. Đọc trạng thái nguồn và đích trong phạm vi đó.
3. Kiểm tra tham chiếu, hai tài khoản khác nhau, loại tiền, số tiền, tiền khả dụng và tràn số.
4. Chuẩn bị bản ghi chuyển tiền đầy đủ.
5. Lưu và commit một lần.
6. Chỉ báo thành công sau khi commit thành công.

Thất bại trước commit phải giữ nguyên trạng thái đã commit mà bên ngoài có thể quan sát. Lưu nguồn trước rồi bù trừ sau là không đủ: thao tác bù trừ cũng có thể thất bại.

## Lựa chọn lưu trữ

| Bộ điều hợp | Lợi ích chính | Trách nhiệm |
|---|---|---|
| Trong bộ nhớ | Phát triển nghiệp vụ và kiểm thử đơn vị nhanh | Bảo đảm ngữ nghĩa sao chép và tính nguyên tử; không lưu bền vững |
| Tệp JSON | Dễ kiểm tra và thực hành đọc/ghi tệp | Thay thế an toàn, khóa giữa tiến trình, khôi phục và xử lý phiên bản dữ liệu |
| SQLite | Truy vấn có cấu trúc, ràng buộc và giao dịch cơ sở dữ liệu | Chọn driver, migration, hành vi khóa và kiểm thử tích hợp |

Một bộ điều hợp bền vững là đủ cho bản phát hành đầu. Chỉ triển khai trong bộ nhớ chưa đáp ứng yêu cầu lưu trữ lâu dài.

## Hướng dẫn bộ điều hợp JSON

- Lưu ảnh chụp trạng thái nhất quán, có phiên bản và chứa toàn bộ trạng thái cần cho lần commit.
- Xây dựng trạng thái dự kiến tách biệt với trạng thái đã commit. Sao chép nông map lồng nhau hoặc slice chứa con trỏ vẫn có thể dùng chung dữ liệu có thể thay đổi.
- Ghi tệp tạm trên cùng hệ thống tệp; hoàn tất ghi, đẩy dữ liệu xuống hệ thống lưu trữ (flush) và đóng tệp trước khi thay tệp đã commit bằng cơ chế phù hợp với nền tảng.
- Chỉ công bố trạng thái mới trong bộ nhớ sau khi commit tệp thành công.
- Ngăn các tiến trình cạnh tranh ghi đè trạng thái của nhau. Mutex trong một tiến trình không điều phối được các tiến trình riêng biệt.
- Không xóa tệp đã commit trước để nhường chỗ cho tệp thay thế.
- Xem tệp hiện có bị hỏng là lỗi, không phải cơ sở dữ liệu rỗng.
- Xác định cách xử lý tệp tạm, bản sao lưu và phiên bản lược đồ không được hỗ trợ khi khởi động.

Thay thế nguyên tử và khả năng giữ dữ liệu sau mất điện là hai thuộc tính riêng. Kiểm chứng hành vi thay thế, flush và siêu dữ liệu hệ thống tệp trên các nền tảng được hỗ trợ trước khi tuyên bố an toàn khi xảy ra sự cố. Chỉ các kiểm thử giả lập lỗi ghi chưa đủ chứng minh dữ liệu bền vững sau mất điện.

## Hướng dẫn bộ điều hợp SQLite

- Đọc số dư và ghi giao dịch trong cùng giao dịch cơ sở dữ liệu, với cơ chế điều phối ghi phù hợp.
- Dùng ràng buộc cho tham chiếu, ID duy nhất và tính hợp lệ cơ bản của dữ liệu; bật và kiểm chứng thực thi khóa ngoại trên các kết nối liên quan.
- Giữ kiểm tra nghiệp vụ trong service. Ràng buộc cơ sở dữ liệu cung cấp thêm một lớp bảo vệ.
- Rollback khi thất bại trước commit và kiểm tra lỗi commit tường minh.
- Giới hạn việc xử lý lỗi busy/locked; tránh thử lại thao tác ghi một cách mù quáng.
- Dùng migration theo thứ tự, kiểm tra phiên bản lược đồ và xác định quy trình sao lưu/khôi phục.

## Thử lại và kết quả chưa chắc chắn

Tiến trình có thể dừng sau commit nhưng trước khi hiển thị thành công. Chạy lại lệnh có thể tạo bản ghi trùng. Biện pháp đề xuất là dùng ID thao tác ổn định: cùng ID và nội dung tương đương xác định một thao tác đã có; cùng ID nhưng nội dung khác là xung đột.

Tạo ID mới cho mỗi lần thử lại không giải quyết việc trùng lặp. Hỗ trợ thử lại an toàn cần hướng dẫn cách dùng lại ID thao tác và kiểm tra kết quả. Lỗi lưu trữ khiến trạng thái commit chưa chắc chắn phải phân biệt được với rollback đã được xác nhận.

## Công việc nghiệm thu

- [ ] Ghi thất bại giữ nguyên trạng thái đã commit trước đó.
- [ ] Khởi động lại khôi phục dữ liệu và báo cáo tương đương.
- [ ] Dữ liệu hỏng hoặc không được hỗ trợ không bị âm thầm ghi đè.
- [ ] Các khoản chi cạnh tranh không thể cùng vượt qua kiểm tra dựa trên số dư lỗi thời.
- [ ] Thay bộ điều hợp vẫn giữ nguyên hành vi nghiệp vụ.
