# REWbot.Mt5

REWbot.Mt5 là bot hỗ trợ giao dịch bán tự động, kết hợp MT5 và Telegram để quét tín hiệu, gửi cảnh báo, quản lý lệnh và ghi nhận lịch sử giao dịch.

Bot được thiết kế theo workflow sau:

1. Quét dữ liệu thị trường từ MT5.
2. Phát hiện tín hiệu theo RSI - EMA - WMA.
3. Gửi ảnh và nội dung tín hiệu cho người dùng qua Telegram.
4. Chờ người dùng xác nhận vào lệnh hoặc hủy.
5. Theo dõi tín hiệu và lệnh đang mở theo thời gian thực.
6. Tự động xử lý SL, retry, chuyển LTF, breakeven và cảnh báo đảo chiều.

## HTF và LTF là gì?

Trong bot này, bạn sẽ thấy 2 khái niệm rất quan trọng:

- HTF: Higher Time Frame, tức khung thời gian lớn hơn. Ví dụ: m15, m30, h1, h2.
- LTF: Lower Time Frame, tức khung thời gian nhỏ hơn dùng để xác nhận hoặc quản lý tín hiệu. Ví dụ: m3, m5, m10.

Hiểu đơn giản:

- HTF cho bạn bối cảnh chính của kèo.
- LTF cho bạn điểm vào lệnh, xác nhận tín hiệu, hoặc theo dõi quản lý lệnh.

Ví dụ:

- HTF m15 + LTF m5 nghĩa là bot đang nhìn xu hướng/bối cảnh ở m15, rồi dùng m5 để tìm tín hiệu cụ thể hơn.
- Nếu m15 vẫn còn quán tính nhưng m5 bị SL nhiều lần, bot có thể chuyển sang LTF tiếp theo như m10 nếu điều kiện còn phù hợp.

## Workflow Chính

### 1. Bot quét thị trường

Mỗi vòng quét, bot lấy dữ liệu M1 từ MT5 rồi resample ra các khung thời gian khác nhau để tìm tín hiệu.

Các nhóm tín hiệu chính:

- Tích lũy tăng / Tích lũy giảm
- Quán tính tăng / Quán tính giảm

### 2. Bot ưu tiên khung nào

Bot sẽ quét theo danh sách HTF/LTF đã được định nghĩa sẵn trong code.

Ví dụ:

- HTF m15 có thể đi với LTF m3 hoặc m5
- HTF m30 có thể đi với LTF m6 hoặc m8
- HTF h1 có thể đi với LTF m8, m10 hoặc m15

Người dùng còn có thể chọn khung tín hiệu nhỏ nhất để nhận, ví dụ chỉ nhận từ m15 trở lên.

### 3. Bot gửi tín hiệu

Khi có tín hiệu hợp lệ, bot sẽ gửi:

- 1 ảnh khung HTF
- 1 ảnh khung LTF
- Entry dự kiến
- SL dự kiến
- TP dự kiến theo RR Ratio
- Volume dự kiến theo % rủi ro

Sau đó bot chờ người dùng bấm Có hoặc Không.

### 4. Bot theo dõi tín hiệu chờ

Tín hiệu sẽ có thời gian sống giới hạn. Nếu quá thời gian này mà chưa xác nhận thì tín hiệu sẽ hết hạn.

Trong lúc chờ, bot vẫn kiểm tra:

- Giá có chạm SL chưa
- Tín hiệu còn hợp lệ không

Nếu giá đã chạm SL, tín hiệu sẽ bị hủy.

### 5. Bot quản lý lệnh sau khi vào

Sau khi lệnh được mở, bot tiếp tục theo dõi:

- Lệnh đã chạm 1R hay chưa
- Có đạt 2R để dời SL về Entry hay không
- Có tín hiệu đảo chiều trên LTF hay không
- Có cần đóng lệnh hoặc chuyển khung quản lý hay không

## Quy Tắc Quản Lý Lệnh

Bot quản lý lệnh theo các quy tắc sau:

### 1. Sau khi vào lệnh

- Bot lưu lệnh vào danh sách theo dõi.
- Bot ghi nhận Entry, SL, TP, hướng lệnh và LTF đang dùng.
- Nếu lệnh là lệnh ngoài bot, người dùng có thể chọn LTF quản lý thủ công.

### 2. Mốc 1R

- Khi lệnh đạt 1R, bot bắt đầu coi đó là một lệnh đang được quản lý tích cực.
- Mốc này không tự dời SL, nhưng là điều kiện để bật các bước quản lý tiếp theo.

### 3. Mốc 2R

- Khi lệnh đạt 2R, bot sẽ xử lý breakeven.
- Nếu đang ở chế độ tự động, bot dời SL về Entry ngay.
- Nếu đang ở chế độ thủ công, bot hỏi người dùng có muốn dời SL về Entry hay không.

### 4. Theo dõi tín hiệu đảo chiều trên LTF

- Bot liên tục quét LTF của lệnh đang mở.
- Nếu xuất hiện tín hiệu đảo chiều đúng hướng quản lý, bot có thể đóng lệnh.
- Với chế độ tự động, bot đóng ngay.
- Với chế độ thủ công, bot gửi cảnh báo để người dùng quyết định.

### 5. Xử lý SL và retry tín hiệu

- Nếu một tín hiệu mới bị SL, bot sẽ đánh dấu trạng thái đó.
- Bot sẽ retry phát hiện và gửi lại cùng LTF tối đa 2 lần.
- Nếu cùng LTF bị SL quá số lần cho phép, bot chuyển sang LTF tiếp theo trong danh sách.
- Nếu không còn LTF phù hợp, state đó bị xóa.

### 6. Quản lý lệnh ngoài bot

- Người dùng có thể đưa lệnh đang mở trên MT5 vào watchlist.
- Sau đó chọn LTF quản lý cho lệnh đó.
- Nếu lệnh chưa có SL, bot sẽ yêu cầu nhập SL trước khi theo dõi.

### 7. Hết hiệu lực tín hiệu

- Tín hiệu chờ xác nhận có TTL giới hạn.
- Nếu hết TTL, bot sẽ hủy tín hiệu đó.
- Nếu giá đã chạm SL trong lúc chờ xác nhận, bot cũng hủy tín hiệu.

## Tính Năng Chính

### Tín hiệu giao dịch

- Phát hiện tín hiệu Tích lũy tăng / giảm
- Phát hiện tín hiệu Quán tính tăng / giảm
- Hỗ trợ retry khi bị SL
- Hỗ trợ chuyển sang LTF tiếp theo khi LTF hiện tại bị SL 2 lần

### Chặn HTF liền kề

Bot có cơ chế chặn HTF liền kề theo cùng hướng trong cùng một chu kỳ quét.

Ví dụ:

- Nếu m10 đã gửi tín hiệu buy trong chu kỳ hiện tại, thì m15 buy cùng branch có thể bị chặn.
- Cơ chế này giúp tránh gửi quá nhiều tín hiệu cùng chiều ở các khung gần nhau.

### Khung tín hiệu nhỏ nhất

Người dùng có thể chọn khung nhỏ nhất để bot bắt đầu nhận tín hiệu.

Danh sách hỗ trợ:

- m5
- m10
- m15
- m20
- m30
- m45
- h1
- h2
- h3
- h4

Ví dụ:

- Chọn m15: bot chỉ nhận tín hiệu từ m15 trở lên, bỏ qua m5, m10.
- Chọn h1: bot chỉ nhận tín hiệu từ h1 trở lên, bỏ qua toàn bộ các khung thấp hơn.

### Bộ lọc hướng kèo

Người dùng có thể chọn:

- Long + Short
- Chỉ Long
- Chỉ Short

### Quản lý lệnh

Bot hỗ trợ:

- Tự động đóng lệnh theo tín hiệu LTF
- Dời SL về Entry khi đạt 2R
- Chế độ tự động hoặc thủ công
- Watchlist cho lệnh ngoài bot
- Retry tối đa 2 lần cho cùng một LTF trước khi chuyển LTF khác
- Chuyển sang LTF tiếp theo nếu LTF hiện tại bị SL nhiều lần

### Báo cáo

- Lưu lịch sử giao dịch
- Xuất báo cáo Excel
- Theo dõi trạng thái lệnh đang mở

## Ví Dụ Cụ Thể

### Ví dụ 1: Tín hiệu m15 + m3

Giả sử:

- HTF m15 đang có quán tính tăng
- LTF m3 xuất hiện Long hợp lệ
- Người dùng không chặn hướng buy
- Khung tín hiệu nhỏ nhất đang chọn là m5

Kết quả:

- Bot sẽ không gửi vì m3 nhỏ hơn ngưỡng m5.

Nếu đổi ngưỡng xuống m3 thì bot mới có thể gửi.

### Ví dụ 2: Retry sau SL

Giả sử:

- Bot gửi tín hiệu m15 + m5
- Lệnh bị SL
- LTF m5 vẫn còn hợp lệ

Kết quả:

- Bot retry lại m5 lần 2.
- Nếu tiếp tục SL thêm lần nữa, bot sẽ chuyển sang LTF kế tiếp, ví dụ m10 nếu còn phù hợp.

### Ví dụ 3: Chặn HTF liền kề

Giả sử:

- m10 đã gửi buy trong chu kỳ hiện tại
- m15 cũng muốn gửi buy cùng branch

Kết quả:

- Bot có thể chặn m15 buy để tránh trùng hướng trên HTF liền kề.

### Ví dụ 4: Tín hiệu hết hạn

Giả sử:

- Bot đã gửi tín hiệu
- Người dùng không bấm xác nhận trong thời gian TTL

Kết quả:

- Tín hiệu tự hết hạn
- Bot ngừng theo dõi tín hiệu đó

## Cách Sử Dụng

### 1. Khởi động bot

- Đảm bảo MT5 đã mở và đăng nhập.
- Chạy file khởi động của project.
- Gửi lệnh /start trong Telegram.

### 2. Cài đặt ban đầu

Bot sẽ hỏi lần lượt:

- Balance
- % rủi ro mỗi lệnh
- RR Ratio

### 3. Điều chỉnh cài đặt

Các lệnh Telegram chính:

- /settings: mở menu cài đặt
- /status: xem trạng thái hiện tại
- /stats: xem thống kê
- /close: đóng lệnh thủ công
- /manage: thay đổi chế độ quản lý lệnh
- /monitor: theo dõi lệnh ngoài bot
- /about: thông tin bot

### 4. Chọn khung tín hiệu nhỏ nhất

Trong /settings:

1. Bấm Thay đổi khung tín hiệu nhỏ nhất
2. Chọn một giá trị từ m5 đến h4
3. Thay đổi có hiệu lực từ chu kỳ quét tiếp theo


## Ghi Chú Kỹ Thuật

- Bot dùng Plotly để vẽ ảnh biểu đồ.
- Ảnh được gửi qua Telegram kèm caption và nút thao tác.
- Có cơ chế khử trùng lặp tín hiệu theo state.
- Có cơ chế kiểm tra SL trước khi xác nhận lệnh.

## Changelog

### 2026-04-08

- Thêm chọn khung tín hiệu nhỏ nhất từ m5 đến h4.
- Bổ sung workflow retry và chuyển LTF cho strong_signal giống pattern.
- Tối ưu render ảnh bằng cách gửi ảnh trực tiếp từ bộ nhớ thay vì ghi ra file rồi mở lại.
