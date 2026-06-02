# **KIẾN TRÚC XỬ LÝ & LƯU TRỮ FILE GIỮA FRONTEND VÀ BACKEND**

*Tài liệu tổng hợp chuyên sâu từ lý thuyết mạng đến thực thi hệ thống lớn*

## **PHẦN 1: TỔNG QUAN HỆ THỐNG MẠNG & TÀI NGUYÊN (RECAP)**

Trước khi đi vào các Case cụ thể, chúng ta cần thống nhất 4 nguyên lý vận hành cốt lõi của mạng và máy chủ:

1. **Băng thông (Bandwidth):** Là tốc độ tối đa của "đường ống mạng" tại một thời điểm (đo bằng Mbps).  
   * Công thức quy đổi tốc độ tải file thực tế:  
     ![][image1]  
     *Ví dụ: Đường truyền 100 Mbps sẽ tải tối đa được 12.5 MB/s trong điều kiện lý tưởng.*  
2. **Traffic & Data Transfer:** Traffic (Số lượng request/giây) tỷ lệ thuận với Data Transfer (Tổng lượng byte truyền qua mạng). Khi nhiều request đồng thời, băng thông Server sẽ tự động **chia đều** cho các kết nối.  
3. **Mã hóa Base64:** Chuyển đổi dữ liệu nhị phân (Binary) sang dạng ký tự văn bản an toàn để truyền qua JSON. Quá trình này **bắt buộc làm phình to file thêm đúng 33.3%** về mặt vật lý (do quy đổi từ cụm 8-bit sang cụm 6-bit).  
4. **Buffer vs Stream:** \* **Buffer:** Nhồi toàn bộ file vào RAM rồi mới xử lý ➡️ Rất dễ gây tràn RAM và kích hoạt **OOM Killer** sập nguồn Backend nếu gặp file nặng.  
   * **Stream:** Chia nhỏ file thành các mảnh (Chunks) cực nhỏ (tầm 16 KB \- 64 KB), nhận đến đâu ghi nối đuôi (Append) xuống ổ cứng nhờ con trỏ vị trí (File Pointer) của Hệ điều hành đến đó ➡️ Giải phóng RAM ngay lập tức, an toàn tuyệt đối.

## **PHẦN 2: PHÂN LOẠI CHI TIẾT CÁC CASE XỬ LÝ UPLOAD FILE**

Dưới đây là 3 Case lớn đại diện cho toàn bộ các kịch bản thực tế trong phát triển phần mềm:

### **🌟 BẢNG SO SÁNH TỔNG QUAN CÁC CASE**

| Phương pháp | Độ phức tạp | Tốn RAM BE | Tốn Băng thông BE | Chống rớt mạng | Khả năng Scale |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Cách A.1: Base64 in JSON** | 🟢 Rất dễ | 🔴 Rất cao | 🔴 Cực cao (+33%) | ❌ Không | ❌ Rất kém |
| **Cách A.2: Multipart Form-Data** | 🟢 Rất dễ | 🟡 Vừa phải | 🔴 Cao (Bằng file x 2\) | ❌ Không | 🟡 Vừa phải |
| **Cách B.1: Multipart Stream** | 🟡 Trung bình | 🟢 Rất thấp | 🔴 Cao (Bằng file x 2\) | ❌ Không | 🟡 Vừa phải |
| **Cách B.2: Chunk-Upload** | 🔴 Rất khó | 🟢 Rất thấp | 🔴 Cao (Bằng file x 2\) | Tuyệt vời | 🟡 Vừa phải |
| **Cách C.1: Presigned URL** | 🟡 Trung bình | 🟢 Cực thấp | 🟢 Gần như bằng 0 | ❌ Không | Vô hạn |

### **CASE A: UPLOAD FILE NHẸ & VỪA (DƯỚI 100MB)**

#### **Cách A.1: Mã hóa Base64 nhét trong JSON**

*FE encode file ảnh thành chuỗi String Base64, nhét vào thuộc tính JSON gửi lên BE qua API thông thường.*

* **Ưu điểm:** Triển khai siêu nhanh, siêu dễ. Không cần cấu hình giao thức multipart, chỉ dùng API JSON tiêu chuẩn.  
* **Nhược điểm:** Làm phình to dung lượng file lên 33.3%, gây áp lực cực kỳ lớn lên RAM và CPU của Backend khi thực hiện giải mã (Decode).  
* **Khi nào nên dùng:** Chỉ dùng cho các file siêu nhẹ như icon, ảnh đại diện thumbnail kích thước cực nhỏ (\< 50 KB) hoặc khi làm các dự án thử nghiệm (MVP) cần code nhanh.

#### **Cách A.2: Multipart/Form-Data truyền thống (Ghi đĩa trực tiếp)**

*FE bọc file ảnh trong đối tượng FormData và gửi lên BE. BE dùng các thư viện middleware (như Multer) để hứng dữ liệu nhị phân gốc và ghi ra thư mục tạm trên ổ cứng.*

* **Sơ đồ luồng:**  
  \[ Trình duyệt FE \] ──(Multipart)──\> \[ Server BE \] ──(Ghi đĩa)──\> \[ Ổ cứng Local \]

* **Ưu điểm:** Giữ nguyên dung lượng file nhị phân gốc, hỗ trợ gửi kèm theo dữ liệu dạng Text dễ dàng.  
* **Nhược điểm:** Tốn băng thông gấp đôi của Server BE (BE nhận file từ client rồi đẩy file đó ra đĩa hoặc cloud storage). Khó khăn cho việc mở rộng hệ thống (Scale ngang).

### **CASE B: UPLOAD FILE LỚN & SIÊU NẶNG (TỪ 500MB ĐẾN HÀNG CHỤC GB)**

#### **Cách B.1: Multipart Stream nguyên khối (1 Request duy nhất)**

*FE gửi 1 file lớn bằng duy nhất 1 request HTTP qua Multipart. BE sử dụng cơ chế lắng nghe sự kiện stream.pipe() hoặc on('data') để ghi cuốn chiếu dữ liệu nhị phân trực tiếp xuống đĩa.*

* **Ưu điểm:** Giữ cho dung lượng RAM của Backend cực kỳ an toàn. Code viết đơn giản hơn so với Chunk-upload.  
* **Nhược điểm:** Rủi ro rớt mạng cao (mất mạng 2 giây ở phút thứ 99% phải tải lại từ đầu). Dễ bị chặn hoặc timeout bởi các proxy trung gian như Nginx hoặc Cloudflare.

#### **Cách B.2: Chunk-Upload thủ công (Nhiều Request liên tục/song song)**

*FE sử dụng JavaScript (.slice()) cắt file lớn thành nhiều chunk nhỏ (ví dụ 5MB mỗi cục) và gửi song song kèm theo Metadata (fileId, chunkIndex, totalChunks). BE hứng song song lưu thành các file tạm .tmp trên ổ cứng, sau đó chạy vòng lặp tuần tự để gộp (Merge) chúng lại bằng Stream nội bộ.*

* **Sơ đồ luồng:**  
  \[ FE \] ──Chặt file thành 4 mảnh 5MB──\> Gửi song song (4 Requests) ──\> \[ BE \]  
                                                                           │  
  \[ BE \] ──Ghi song song 4 file tạm .tmp xuống đĩa cứng (RAM an toàn) ─────┘  
                             │  
  \[ BE \] ──Nhận đủ 4 file tạm ──\> Chạy Stream gộp tuần tự ──\> \[ File 20MB Hoàn chỉnh \]

* **Bản chất phần cứng:** Ở tầng vật lý, ổ cứng chỉ xử lý tuần tự. Tuy nhiên, dữ liệu qua internet truyền về không đồng đều (Interleaving). BE sử dụng **nhiều file tạm** hoặc **ghi theo vị trí định sẵn (Offset)** để đưa các gói tin xáo trộn mạng về đúng hàng ngũ tuần tự mà không làm nghẽn luồng ghi của hệ điều hành.  
* **Ưu điểm:** Chống rớt mạng tuyệt hảo (Resumable Upload \- hỏng mảnh nào gửi lại mảnh đó). Tốc độ nhanh nhờ gửi song song tối đa băng thông Client. Vượt qua giới hạn Timeout của Proxy.  
* **Nhược điểm:** Code cực kỳ phức tạp ở cả FE (chia file, quản lý hàng đợi gửi) lẫn BE (đọc ghi file tạm, ghép nối, dọn dẹp file tạm rác). Gây áp lực ghi đọc đĩa (Disk I/O) rất lớn lên máy chủ Backend.

### **CASE C: KIẾN TRÚC UPLOAD TỐI ƯU CHO HỆ THỐNG LỚN (SAAS, CLOUD-NATIVE)**

#### **Cách C.1: Presigned URL (FE tải trực tiếp lên Cloud Storage \- S3/Cloudinary)**

*BE đóng vai trò là "Cơ quan cấp phép": Chỉ kiểm tra quyền rồi sinh ra một đường dẫn có chữ ký giới hạn thời gian (Presigned URL) siêu nhẹ và trả về dạng JSON. FE nhận link này và dùng phương thức PUT để đẩy trực tiếp file nhị phân từ trình duyệt lên S3.*

* **Sơ đồ luồng:**  
  Bước 1: \[ FE \] ──(Xin cấp phép: Tên file, size)──\> \[ BE \] (Rất nhẹ)  
  Bước 2: \[ FE \] \<──(Trả về: Presigned URL)──────── \[ BE \] (Rất nhẹ)  
  Bước 3: \[ FE \] ──(Đẩy trực tiếp file nhị phân)──\> \[ AWS S3 Cloud \] (S3 gánh toàn bộ băng thông)

* **Ưu điểm:** BE nhàn hạ tuyệt đối (không tốn băng thông nhận/đẩy file, không tốn RAM/ổ cứng xử lý file). Khả năng mở rộng (Scale) vô hạn nhờ hạ tầng mạng khổng lồ của các ông lớn Cloud. An toàn bảo mật cực cao nhờ các thuật toán chữ ký số của Cloud.  
* **Nhược điểm:** Cấu hình phức tạp ở phía Cloud (IAM, CORS). FE phải xử lý luồng upload qua 2 bước request.

## **PHẦN 3: ĐI SÂU KIẾN TRÚC PRESIGNED URL (AWS S3)**

Phương pháp Presigned URL chính là chuẩn công nghiệp để tối ưu hóa tài nguyên cho hệ thống phân phối lớn. Dưới đây là phân tích chi tiết cơ chế vận hành:

### **1\. Phân biệt uploadUrl vs fileUrl (Bản chất nhị phân)**

Khi BE trả về thông tin cấp phép, nó thường cung cấp 2 loại đường dẫn có bản chất hoàn toàn khác nhau:

| Tiêu chí | uploadUrl (Link để GHI) | fileUrl (Link để ĐỌC / LƯU DB) |
| :---- | :---- | :---- |
| **Bản chất** | Link tạm thời chứa chữ ký bảo mật | Link gốc, cố định và sạch sẽ |
| **Cấu trúc** | https://.../name.png?X-Amz-Signature=abc... | https://.../name.png |
| **Hành động** | Dùng duy nhất cho lệnh Axios PUT/POST đưa file lên | Dùng hiển thị (\<img src\>) hoặc tải file về |
| **Tuổi thọ** | Cực kỳ ngắn (1 \- 5 phút do BE cấu hình) | Vĩnh viễn (Theo vòng đời của file) |
| **Cơ chế** | Đóng vai trò là "thẻ thông hành" dùng một lần | Địa chỉ nhà cố định của file trong kho |

**Lưu ý:** AWS S3 sau khi nhận file thành công chỉ trả về HTTP 200 OK chứ **không trả về fileUrl**. BE phải tính toán và trả về fileUrl này ngay từ Bước 2 dựa vào quy luật cộng chuỗi vì BE là người chủ động đặt tên file (Key) và Bucket. Nếu BE không trả về, FE bắt buộc phải dùng thuật toán gọt chuỗi JavaScript để cắt bỏ phần Signature sau dấu chấm hỏi ? của uploadUrl.

### **2\. Cơ chế S3 "đọc vị" ràng buộc từ Backend**

Backend tạo Presigned URL hoàn toàn **nội bộ (Offline)** bằng cách băm (Hash) các thông số ràng buộc với **Secret Access Key** của hệ thống bằng thuật toán **HMAC-SHA256**. S3 không cần BE báo cáo trước mà tự giải mã và kiểm tra gói tin của FE gửi lên dựa trên 4 yếu tố:

* **Kiểm tra Thời hạn (Expiration):** S3 lấy thời gian hiện tại trừ đi X-Amz-Date trên URL. Nếu khoảng cách lớn hơn X-Amz-Expires (ví dụ: 300 giây), S3 lập tức từ chối (HTTP 403 Request has expired).  
* **Kiểm tra Tên file & Đường dẫn (Object Key):** Object Key được khóa cứng trong URL chữ ký. Nếu FE tự ý sửa tên file trên URL, thuật toán băm lại của S3 sẽ cho ra kết quả không khớp với X-Amz-Signature trên link (HTTP 403 SignatureDoesNotMatch).  
* **Kiểm tra Định dạng file (Content-Type):** Được khóa cứng thông qua danh sách SignedHeaders. Khi FE upload qua Axios/Fetch, bắt buộc phải truyền Header Content-Type khớp 100% với định dạng mà BE đã ký ở Bước 1\. Sai lệch sẽ bị S3 block thẳng tay.  
* **Kiểm tra Dung lượng file (Content-Length):** \* Nếu dùng phương thức PUT: BE có thể khóa cứng số Bytes chính xác của file bằng cách truyền thuộc tính ContentLength khi tạo URL.  
  * Nếu dùng phương thức POST Policy: BE có thể giới hạn một khoảng dung lượng cho phép (ví dụ: \["content-length-range", 10240, 52428800\] \- từ 10KB đến 50MB). Nếu FE cố tình nhét file nặng hơn, S3 sẽ báo lỗi HTTP 400 EntityTooLarge.

## **PHẦN 4: CÁC BÀI TOÁN THỰC CHIẾN & GIẢI PHÁP PHÒNG THỦ TỐI CAO**

### **1\. Tránh thảm họa trùng tên file và mất dữ liệu**

Nếu 2 người dùng khác nhau cùng tải lên file tên avatar.png vào cùng một Bucket, S3 sẽ tự động ghi đè và làm mất dữ liệu.

* **Giải pháp:** Backend tại Bước 2 bắt buộc phải sinh ra một tên file độc nhất vô nhị bằng cách băm **UUID v4** kết hợp với định dạng file gốc.  
  ![][image2]  
  *Ví dụ:* avatar.png ➡️ d3b07384-d113-4ec6-a192-39832743818f.png  
* Ngoài ra, phân chia thư mục ảo trên S3 theo cấu trúc cây thư mục dựa trên ID người dùng để quản lý khoa học hơn: uploads/users/{userId}/{UUID}.png.

### **2\. Quản lý trạng thái Database (DB State Consistency)**

* **Quy tắc vàng:** Dữ liệu vật lý có thật thì Database mới được phép ghi nhận.  
* **Luồng xử lý chuẩn:** Tại Bước 2, BE tuyệt đối **không được lưu fileUrl vào Database** ngay vì có nguy cơ User hủy upload nửa chừng hoặc mạng sập ở Bước 3 làm xuất hiện "link ma" (Ghost Data). BE chỉ đưa fileUrl cho FE làm biến tạm. Khi FE upload lên S3 thành công nhận về 200 OK, FE mới gọi tiếp API Bước 4 (Báo cáo hoàn thành) kèm theo fileUrl ➡️ Lúc này BE mới chính thức lưu vào Database.

### **3\. Khắc phục lỗ hổng "Magic Bytes" (Fake File Signature)**

AWS S3 chỉ kiểm tra định dạng ở mức bề nổi là Header Content-Type của request chứ không đọc vào nội dung nhị phân thực tế của file để xác minh tính an toàn. Hacker có thể lừa S3 bằng cách đổi đuôi file mã độc thành .jpg và gán Header image/jpeg.

* **Giải pháp vá lỗ hổng:**  
  * **Cách 1 (AWS Lambda Trigger):** Thiết lập S3 Event Notification để gọi hàm AWS Lambda thức dậy ngay khi file vừa được ghi xong. Lambda sẽ nhảy vào đọc Magic Bytes nhị phân của file. Nếu phát hiện giả mạo, thực hiện lệnh **Delete Object** xóa sổ file ngay lập tức.  
  * **Cách 2 (BE post-validation):** BE sau khi nhận được báo cáo thành công ở Bước 4 sẽ chủ động stream đọc ngược vài byte đầu tiên của file từ S3 về để check tính toàn vẹn trước khi ghi nhận vào DB.

### **4\. Phòng thủ bão Spam và lộ Presigned URL**

Nếu một Presigned URL bị lộ giữa đường truyền:

* **S3 ghi đè vật lý bảo vệ kho hàng:** Hacker không thể dùng một link bị lộ để spam hàng triệu file làm tràn ngập bộ nhớ S3 của bạn. Vì link đã khóa cứng Object Key, Hacker gửi 1 triệu lần thì S3 vẫn chỉ ghi đè lên đúng một vị trí file duy nhất đó.  
* **Bảo vệ cổng API tạo URL ở Backend:** Hacker sẽ tập trung spam API tạo link (POST /api/get-presigned-url) để xin hàng triệu link cho các file khác nhau. Do đó, BE bắt buộc phải:  
  1. Yêu cầu **Token Authentication** (chỉ user đăng nhập mới được xin link).  
  2. Áp dụng **Rate Limiting / Throttling** (Giới hạn tối đa 3-5 lần xin link trong 1 phút cho mỗi user).  
  3. Truyền dữ liệu qua giao thức bảo mật mã hóa **HTTPS** để tránh bị bắt gói tin nhặt URL trên đường truyền.  
  4. Đặt thời gian hết hạn của link cực ngắn (Short TTL) khoảng **30 \- 60 giây**.

### **5\. Bản chất CORS vs Postman**

CORS (Cross-Origin Resource Sharing) cấu hình trên S3 **không phải là bức tường lửa bảo mật** để chặn Hacker.

* CORS là bộ quy tắc ứng xử dành riêng cho Trình duyệt. Nếu gọi từ trình duyệt có Origin không hợp lệ, trình duyệt nhận kết quả và chủ động giấu đi để bảo vệ user.  
* Postman hay các công cụ cào dữ liệu (Crawler) không chạy trên trình duyệt nên không có Origin header và phớt lờ luật CORS, do đó chúng vẫn kết nối và tải file từ S3 bình thường nếu Signature hợp lệ. Bảo mật thực sự phải dựa vào **Signature, IP Restriction** và **Short TTL**.

## **PHẦN 5: KIẾN TRÚC QUẢN LÝ OBJECT ID NÂNG CAO**

Trong các hệ thống phân phối cấp cao, việc định danh tên file (Object ID / Object Key) trên S3 không chỉ đơn thuần là dùng chuỗi ngẫu nhiên, mà nó được thiết kế gắn liền với logic nghiệp vụ để tối ưu hóa hiệu năng và chi phí.

### **Kỹ thuật 1: Database Object ID Mapping (Mối liên kết 1:1)**

Thay vì tạo chuỗi ngẫu nhiên, Backend chủ động tạo trước một bản ghi (Record) rỗng hoặc ở trạng thái chờ (PENDING) trong Database để **lấy ra mã ID định danh của chính bản ghi đó** (ObjectId của MongoDB hoặc UUID Primary Key của SQL). BE dùng chính mã ID này làm tên file trên S3.

* **Quy trình hoạt động:**  
  1. FE gửi request xin upload ➡️ BE chọc vào DB tạo bản ghi, lấy về \_id: 64a7b8c9d01e....  
  2. BE lấy luôn cái \_id này đặt tên file trên S3 và ký link: uploads/64a7b8c9d01e...png.  
  3. FE upload lên S3 ➡️ Quay về báo cáo thành công ➡️ BE tìm đúng \_id đó chuyển trạng thái từ PENDING thành SUCCESS.  
* **Ưu điểm:**  
  * **Truy vấn siêu tốc:** Từ tên file trên S3, lập trình viên nhìn vào là biết ngay nó thuộc về bản ghi, user nào trong hệ thống mà không cần chạy lệnh tìm kiếm văn bản.  
  * **Dọn rác tự động tuyệt hảo (Cronjob Cleanup):** Định kỳ, hệ thống quét qua S3, đối chiếu với DB. File nào trên S3 không có ID tương ứng trong DB hoặc có trong DB nhưng trạng thái mãi là PENDING quá 24h ➡️ Xóa thẳng tay để giải phóng bộ nhớ.

### **Kỹ thuật 2: Content-Addressable Storage \- CAS (Định danh bằng Hash nội dung)**

Đây là công nghệ đỉnh cao được các ông lớn như Google Drive, OneDrive áp dụng. Tên file (Object ID) không do DB hay UUID sinh ra, mà được tính toán bằng cách quét thuật toán băm (SHA-256 hoặc MD5) **trên chính dữ liệu nhị phân (ruột) của file**.

* **Quy trình hoạt động:**  
  1. File video 2GB của bạn được chạy thuật toán băm nội dung ra chuỗi duy nhất: e3b0c44298fc1c149af....  
  2. Hệ thống kiểm tra xem trên S3 đã tồn tại file mang tên hashes/e3b0c44298fc1c149af... này chưa.  
  3. **Nếu chưa có:** BE cấp Presigned URL cho FE tải lên bình thường.  
  4. **Nếu đã có (Trùng lặp nội dung):** BE **không cấp link upload nữa**, báo thành công luôn cho FE, và chỉ lưu một dòng liên kết trỏ về file có sẵn đó trong Database. Kỹ thuật này gọi là **Khử trùng lặp dữ liệu (Deduplication)**.  
* **Ưu điểm:**  
  * **Tiết kiệm chi phí khổng lồ:** Nếu 1.000 user cùng upload một bộ phim 2GB trùng nhau, hệ thống UUID tốn 2TB dung lượng Cloud. Hạ tầng CAS chỉ tốn đúng 2GB để lưu 1 file duy nhất, 999 user còn lại dùng chung liên kết trỏ tới.  
  * **Upload tức thì (Instant Upload):** User thứ 2 trở đi bấm upload file trùng sẽ thấy thanh tiến trình nhảy vọt lên 100% trong 0.1 giây vì thực chất file đã nằm trên Mây từ trước, hệ thống không tốn băng thông truyền lại dữ liệu.  
* **Nhược điểm:** Tốn tài nguyên CPU ở Client hoặc Backend để tính toán chuỗi Hash đối với các file dung lượng siêu lớn.

## **PHẦN 6: KIẾN TRÚC XỬ LÝ FILE HỆ THỐNG SIÊU KHỦNG LONG (TRÙM CUỐI)**

Khi hệ thống scale lên mức toàn cầu phục vụ hàng triệu người dùng với dung lượng file cực lớn (nền tảng streaming video, dữ liệu big data), các kỹ sư hệ thống sẽ áp dụng 3 kỹ thuật đỉnh cao sau để đạt hiệu năng tuyệt đối:

### **1\. S3 Native Multipart Upload (Gộp Chunk trực tiếp trên Mây)**

Ở phương pháp Chunk-upload truyền thống, Server Backend phải gánh việc lưu các file tạm rồi chạy lệnh đọc-ghi nội bộ để gộp file. Đối với các file siêu to khổng lồ (từ vài chục GB đến 5TB), phương pháp tối ưu nhất là đẩy toàn bộ việc gộp file cho hạ tầng của AWS xử lý.

* **Cơ chế hoạt động:**  
  1. FE báo sắp up file 50GB. BE gọi sang AWS S3 phát lệnh khởi tạo: InitiateMultipartUpload.  
  2. S3 trả về một UploadId. BE dựa vào ID này để sinh ra **hàng loạt Presigned URL**, mỗi URL đại diện cho một Part (mảnh) của file.  
  3. FE nhận danh sách link, kích hoạt luồng đẩy song song các Part **thẳng lên S3**.  
  4. Sau khi S3 nhận đủ 100% các mảnh, FE phát lệnh chốt sổ CompleteMultipartUpload lên S3 (thông qua BE). **S3 tự dùng hạ tầng đám mây của AWS để gộp các part** thành một file tổng mà không tiêu tốn bất kỳ 1 Byte RAM hay đĩa nào của Backend.  
* **Ứng dụng:** Thích hợp cho các ứng dụng upload video chất lượng cao 4K/8K (như YouTube, TikTok), hoặc các tệp sao lưu dữ liệu hệ thống dung lượng lớn.

### **2\. Giao thức Tus Protocol (Chuẩn hóa Resumable Upload chống rớt mạng)**

Thay vì tự viết code chắp vá để quản lý việc cắt file, lưu vết byte đã truyền ở FE và BE, thế giới đã chuẩn hóa luồng này thành một mã nguồn mở mãnh mẽ tên là **Tus Protocol**.

* **Cơ chế hoạt động:** Tus định nghĩa sẵn một tập hợp các HTTP Header và quy tắc giao tiếp (Stateful) cực kỳ chặt chẽ giữa Client và Server. Bạn chỉ cần cài thư viện Tus-Client ở FE và Tus-Server ở BE (hoặc dùng các Cloud gánh hộ như Cloudflare Stream, Vimeo).  
* Luồng truyền tải vô cùng "lỳ đòn": User đang đi trên tàu upload file 10GB mà bị mất mạng, 3 ngày sau mở app lại, hệ thống tự gửi request HEAD để hỏi máy chủ xem đã nhận đến byte thứ bao nhiêu, rồi tự động **bơm tiếp dữ liệu từ đúng vị trí bị ngắt quãng**.  
* **Ứng dụng:** Tiêu chuẩn bắt buộc cho các dịch vụ cốt lõi chuyên về truyền tải video chuyên nghiệp hoặc lưu trữ đám mây (Cloud Drive).

### **3\. On-the-fly Transformation (Biến đổi định dạng ngay trên đường truyền \- Edge Computing)**

Kỹ thuật này giải quyết bài toán ở đầu ra (ĐỌC / VIEW file). Thông thường, khi user upload một ảnh gốc nặng 10MB, nếu bạn lưu nguyên bản và bắt các client khác tải đúng 10MB đó về làm thumbnail trên điện thoại thì sẽ phá hủy băng thông và trải nghiệm người dùng.

Thay vì bắt Backend chạy Cronjob ngầm để resize thành hàng loạt file ảnh small.jpg, medium.jpg gây tốn tài nguyên và phình to dung lượng lưu trữ trên S3, người ta áp dụng **Edge Computing** tại tầng CDN (như AWS CloudFront \+ Lambda@Edge hoặc Cloudflare Images).

* **Cơ chế hoạt động:**  
  1. Bạn **chỉ lưu đúng 1 file ảnh gốc duy nhất** trên S3 Bucket.  
  2. Khi Frontend muốn lấy ảnh nhỏ về hiển thị, nó chỉ cần gọi một đường link động qua CDN kèm tham số: https://cdn.app.com/avatar.jpg?width=150\&height=150\&format=webp.  
  3. Máy chủ Edge CDN ở gần vị trí địa lý của user nhất sẽ bốc ảnh gốc từ S3 ra, **tiến hành nén, co dãn kích thước và chuyển sang định dạng .webp siêu nhẹ ngay trên đường truyền (Real-time / On-the-fly)** rồi trả về cho user.  
  4. CDN tự động Cache (lưu tạm) kết quả vừa biến đổi đó lại để phục vụ ngay lập tức cho các user tiếp theo.  
* **Ứng dụng:** Tiêu chuẩn bắt buộc cho các nền tảng Thương mại điện tử (E-commerce) ngập tràn hình ảnh sản phẩm (như Shopee, Amazon) hoặc các trang tin tức lớn để tối ưu hóa SEO và tốc độ tải trang.

## **PHẦN 7: LƯU TRỮ VÀ BẢO MẬT CHUẨN DOANH NGHIỆP (BEST PRACTICES)**

### **1\. Nguyên Tắc Lưu Trữ Vào Database (DB)**

* **KHÔNG BAO GIỜ** lưu trực tiếp file nhị phân (dưới dạng BLOB/Buffer) vào Database (MySQL, PostgreSQL, MongoDB). Làm vậy sẽ khiến Database phình to cực nhanh, làm tê liệt hệ thống backup và làm chậm các truy vấn tìm kiếm thông thường.  
* **CHỈ LƯU** đường dẫn tĩnh (URL) hoặc mã định danh duy nhất (Object Key) của file vào Database dưới dạng chuỗi String.  
  *Ví dụ trường lưu trong DB:* https://s3.amazonaws.com/my-bucket/avatars/user-123.webp

### **2\. Các Lớp Phòng Thủ Bảo Mật Cho Backend (Guardrails)**

* **Lớp 1: Chặn từ cổng mạng (Nginx Proxy):** Cấu hình client\_max\_body\_size 50M; để lọc và từ chối thẳng thừng các file quá lớn ngay từ cửa ngõ, không cho phép chạm vào RAM của BE.  
* **Lớp 2: Kiểm tra Magic Bytes (File Signature):** Không bao giờ tin tưởng đuôi file .png hay .jpg mà user truyền lên. BE bắt buộc phải đọc vài byte đầu tiên của file nhị phân để xác định định dạng thực tế của file:  
  * Ảnh JPEG luôn bắt đầu bằng các byte: FF D8 FF  
  * Ảnh PNG luôn bắt đầu bằng các byte: 89 50 4E 47  
  * *Nếu phát hiện sai lệch thực tế, từ chối lưu và báo lỗi bảo mật ngay lập tức.*  
* **Lớp 3: Đổi tên file (Sanitize Filename):** Đổi tên tất cả file nhận được thành chuỗi Hash ngẫu nhiên hoặc UUID (ví dụ: image\_1.png ➡️ d3b07384-d113-4ec6.png) để loại bỏ các ký tự đặc biệt nguy hiểm và tránh việc ghi đè file trùng tên giữa các người dùng.

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,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>
