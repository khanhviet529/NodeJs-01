# SMTP, IMAP, POP3 — Giải thích chi tiết

## 1. SMTP là gì?

**SMTP (Simple Mail Transfer Protocol)** là giao thức chuẩn dùng để **gửi email** qua Internet.

### Luồng gửi email đầy đủ:

```
Bạn nhấn "Gửi" trong Outlook/Gmail
    ↓
Ứng dụng email (client) kết nối đến SMTP Server qua port 587
    ↓
SMTP Server của bạn (vd: smtp.gmail.com) xác thực và nhận thư
    ↓
SMTP Server chuyển tiếp đến SMTP Server của người nhận (vd: smtp.outlook.com)
    ↓
Thư được lưu trữ trên server của người nhận
    ↓
Ứng dụng Outlook của người nhận dùng IMAP/POP3 để tải thư về
    ↓
Người nhận thấy email trong hộp thư đến
```

### Đặc điểm:
- Hoạt động ở tầng ứng dụng (Application Layer) trong mô hình TCP/IP
- Chỉ dùng để **gửi** email — để nhận email cần dùng IMAP hoặc POP3
- Hoạt động theo mô hình client-server: client kết nối, xác thực, rồi chuyển thư đi

---

## 2. Các giao thức liên quan

| Giao thức | Vai trò | Port phổ biến |
|---|---|---|
| **SMTP** | Gửi email | 587, 465, 25 |
| **IMAP** | Nhận email, đồng bộ nhiều thiết bị | 993 (SSL), 143 |
| **POP3** | Nhận email, tải về máy local | 995 (SSL), 110 |

### Tại sao không dùng SMTP để nhận thư?
SMTP chỉ thiết kế để **đẩy (push) thư đi**, không có cơ chế để client **kéo (pull) thư về**.
Vì vậy cần IMAP hoặc POP3 để hỏi server "có thư mới không?".

---

## 3. IMAP — Internet Message Access Protocol

### IMAP là gì?
IMAP là giao thức dùng để **nhận và quản lý email**, thư **vẫn được lưu trên server**, ứng dụng chỉ đồng bộ về máy để hiển thị.

### Cách hoạt động:
```
Server lưu thư
    ↓
Outlook (máy A) ─── IMAP ───► Đồng bộ → thấy thư
Gmail app (điện thoại) ─── IMAP ───► Đồng bộ → thấy thư (giống hệt)
Thunderbird (máy B) ─── IMAP ───► Đồng bộ → thấy thư (giống hệt)
```

### Đặc điểm:
- Thư **lưu trên server** — không mất khi xóa app hay đổi máy
- **Đồng bộ 2 chiều**: đọc thư trên điện thoại → máy tính cũng tự động đánh dấu đã đọc
- Xóa thư trên một thiết bị → **xóa trên tất cả thiết bị**
- Hỗ trợ **folders, labels, search** trên server
- Cần kết nối internet liên tục để xem thư mới

### Port IMAP:
| Port | Mã hóa | Trạng thái |
|---|---|---|
| 143 | STARTTLS | Vẫn dùng |
| **993** | SSL/TLS | **Khuyến nghị** |

### Khi nào dùng IMAP?
- Dùng email trên **nhiều thiết bị** (điện thoại + máy tính + tablet)
- Muốn thư được **backup trên server**
- Dùng **Gmail, Outlook, Yahoo** — tất cả đều mặc định IMAP

---

## 4. POP3 — Post Office Protocol version 3

### POP3 là gì?
POP3 là giao thức dùng để **tải email từ server về máy local**, sau khi tải xong thư **thường bị xóa khỏi server**.

### Cách hoạt động:
```
Server lưu thư
    ↓
Outlook (máy A) ─── POP3 ───► Tải thư về máy A
    ↓
Thư bị XÓA khỏi server (mặc định)
    ↓
Gmail app (điện thoại) ─── POP3 ───► Không thấy thư nữa (đã bị xóa trên server)
```

### Đặc điểm:
- Thư **tải về máy local** — không cần internet sau khi đã tải
- **Không đồng bộ** nhiều thiết bị — thư chỉ có trên máy đã tải
- Mặc định **xóa thư trên server** sau khi tải (có thể cấu hình giữ lại)
- Đơn giản, ít tốn tài nguyên server
- Không hỗ trợ folders, labels phía server

### Port POP3:
| Port | Mã hóa | Trạng thái |
|---|---|---|
| 110 | STARTTLS | Vẫn dùng |
| **995** | SSL/TLS | **Khuyến nghị** |

### Khi nào dùng POP3?
- Chỉ dùng email trên **một thiết bị duy nhất**
- Muốn **lưu thư offline** không cần internet
- Dung lượng server bị giới hạn — tải về để giải phóng server
- Hệ thống cũ, legacy

---

## 5. So sánh IMAP vs POP3

| Tiêu chí | IMAP | POP3 |
|---|---|---|
| Thư lưu ở đâu | **Server** | Máy local |
| Đồng bộ nhiều thiết bị | **Có** | Không |
| Xóa thư | Xóa trên tất cả thiết bị | Chỉ xóa trên máy đó |
| Cần internet | Có | Chỉ khi tải |
| Backup tự động | **Có (trên server)** | Không |
| Hỗ trợ folders/labels | **Có** | Không |
| Phổ biến hiện nay | **Rất phổ biến** | Ít dùng |
| Port SSL | 993 | 995 |

> **Kết luận**: Hầu hết người dùng hiện nay nên dùng **IMAP** — đồng bộ tốt, backup tự động, dùng được nhiều thiết bị.

---

## 6. Cấu hình SMTP trong ứng dụng

```env
SMTP_HOST=smtp.gmail.com         # Địa chỉ server SMTP
SMTP_PORT=587                    # Cổng kết nối
SMTP_USER=your@gmail.com         # Tài khoản
SMTP_PASS=xxxx xxxx xxxx xxxx   # App Password (không phải mật khẩu thật)
```

### SMTP_HOST phổ biến:
| Nhà cung cấp | SMTP_HOST | Port |
|---|---|---|
| Gmail | smtp.gmail.com | 587 |
| Outlook | smtp.office365.com | 587 |
| Yahoo | smtp.mail.yahoo.com | 587 |
| AWS SES | email-smtp.us-east-1.amazonaws.com | 587 |

### App Password:
- Tài khoản bật **2FA** → không thể dùng mật khẩu thường qua SMTP
- Tạo tại: Google Account → Security → App Passwords
- Chỉ có **Name** (tên để nhận biết) và **Password (token)**
- Token chỉ hiển thị **một lần duy nhất** — phải copy ngay
- Có thể thu hồi bất cứ lúc nào mà không ảnh hưởng mật khẩu chính

---

## 7. AWS SES

AWS SES (Simple Email Service) = dịch vụ SMTP chuyên nghiệp của Amazon, thay thế Gmail SMTP khi cần gửi email số lượng lớn trong production.

| | Gmail SMTP | AWS SES |
|---|---|---|
| Giới hạn | ~500 email/ngày | Hàng triệu/ngày |
| Mục đích | Test, cá nhân | Production |
| Giá | Miễn phí | ~$0.10/1000 email |
| Độ tin cậy | Trung bình | Cao (ít vào Spam) |

### Lấy SMTP credentials AWS SES:
1. AWS Console → Simple Email Service
2. SMTP Settings → **Create SMTP credentials**
3. Đặt tên IAM user → Create
4. Copy **SMTP username** và **SMTP password** (chỉ hiện **1 lần duy nhất**)

> **Lưu ý**: SMTP credentials của SES **khác** với AWS Access Key — phải tạo riêng từ SES Console

---

## 8. SSL và TLS

SSL/TLS là giao thức **mã hóa dữ liệu** khi truyền qua mạng — kẻ tấn công không đọc được nội dung dù bắt được gói tin.

| Phiên bản | Năm | Trạng thái |
|---|---|---|
| SSL 2.0, 3.0 | 1994–1996 | Lỗi thời, có lỗ hổng |
| TLS 1.0, 1.1 | 1999–2006 | Không khuyến nghị |
| TLS 1.2 | 2008 | Vẫn dùng phổ biến |
| **TLS 1.3** | 2018 | **Hiện đại nhất** |

> Ngày nay **SSL đã lỗi thời** — thực tế đang dùng **TLS** nhưng vẫn hay gọi là "SSL" vì quen miệng (vd: "SSL certificate").

---

## 9. SMTP Port — Giải thích chi tiết

## Có bao nhiêu port SMTP?

Có **3 port chính** được dùng cho SMTP:

| Port | Tên | Trạng thái |
|---|---|---|
| **25** | SMTP gốc | Cũ, bị ISP/AWS chặn |
| **465** | SMTPS (SMTP over SSL) | Không còn chuẩn RFC, vẫn hoạt động |
| **587** | SMTP Submission + STARTTLS | **Hiện đại, khuyến nghị** |

---

## Port 25 — SMTP gốc (1982)

```
Client ──────────────────► SMTP Server :25
       (không mã hóa)
```

### Đặc điểm:
- Giao thức SMTP **đầu tiên**, ra đời từ năm 1982
- **Không mã hóa** — dữ liệu truyền dạng plain text, ai cũng đọc được
- Thiết kế để **server gửi thư cho server** (MTA to MTA), không phải cho client

### Vấn đề:
- **ISP (nhà mạng) chặn port 25 outbound** từ mạng gia đình và doanh nghiệp
  - Lý do: ngăn máy tính cá nhân bị malware biến thành **spam bot** gửi hàng triệu thư rác
- **AWS, Google Cloud, Azure cũng chặn** port 25 mặc định trên các máy EC2/VM
  - Phải gửi yêu cầu lên AWS mới được mở, và thường bị từ chối
- Không yêu cầu xác thực → ai cũng gửi được → dễ bị lợi dụng

### Minh họa bị chặn:
```
Máy bạn ──port 25──► ISP phát hiện ──► CHẶN ✗
Máy bạn ──port 587─► ISP cho qua  ──► Gmail ✓
```

### Khi nào còn dùng port 25?
- Chỉ dùng giữa **mail server với mail server** trong hạ tầng nội bộ
- Ví dụ: Gmail server gửi thư đến Outlook server — đây là server-to-server, không qua ISP thông thường
- **Không bao giờ dùng** trong ứng dụng để gửi email cho người dùng

---

## Port 465 — SMTPS / SSL (1998)

```
Client ══════════════════► SMTP Server :465
       (SSL ngay từ đầu, toàn bộ phiên được mã hóa)
```

### Đặc điểm:
- Kết nối được **bọc SSL ngay lập tức** từ byte đầu tiên
- Mã hóa **toàn bộ** phiên làm việc trước khi trao đổi bất kỳ dữ liệu nào
- Từng được IANA (tổ chức quản lý port) chính thức assign năm 1998

### Lịch sử và vấn đề:
- Năm 1998, sau khi port 465 được dùng cho SMTP/SSL, IANA **thu hồi** và assign lại cho mục đích khác (URD protocol)
- RFC (tiêu chuẩn Internet) chính thức **không còn công nhận** port 465 cho SMTP
- Tuy nhiên nhiều nhà cung cấp (Gmail, Yahoo, Outlook...) **vẫn hỗ trợ** vì lý do tương thích ngược với hệ thống cũ

### RFC không công nhận nhưng vẫn gửi được — tại sao?
- RFC chỉ là **tài liệu tiêu chuẩn/khuyến nghị**, không phải luật bắt buộc
- Các nhà cung cấp **tự chọn** giữ port 465 hoạt động để không làm hỏng hệ thống cũ đang chạy
- Giống như đường mòn không có trong bản đồ chính thức — nhưng người ta vẫn đi được

### Khi nào dùng port 465?
- Dịch vụ yêu cầu SSL bắt buộc và không hỗ trợ 587
- Legacy system (hệ thống cũ được xây dựng trước 2006)
- Một số dịch vụ email doanh nghiệp cũ

---

## Port 587 — SMTP Submission + STARTTLS (chuẩn hóa 2006-2011)

```
Bước 1: Kết nối plain text
Client ──────────────────► SMTP Server :587

Bước 2: Client gửi lệnh STARTTLS
Client ── "STARTTLS" ────► Server

Bước 3: Nâng cấp lên TLS
Client ══════════════════► Server (mã hóa TLS từ đây trở đi)

Bước 4: Xác thực và gửi thư
Client ── AUTH LOGIN ════► Server (username + password, đã mã hóa)
Client ── MAIL FROM ════► Server
Client ── DATA ══════════► Server (nội dung email, đã mã hóa)
```

### Đặc điểm kỹ thuật:
- Kết nối **bắt đầu plain text** → sau đó client gửi lệnh `STARTTLS` → **nâng cấp lên TLS**
- Được **RFC 4409 (2006)** và **RFC 6409 (2011)** chính thức quy định cho email submission
- **Bắt buộc xác thực** (AUTH) trước khi được phép gửi thư → chống spam hiệu quả
- Được hỗ trợ bởi **tất cả** nhà cung cấp lớn: Gmail, Outlook, Yahoo, AWS SES...

### Tại sao chọn Port 587?

**1. Được RFC chính thức công nhận**
- RFC 6409 quy định rõ port 587 là cổng dành cho **mail submission** (client gửi thư lên server)
- Đây là tiêu chuẩn được cộng đồng Internet đồng thuận, không như port 465

**2. Không bị ISP chặn**
- ISP chặn port 25 vì lý do chống spam, nhưng **không chặn port 587**
- Port 587 yêu cầu xác thực → ISP tin tưởng hơn
- AWS, Google Cloud, Azure đều cho phép traffic qua port 587 mặc định

**3. Bảo mật tốt nhờ TLS (STARTTLS)**
- Dữ liệu được mã hóa trước khi gửi username, password và nội dung thư
- Kẻ tấn công bắt được gói tin cũng không đọc được nội dung
- TLS 1.2 và 1.3 là các thuật toán mã hóa mạnh nhất hiện nay

**4. Hỗ trợ bởi tất cả nhà cung cấp lớn**

| Nhà cung cấp | SMTP Host | Port |
|---|---|---|
| Gmail | smtp.gmail.com | **587** |
| Outlook | smtp.office365.com | **587** |
| Yahoo | smtp.mail.yahoo.com | **587** |
| AWS SES | email-smtp.us-east-1.amazonaws.com | **587** |
| Mailtrap | sandbox.smtp.mailtrap.io | **587** |

### Cấu hình trong code với port 587:

```js
// Node.js - Nodemailer
const transporter = nodemailer.createTransport({
  host: 'smtp.gmail.com',
  port: 587,
  secure: false,   // false = dùng STARTTLS (nâng cấp lên TLS sau khi kết nối)
  auth: {
    user: 'your@gmail.com',
    pass: 'app-password-here'
  }
});
```

> **Lưu ý:** `secure: false` với port 587 **không có nghĩa là không bảo mật** — nó có nghĩa là "bắt đầu plain text rồi STARTTLS", khác với `secure: true` (SSL ngay từ đầu, dùng cho port 465)

---

## So sánh tổng hợp

| Tiêu chí | Port 25 | Port 465 | Port 587 |
|---|---|---|---|
| Ra đời | 1982 | 1998 | 1998, chuẩn hóa 2006 |
| RFC công nhận | Có (cũ) | Không | **Có (mới nhất)** |
| Bị ISP chặn | **Thường xuyên** | Ít | Hiếm khi |
| Bị AWS chặn | **Mặc định** | Không | Không |
| Mã hóa | Không | SSL (ngay từ đầu) | **TLS (STARTTLS)** |
| Xác thực bắt buộc | Không | Có | **Có** |
| Dùng cho | Server-to-server | Legacy system | **Ứng dụng hiện đại** |
| Khuyến nghị | Không | Không | **Có** |

---

## Kết luận

> - **Port 25**: Chỉ dành cho mail server nội bộ, **không dùng từ ứng dụng** vì bị chặn khắp nơi
> - **Port 465**: Vẫn hoạt động nhưng **không phải chuẩn chính thức**, chỉ dùng khi bắt buộc
> - **Port 587**: Chuẩn hiện đại, bảo mật, không bị chặn → **luôn chọn cái này** cho mọi ứng dụng
