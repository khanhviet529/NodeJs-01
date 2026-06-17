Tài liệu học CORS, Cookie, SameSite và CSRF
1. Bức tranh tổng quan
Ngày nay Frontend và Backend thường được tách riêng:
```txt
FE: https://app.example.com
BE: https://api.example.com
```
Khi FE gọi API sang BE khác `origin`, trình duyệt sẽ coi đây là cross-origin request.
`Origin` được xác định bởi 3 phần:
```txt
protocol + domain + port
```
Ví dụ:
```txt
https://example.com:443
```
Trong đó:
```txt
protocol = https
host/domain = example.com
port = 443
```
Chỉ cần khác 1 trong 3 phần trên thì là khác origin.
Ví dụ khác origin:
```txt
https://example.com
http://example.com
```
Khác `protocol`.
```txt
https://app.example.com
https://api.example.com
```
Khác `domain/subdomain`.
```txt
http://localhost:3000
http://localhost:8080
```
Khác `port`.
---
2. Same-Origin Policy là gì?
Same-Origin Policy là cơ chế bảo mật của trình duyệt.
Mặc định, JavaScript chạy ở website A không được tự do đọc dữ liệu từ website B nếu khác origin.
Ví dụ:
```txt
Website đang mở: https://evil.com
API muốn gọi:     https://bank.com/api/user
```
Nếu không có cơ chế bảo vệ, `evil.com` có thể gọi API của `bank.com` và đọc thông tin user.
Vì vậy trình duyệt cần một cơ chế kiểm soát việc đọc response giữa các origin khác nhau.
Cơ chế đó là CORS.
---
3. CORS là gì?
CORS là viết tắt của Cross-Origin Resource Sharing.
Nói ngắn gọn:
```txt
CORS là cơ chế để server khai báo origin nào được phép đọc response của server trên trình duyệt.
```
Điểm quan trọng:
```txt
CORS chủ yếu kiểm soát việc JavaScript có được đọc response hay không.
CORS không phải cơ chế xác thực user.
CORS không thay thế CSRF protection.
```
Ví dụ:
```txt
FE: https://app.example.com
BE: https://api.example.com
```
Nếu BE cho phép FE đọc response, BE trả header:
```http
Access-Control-Allow-Origin: https://app.example.com
```
Nếu không có header CORS hợp lệ, request có thể vẫn được gửi tới BE, BE vẫn có thể xử lý, nhưng JavaScript ở FE sẽ không đọc được response.
---
4. Flow CORS với simple request
Ví dụ FE gọi:
```js
fetch('https://api.example.com/users')
```
Browser gửi request thật lên BE:
```http
GET /users HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
```
BE trả response:
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Content-Type: application/json

{
  "id": 1,
  "name": "Khanh"
}
```
Browser kiểm tra:
```txt
Origin của FE: https://app.example.com
BE allow:      https://app.example.com
=> Hợp lệ
=> JavaScript được đọc response
```
Nếu BE trả:
```http
Access-Control-Allow-Origin: https://other-site.com
```
thì browser chặn JavaScript đọc response.
---
5. Simple request và Preflight request
Không phải cứ `GET/POST` là không preflight, và cũng không phải chỉ `PUT/PATCH/DELETE` mới preflight.
Cách hiểu đúng:
```txt
Simple request    => Browser gửi request thật luôn.
Non-simple request => Browser gửi OPTIONS preflight trước.
```
5.1. Simple request thường là gì?
Thường là các request đơn giản như:
```txt
GET
HEAD
POST với Content-Type đơn giản
```
Một số `Content-Type` đơn giản:
```txt
application/x-www-form-urlencoded
multipart/form-data
text/plain
```
5.2. Khi nào bị preflight?
Request thường bị preflight nếu có:
```txt
PUT / PATCH / DELETE
Authorization header
Content-Type: application/json
Custom header như X-CSRF-Token
```
Ví dụ request này thường bị preflight:
```js
fetch('https://api.example.com/users/1', {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token'
  },
  body: JSON.stringify({ name: 'Khanh' })
})
```
---
6. Flow Preflight OPTIONS
FE muốn gửi request thật:
```http
PATCH https://api.example.com/users/1
```
Browser gửi OPTIONS trước:
```http
OPTIONS /users/1 HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: Content-Type, Authorization
```
BE kiểm tra origin, method, headers.
Nếu hợp lệ, BE trả:
```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PATCH, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```
Browser thấy hợp lệ thì mới gửi request thật:
```http
PATCH /users/1 HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Content-Type: application/json
Authorization: Bearer token

{
  "name": "Khanh"
}
```
Nếu preflight không hợp lệ:
```txt
Browser không gửi request thật.
JavaScript nhận lỗi CORS.
```
---
7. Các CORS header quan trọng
7.1. Access-Control-Allow-Origin
Cho biết origin nào được phép đọc response.
```http
Access-Control-Allow-Origin: https://app.example.com
```
Có thể dùng `*` nếu API public và không dùng credentials:
```http
Access-Control-Allow-Origin: *
```
Nhưng nếu có cookie/credentials thì không được dùng `*`.
---
7.2. Access-Control-Allow-Methods
Cho biết method nào được phép dùng trong cross-origin request.
```http
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE
```
---
7.3. Access-Control-Allow-Headers
Cho biết request được phép dùng những header nào.
```http
Access-Control-Allow-Headers: Content-Type, Authorization, X-CSRF-Token
```
---
7.4. Access-Control-Allow-Credentials
Cho phép request cross-origin có credentials như cookie.
```http
Access-Control-Allow-Credentials: true
```
Khi dùng credentials, BE phải allow origin cụ thể:
```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```
Không hợp lệ:
```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```
---
8. Cookie hoạt động như thế nào?
Cookie là dữ liệu server yêu cầu browser lưu lại bằng header `Set-Cookie`.
Ví dụ BE trả:
```http
Set-Cookie: access_token=jwt_access_token; HttpOnly; Secure; SameSite=Lax
Set-Cookie: refresh_token=random_refresh_token; HttpOnly; Secure; SameSite=Lax
```
Browser lưu cookie theo domain/path.
Khi browser gửi request đến domain khớp với cookie, browser có thể tự đính kèm cookie vào request.
Ví dụ cookie thuộc:
```txt
api.example.com
```
Request đến:
```txt
https://api.example.com/users
```
Browser kiểm tra:
```txt
Domain có khớp không?
Path có khớp không?
Có Secure mà request có HTTPS không?
SameSite có cho phép gửi không?
Request cross-origin có credentials không?
```
Nếu thỏa điều kiện, browser tự gắn:
```http
Cookie: access_token=jwt_access_token; refresh_token=random_refresh_token
```
Điểm quan trọng:
```txt
JavaScript không tự gắn HttpOnly cookie.
JavaScript cũng không đọc được HttpOnly cookie.
Browser mới là bên tự gửi cookie nếu điều kiện cho phép.
```
---
9. Cookie không lưu riêng theo từng tab
Cookie không phải mỗi tab có một bản riêng.
Cookie thường được lưu theo:
```txt
browser profile + domain/site
```
Ví dụ tab 1 login:
```txt
https://bank.com
```
Browser lưu cookie `bank_session` cho `bank.com`.
Sau đó tab 2 mở:
```txt
https://bank.com/profile
```
Tab 2 vẫn có trạng thái login vì browser dùng chung cookie `bank.com`.
Website khác như `evil.com` không đọc được cookie của `bank.com`, nhưng có thể yêu cầu browser gửi request đến `bank.com`. Khi request đích là `bank.com`, browser sẽ xét cookie của `bank.com`.
---
10. credentials: include và withCredentials
Nếu FE và BE khác origin, và cần gửi cookie, FE phải bật credentials.
Với `fetch`:
```js
fetch('https://api.example.com/user', {
  credentials: 'include'
})
```
Với Axios:
```js
axios.get('https://api.example.com/user', {
  withCredentials: true
})
```
Ý nghĩa:
```txt
FE yêu cầu browser gửi credentials như cookie trong cross-origin request nếu cookie đủ điều kiện.
```
BE phải cấu hình:
```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```
Nếu BE dùng:
```http
Access-Control-Allow-Origin: *
```
thì với request có credentials, browser sẽ không cho JavaScript đọc response.
---
11. SameSite là gì?
`SameSite` là thuộc tính của cookie, dùng để quy định cookie có được gửi trong request đến từ site khác hay không.
Ví dụ:
```http
Set-Cookie: session_id=abc; HttpOnly; Secure; SameSite=Lax
```
Có 3 giá trị chính:
```txt
Strict
Lax
None
```
---
12. SameSite=Strict
```http
Set-Cookie: session_id=abc; HttpOnly; Secure; SameSite=Strict
```
Ý nghĩa:
```txt
Chỉ gửi cookie trong request cùng site.
```
Ví dụ user đang ở `google.com`, click link sang:
```txt
https://bank.com/profile
```
Với `SameSite=Strict`, browser không gửi cookie `bank.com` trong request điều hướng từ site khác sang.
Kết quả có thể là:
```txt
bank.com không thấy cookie
=> tưởng user chưa login
=> chuyển về login
```
`Strict` chống CSRF mạnh hơn nhưng có thể ảnh hưởng trải nghiệm khi user đi từ link bên ngoài vào website.
---
13. SameSite=Lax
```http
Set-Cookie: session_id=abc; HttpOnly; Secure; SameSite=Lax
```
Ý nghĩa:
```txt
Cookie được gửi trong request cùng site.
Cookie cũng có thể được gửi khi user điều hướng top-level GET từ site khác sang.
Cookie thường không được gửi trong request cross-site nguy hiểm như POST bằng fetch/form.
```
Ví dụ user click link từ Google sang:
```txt
https://bank.com/profile
```
Với `Lax`, browser có thể gửi cookie, nên user vẫn giữ trạng thái login.
Nhưng nếu `evil.com` gọi:
```js
fetch('https://bank.com/api/transfer', {
  method: 'POST',
  credentials: 'include'
})
```
Với `Lax`, cookie thường không được gửi.
---
14. SameSite=None
```http
Set-Cookie: session_id=abc; HttpOnly; Secure; SameSite=None
```
Ý nghĩa:
```txt
Cho phép gửi cookie trong cross-site request.
```
`SameSite=None` bắt buộc đi kèm `Secure`, tức là cần HTTPS.
Dùng khi FE và BE khác site thật sự:
```txt
FE: https://frontend.com
BE: https://api-backend.com
```
Nếu dùng `SameSite=None`, cần đặc biệt chú ý chống CSRF bằng:
```txt
CSRF token
Origin/Referer check
CORS allow đúng FE origin
```
---
15. SameSite khác Same-Origin
`Same-Origin` xét:
```txt
protocol + domain + port
```
`SameSite` xét theo site chính.
Ví dụ:
```txt
https://app.example.com
https://api.example.com
```
Hai URL này:
```txt
Khác origin
Nhưng có thể cùng site example.com
```
Vì vậy cần phân biệt:
```txt
CORS xử lý cross-origin.
SameSite xử lý việc cookie có được gửi trong same-site/cross-site request hay không.
```
---
16. CSRF là gì?
CSRF là viết tắt của Cross-Site Request Forgery.
Nói ngắn gọn:
```txt
CSRF là lỗi website xấu lợi dụng browser của user đang đăng nhập để gửi request không mong muốn đến server.
```
Ví dụ:
```txt
User đã login bank.com
Browser có cookie bank.com
User mở evil.com
evil.com tạo request POST đến bank.com/transfer
Browser có thể tự gửi cookie bank.com kèm request
bank.com tưởng request là user thật gửi
```
Điểm quan trọng:
```txt
evil.com không cần đọc cookie bank.com.
evil.com chỉ cần lợi dụng browser tự gửi cookie bank.com.
```
---
17. Ví dụ tấn công CSRF
Trang `evil.com` có thể có form:
```html
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker" />
  <input name="amount" value="1000" />
</form>

<script>
  document.forms[0].submit()
</script>
```
Nếu user đã login `bank.com`, và cookie được gửi trong request này, server `bank.com` có thể hiểu nhầm rằng user tự gửi request chuyển tiền.
CSRF nguy hiểm với các API thay đổi dữ liệu:
```txt
POST /transfer-money
POST /change-password
PUT /profile
DELETE /account
POST /create-order
```
Không nên dùng `GET` cho hành động thay đổi dữ liệu.
---
18. CORS và CSRF khác nhau thế nào?
Tiêu chí	CORS	CSRF
Mục tiêu	Kiểm soát JS từ origin khác có được đọc response không	Chống website khác lợi dụng cookie/session để gửi request giả mạo
Bảo vệ chính	Dữ liệu response	Hành động thay đổi dữ liệu trên server
Có chặn request gửi đi không?	Không đảm bảo	Có thể chặn bằng token/origin/samesite
Liên quan cookie?	Có khi dùng credentials	Rất liên quan nếu auth dùng cookie/session
Cách xử lý	CORS headers	SameSite, CSRF token, Origin/Referer check
Câu dễ nhớ:
```txt
CORS chống đọc trộm response.
CSRF chống gửi request giả mạo bằng cookie/session của user.
```
---
19. Chống CSRF bằng SameSite
Server set cookie:
```http
Set-Cookie: session_id=abc; HttpOnly; Secure; SameSite=Lax
```
Khi website xấu gửi request cross-site kiểu POST:
```js
fetch('https://bank.com/transfer', {
  method: 'POST',
  credentials: 'include'
})
```
Browser kiểm tra `SameSite=Lax` và thường không gửi cookie.
Kết quả:
```txt
Request đến bank.com nhưng không có session cookie
=> bank.com không xác thực được user
=> request bị từ chối
```
SameSite là cách đơn giản và hiệu quả để giảm rủi ro CSRF, nhưng không nên coi là lớp bảo vệ duy nhất cho hệ thống quan trọng.
---
20. Chống CSRF bằng CSRF token
Flow tổng quát:
```txt
User login
BE set cookie auth HttpOnly
BE cấp CSRF token
FE lưu CSRF token trong memory/store
FE gửi CSRF token trong header khi gọi POST/PUT/PATCH/DELETE
BE kiểm tra cookie + CSRF token
```
Ví dụ BE trả khi login:
```json
{
  "user": {
    "id": 1,
    "name": "Khanh"
  },
  "csrfToken": "csrf_abc_123"
}
```
FE gửi request:
```js
axios.post(
  'https://api.example.com/orders',
  { productId: 1, quantity: 2 },
  {
    withCredentials: true,
    headers: {
      'X-CSRF-Token': csrfToken
    }
  }
)
```
Request thực tế:
```http
POST /orders HTTP/1.1
Host: api.example.com
Cookie: access_token=jwt
X-CSRF-Token: csrf_abc_123
Content-Type: application/json
```
BE kiểm tra:
```txt
Cookie auth hợp lệ không?
X-CSRF-Token có tồn tại không?
CSRF token có đúng không?
```
Nếu thiếu hoặc sai token:
```http
403 Forbidden
```
---
21. Chống CSRF bằng Origin / Referer check
Với các request thay đổi dữ liệu, browser thường gửi header `Origin`.
Request hợp lệ:
```http
POST /orders HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Cookie: access_token=jwt
```
Request đáng ngờ:
```http
POST /orders HTTP/1.1
Host: api.example.com
Origin: https://evil.com
Cookie: access_token=jwt
```
BE kiểm tra:
```txt
Origin có nằm trong danh sách FE hợp lệ không?
Nếu không => reject 403
```
Ví dụ logic:
```js
const allowedOrigins = ['https://app.example.com']

function csrfOriginCheck(req, res, next) {
  const unsafeMethods = ['POST', 'PUT', 'PATCH', 'DELETE']

  if (!unsafeMethods.includes(req.method)) {
    return next()
  }

  const origin = req.headers.origin

  if (!origin || !allowedOrigins.includes(origin)) {
    return res.status(403).json({ message: 'Invalid origin' })
  }

  next()
}
```
`Referer` có thể dùng làm fallback, nhưng thường ưu tiên `Origin` vì rõ ràng hơn.
---
22. Có cần kết hợp SameSite + CSRF token + Origin check không?
Câu trả lời thực tế:
```txt
Có thể kết hợp nhiều lớp để an toàn hơn.
```
Vai trò từng lớp:
```txt
SameSite
=> Hạn chế browser gửi cookie từ site lạ.

CSRF token
=> Chứng minh request được tạo từ FE hợp lệ.

Origin/Referer check
=> BE kiểm tra nguồn request có đúng domain FE hợp lệ không.
```
Với hệ thống bình thường:
```txt
HttpOnly + Secure + SameSite=Lax + Origin check
```
Với hệ thống quan trọng như tài chính, thanh toán, đổi mật khẩu:
```txt
HttpOnly + Secure + SameSite=Lax/Strict + CSRF token + Origin check
```
Nếu bắt buộc dùng `SameSite=None` vì FE/BE khác site:
```txt
SameSite=None; Secure
+ CSRF token
+ Origin check
+ CORS allow đúng FE origin
```
---
23. Cấu hình Express CORS ví dụ
```js
import cors from 'cors'
import express from 'express'

const app = express()

app.use(cors({
  origin: 'https://app.example.com',
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token']
}))
```
Nếu không dùng cookie/credentials, API public có thể dùng:
```js
app.use(cors({
  origin: '*'
}))
```
Nhưng khi có cookie:
```js
app.use(cors({
  origin: 'https://app.example.com',
  credentials: true
}))
```
Không dùng:
```js
app.use(cors({
  origin: '*',
  credentials: true
}))
```
---
24. Cấu hình Axios ví dụ
```js
import axios from 'axios'

const api = axios.create({
  baseURL: 'https://api.example.com',
  withCredentials: true
})

api.interceptors.request.use((config) => {
  const csrfToken = authStore.csrfToken

  if (csrfToken) {
    config.headers['X-CSRF-Token'] = csrfToken
  }

  return config
})
```
Ý nghĩa:
```txt
withCredentials: true
=> Cho phép browser gửi cookie cross-origin nếu cookie đủ điều kiện.

X-CSRF-Token
=> Token chống CSRF do FE gửi lên để BE kiểm tra.
```
---
25. Flow đầy đủ: Cookie + CORS + CSRF token
Login
```txt
1. FE gọi POST /login
2. BE xác thực user
3. BE set HttpOnly cookie access_token / refresh_token
4. BE trả user info + csrfToken
5. FE lưu csrfToken trong memory/store
```
Response:
```http
Set-Cookie: access_token=jwt; HttpOnly; Secure; SameSite=Lax
Set-Cookie: refresh_token=random; HttpOnly; Secure; SameSite=Lax
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```
Body:
```json
{
  "user": {
    "id": 1,
    "name": "Khanh"
  },
  "csrfToken": "csrf_abc_123"
}
```
Gọi API thay đổi dữ liệu
FE:
```js
api.post('/orders', data, {
  headers: {
    'X-CSRF-Token': csrfToken
  }
})
```
Request:
```http
POST /orders HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Cookie: access_token=jwt
X-CSRF-Token: csrf_abc_123
```
BE kiểm tra:
```txt
Cookie access_token hợp lệ
Origin hợp lệ
CSRF token hợp lệ
```
Nếu hợp lệ:
```txt
Xử lý request
```
Nếu không hợp lệ:
```txt
403 Forbidden
```
---
26. Các hiểu nhầm thường gặp
Hiểu nhầm 1: CORS chặn request gửi lên server
Không chính xác.
```txt
CORS chủ yếu chặn JavaScript đọc response.
Một số request vẫn có thể được gửi đến server.
Riêng preflight fail thì browser không gửi request thật.
```
---
Hiểu nhầm 2: Cookie là riêng từng tab
Không chính xác.
```txt
Cookie thường dùng chung trong cùng browser profile và theo domain/site.
```
---
Hiểu nhầm 3: HttpOnly cookie chống được CSRF
Không đủ.
```txt
HttpOnly ngăn JavaScript đọc cookie.
Nhưng browser vẫn có thể tự gửi cookie.
Vì vậy vẫn cần SameSite / CSRF token / Origin check.
```
---
Hiểu nhầm 4: CORS thay thế CSRF protection
Không chính xác.
```txt
CORS kiểm soát đọc response.
CSRF protection kiểm soát request giả mạo gây thay đổi dữ liệu.
```
---
Hiểu nhầm 5: GET/POST luôn không preflight
Không chính xác.
```txt
GET/POST vẫn có thể preflight nếu có custom header, Authorization, hoặc Content-Type không đơn giản như application/json.
```
---
27. Câu trả lời phỏng vấn ngắn gọn
```txt
CORS là cơ chế để server kiểm soát origin nào được phép đọc response của server trên trình duyệt. Khi FE và BE khác origin, browser gửi Origin header, BE trả các CORS header như Access-Control-Allow-Origin, Access-Control-Allow-Methods, Access-Control-Allow-Headers. Nếu request là non-simple, browser sẽ gửi OPTIONS preflight trước để kiểm tra server có cho phép method và header đó không.

Nếu request cần gửi cookie cross-origin, FE phải bật credentials: include hoặc axios withCredentials: true. BE phải trả Access-Control-Allow-Credentials: true và Access-Control-Allow-Origin phải là origin cụ thể, không được dùng *.

CSRF là lỗi website khác lợi dụng browser của user đang đăng nhập để gửi request kèm cookie/session đến server. CSRF không cần đọc cookie, nó chỉ lợi dụng việc browser tự gửi cookie. Cách chống gồm SameSite cookie, CSRF token và kiểm tra Origin/Referer. SameSite giúp hạn chế cookie được gửi từ site lạ, CSRF token chứng minh request được tạo từ FE hợp lệ, còn Origin check giúp BE chặn request đến từ domain không hợp lệ.
```
---
28. Checklist triển khai thực tế
Nếu dùng cookie để auth:
```txt
[ ] Cookie có HttpOnly
[ ] Cookie có Secure khi chạy HTTPS
[ ] Cookie có SameSite=Lax hoặc Strict nếu có thể
[ ] Nếu SameSite=None thì bắt buộc Secure
[ ] FE dùng withCredentials/credentials khi cross-origin cần cookie
[ ] BE Access-Control-Allow-Origin là domain FE cụ thể
[ ] BE Access-Control-Allow-Credentials: true
[ ] Không dùng Access-Control-Allow-Origin: * với credentials
[ ] API POST/PUT/PATCH/DELETE có CSRF token hoặc Origin check
[ ] Không dùng GET cho hành động thay đổi dữ liệu
[ ] Refresh token không lưu ở localStorage nếu có thể tránh
```
---
29. Tài liệu tham khảo
MDN Web Docs — Cross-Origin Resource Sharing: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
MDN Web Docs — Set-Cookie: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie
MDN Web Docs — Using HTTP cookies: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies
OWASP Cheat Sheet — Cross-Site Request Forgery Prevention: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
