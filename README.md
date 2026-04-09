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

Người dùng còn có thể chọn khung tín hiệu nhỏ nhất để nhận theo HTF, ví dụ chỉ nhận HTF từ m15 trở lên.

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

### 4.1 Khôi phục key HTF/LTF sau khi restart

- Bot lưu state quét tín hiệu ra file `scan_state.json` theo chu kỳ.
- Khi khởi động lại, bot khôi phục lại key HTF/LTF và log quán tính từ file này.
- Mục tiêu là tránh mất state khiến bot gửi lại tín hiệu cũ sau restart.

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

### Chặn tín hiệu cùng hướng và loại (Branch-aware blocking)

Bot có cơ chế chặn tín hiệu khi đã có pending signal cùng:
- **Symbol** (cặp tiền)
- **Direction** (hướng: buy/sell)
- **Branch** (loại tín hiệu: pattern hay strong)

**Tây tác này giúp tránh gửi quá nhiều tín hiệu cùng hướng trên cùng một symbol, nhưng vẫn cho phép signal khác loại được gửi.**

#### Ví dụ cụ thể:

**Chu kỳ 1:**
- m15 + m3 xuất hiện **Tích lũy tăng** (pattern long) → gửi signal
- Pending: `{"signal_key": "pattern:EURUSD:m15:m3:123", "branch": "pattern", "direction": "buy"}`

**Chu kỳ 2-3 (trong quá trình m15-m3 pattern long chưa SL):**
- m10 quét được tích lũy tăng (pattern long) → **bị chặn** vì đã có pending pattern buy cùng symbol
- h1 quét được tích lũy tăng (pattern long) → **bị chặn** vì đã có pending pattern buy cùng symbol
- **Nhưng** h1 quét được **quán tính tăng** (strong long) → **được gửi bình thường** vì khác loại (pattern ≠ strong)
- **Và** m15 + m3 quét được **quán tính tăng** (strong long) → **được gửi bình thường** vì khác loại

Log bot sẽ hiển thị:
```
[BLOCK-SIGNAL] Bỏ qua EURUSD h1 (pattern buy) vì đã có pending signal cùng loại và hướng
[SCANNER] Đã gửi EURUSD h1+m15 (lần 1) - tín hiệu quán tính tăng
```

#### Lợi ích:

- **Tránh spam cùng loại:** Nếu m15-m3 đang có tín hiệu tích lũy (pattern), bạn không nhận quá nhiều tín hiệu tích lũy từ m10, m20, h1 v.v.
- **Cho phép tín hiệu khác loại:** Nhưng bạn vẫn nhận được tín hiệu quán tính (strong) vì chúng là loại khác.
- **Giữ workflow 2 lần:** Logic chặn liền kề vẫn hoạt động bình thường (m5 → m10, m10 → m15).
- **Khôi phục sau restart:** Tín hiệu pending được lưu vào `scan_state.json`, khi khởi động lại bot sẽ tiếp tục chặn tín hiệu cùng loại.

### Khung tín hiệu nhỏ nhất

Người dùng có thể chọn khung nhỏ nhất để bot bắt đầu nhận tín hiệu theo HTF.

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

- Chọn m15: bot chỉ xét các HTF từ m15 trở lên, bỏ qua HTF m10.
- Chọn h1: bot chỉ xét các HTF từ h1 trở lên.

Lưu ý quan trọng:

- Ngưỡng này chỉ lọc HTF.
- LTF của từng HTF vẫn chạy bình thường theo map (không bị block bởi ngưỡng này).

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

### Điều chỉnh SL để bảo vệ

Bot tự động kéo SL ra xa một chút (khoảng 10% khoảng cách từ entry đến SL gốc) để tránh SL bị "cắt chặt":

**Cách hoạt động đơn giản:**
- Máy tính ra khoảng cách từ vị trí vào lệnh đến SL bạn chỉ định
- Rồi kéo SL ra xa hơn 10% so với khoảng cách đó
- Ví dụ: Vào lệnh ở 2650, SL gốc ở 2640 → khoảng cách = 10 pip → Bot sẽ kéo SL xuống còn 2639

**Tại sao cần làm vậy:**
- Giá có lúc đảo chiều rất nhanh, SL hiện tại có thể "cắt sớm" do biến động bình thường
- Kéo SL ra xa một chút giúp tín hiệu có thêm không gian để phát triển
- Từ đó tỷ lệ thắng mất giảm, lợi nhuận cao hơn

**Lưu ý:**
- SL đã được điều chỉnh này sẽ hiển thị trong cảnh báo Telegram
- Bạn có thể nói không bằng cách bấm "Đổi SL" để sử dụng SL gốc hoặc nhập mức khác

**Ví dụ thực tế:**
- Nếu tín hiệu gợi ý vào BUY ở 2650, SL ở 2640
- Bot sẽ báo: "Entry: 2650 | SL: 2639" (đã kéo ra 1 pip)
- Bạn có quyền chọn:
  - Vào lệnh với SL 2639 (như bot gợi ý)
  - Hoặc bấm "Đổi SL" nhập 2640 (sử dụng SL gốc)
  - Hoặc nhập mức khác tùy ý

### Báo cáo

- Lưu lịch sử giao dịch
- Xuất báo cáo Excel
- Theo dõi trạng thái lệnh đang mở

## Ví Dụ Cụ Thể

### Ví dụ 1: M15 + M3 tích lũy, nhưng quán tính vẫn được gửi

Giả sử:

- Chu kỳ 1: HTF m15 + LTF m3 xuất hiện **tích lũy tăng** (pattern long) → bot gửi signal
- Chu kỳ 2: m10 hoặc h1 quét được **tích lũy tăng** (pattern long) → **bị chặn** (cùng loại và hướng)
- Chu kỳ 2: Nhưng m15 + m3 hoặc h1 quét được **quán tính tăng** (strong long) → **được gửi** (khác loại)

Kết quả:

- Người dùng không bị spam nhiều tín hiệu tích lũy cùng hướng.
- Nhưng vẫn nhận được tín hiệu quán tính nếu có, vì đó là loại khác.

### Ví dụ 2: M15 + M5 pattern long pending, m15 + m10 strong long được phép gửi

Giả sử:

- Trạng thái: `pending[key] = {"symbol": "EURUSD", "direction": "buy", "branch": "pattern"}`
- m30 quét được pattern long (direction: buy, branch: pattern) → **kiếm tra** `has_pending_signal_with_direction_and_branch("EURUSD", "buy", "pattern")`
  - Kết quả: TRUE → **bị chặn**
  - Log: `[BLOCK-SIGNAL] Bỏ qua EURUSD m30 (pattern buy) vì đã có pending signal cùng loại và hướng`

- Đồng thời, h1 quét được strong long (direction: buy, branch: strong) → **kiểm tra** `has_pending_signal_with_direction_and_branch("EURUSD", "buy", "strong")`
  - Kết quả: FALSE → **được gửi**
  - Log: `[SCANNER] Đã gửi EURUSD h1+m15 (lần 1) - tín hiệu quán tính`

### Ví dụ 3: Pattern short pending, strong long được gửi

Giả sử:

- Trạng thái: `pending[key] = {"symbol": "GBPUSD", "direction": "sell", "branch": "pattern"}`
- m20 quét được pattern short (direction: sell, branch: pattern) → **bị chặn** (cùng direction+branch)
- h1 quét được strong long (direction: buy, branch: strong) → **được gửi** (khác direction và khác branch)

Kết quả:

- Pattern short bị chặn, nhưng signal ngược chiều (long) vẫn được gửi.

### Ví dụ 4: Retry sau SL - pattern được phépa gửi lần 2

Giả sử:

- Chu kỳ 1: m15+m3 pattern long gửi → pending
- Chu kỳ 2: Lệnh bị SL
- Chu kỳ 3: m15 muốn retry cùng pattern long (lần 2) → **được phép** vì signal cũ đã xóa từ pending

Kết quả:

- Bot retry pattern long lần 2 bình thường.
- Nếu chỉ m15+m3 mà không có h1+m15 chạy song song, thì không có xung đột.

### Ví dụ 5: Tín hiệu hết hạn và khôi phục

Giả sử:

- Lúc 10:00: m15+m3 pattern long gửi / saved to scan_state.json
- Lúc 10:15: TTL hết (15 phút) → tín hiệu bị xóa khỏi pending
- Bot bị tắt rồi bật lại lúc 10:20

Kết quả:

- Khi restore_scan_state() chạy, nó kiểm tra TTL mỗi pending signal.
- Signal của lúc 10:00 đã quá 15 phút → bị bỏ qua (không khôi phục).
- Log: `[RESTORE] Bỏ qua pending signal pattern:EURUSD:m15:m3:123 - đã hết hạn`
- Các signal mới có thể được quét lại bình thường.

### Ví dụ 6 (Cũ): Retry sau SL

Giả sử:

- Bot gửi tín hiệu m15 + m5
- Lệnh bị SL
- LTF m5 vẫn còn hợp lệ

Kết quả:

- Bot retry lại m5 lần 2.
- Nếu tiếp tục SL thêm lần nữa, bot sẽ chuyển sang LTF kế tiếp, ví dụ m10 nếu còn phù hợp.

### Ví dụ 7 (Cũ): Chặn HTF liền kề

Giả sử:

- m10 đã gửi buy trong chu kỳ hiện tại
- m15 cũng muốn gửi buy cùng branch

Kết quả:

- Bot kiểm tra cycle_sent (workflow liền kề), thấy m10 đã gửi cùng hướng.
- Bỏ qua m15 để tránh ngay lập tức 2 signal gần nhau.

### Ví dụ 8 (Cũ): Tín hiệu hết hạn

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
- Có cơ chế persist state scanner qua `scan_state.json` để giữ key HTF/LTF sau khi tắt mở bot.
- Có cơ chế block signal cùng direction và branch (pattern chặn pattern, strong chặn strong).
- Pending signals được lưu vào `scan_state.json` và khôi phục khi bot restart.
  - Tín hiệu hết hạn (> SIGNAL_TTL) sẽ tự động bị loại bỏ khi restore.
  - Ensures direction blocking logic vẫn hoạt động sau restart.

## Changelog

### 2026-04-08 (Update 2)

- Thêm chặn tín hiệu cùng hướng và loại (branch-aware blocking).
  - Pattern signal chỉ chặn pattern signal cùng hướng.
  - Strong signal chỉ chặn strong signal cùng hướng.
  - Pattern long pending không chặn strong long mới.
- Lưu branch thông tin vào pending dict.
- Khôi phục pending signals từ scan_state.json với TTL validation.
  - Signal hết hạn (> 15 phút) sẽ tự động bị loại khi restore.
- Update helper function: `has_pending_signal_with_direction_and_branch()` để check cả 3 criteria.

### 2026-04-08

- Thêm chọn khung tín hiệu nhỏ nhất từ m5 đến h4.
- Sửa logic phát hiện hỗ trợ kháng cự cao/thấp dần
- Vị trí sl của từng vị thế đã chính xác hơn
