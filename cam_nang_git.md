# BẢN CHẤT GIT: CẨM NANG TOÀN DIỆN VỀ PHÂN VÙNG, CON TRỎ VÀ CÁC THAO TÁC SỬA LỖI THỰC CHIẾN

Tài liệu chuyên sâu phân tích cơ chế vận hành cốt lõi, quản lý trạng thái con trỏ HEAD, phân biệt các nhánh và bộ giải pháp xử lý lỗi thực tế từ cơ bản đến nâng cao.

## 1. Bản Chất 3 Phân Vùng Của Git & Con Trỏ HEAD

Để thực sự làm chủ và tự tin gõ bất kỳ lệnh sửa sai nào trong Git, bạn bắt buộc phải hiểu cách Git chia tách dữ liệu trên máy tính của bạn thành 3 phân vùng vật lý:

```text
+-----------------------------------------------------------------------------------+
|                                   MÁY LOCAL CỦA BẠN                               |
|                                                                                   |
|  [1. Working Directory]  =======>  [2. Staging Area]  =======>  [3. Local Repo]    |
|    (Thư mục làm việc)                 (Vùng đệm/Chờ)               (Kho lưu trữ)   |
|                                                                                   |
|  Là những file thực tế     File được đánh dấu chuẩn    Nơi lưu chính thức         |
|  bạn đang mở và chỉnh      bị commit. Hiện màu xanh    các commit lịch sử.        |
|  sửa trực tiếp bằng        lá cây khi gõ:              Được quản lý thông qua     |
|  VS Code, Cursor,...       `git status`                con trỏ HEAD và các nhánh. |
+-----------------------------------------------------------------------------------+
```

### Con trỏ HEAD là gì?

* **HEAD** là một con trỏ định vị vị trí làm việc hiện tại của bạn trong lịch sử Git.
* **Attached HEAD (Trạng thái bình thường):** HEAD bám chặt vào một Nhánh Local di động (ví dụ: HEAD -> main -> Commit mới nhất). Khi bạn tạo commit mới, nhánh di chuyển, HEAD tự động tiến theo để ghi nhận trạng thái mới.
* **Detached HEAD (Trạng thái tách rời):** HEAD bị tháo khỏi nhánh và cắm trực tiếp vào một mã commit cố định (ví dụ khi bạn checkout một tag, một commit cũ, hoặc một nhánh remote origin/abc).

## 2. Giải Mã Ý Nghĩa & Màu Sắc Trong git log

Khi bạn gõ lệnh `git log`, các ký tự đặc biệt và màu sắc xuất hiện trên màn hình terminal cung cấp cho bạn một bản đồ phân bố các nhánh vô cùng trực quan:

* 🟡 **Màu Vàng (Mã Hash):** Mã SHA-1 duy nhất đại diện cho định danh của commit đó trong cơ sở dữ liệu.
* 🔷 **Màu Xanh Lam Sáng (HEAD -> ...):** Chỉ thị **Vị trí hiện tại của bạn**. Thư mục làm việc vật lý trên máy của bạn đang hiển thị chính xác code của commit mang nhãn màu này.
* 🟢 **Màu Xanh Lá Cây (Tên nhánh):** Đại diện cho các **Nhánh Local** hiện có trên máy của bạn (ví dụ: develop, feature/registration-status-list).
* 🔴 **Màu Đỏ (origin/...):** Đại diện cho các **Nhánh Remote** nằm trên Server (GitHub/GitLab) mà máy của bạn ghi nhận được ở lần đồng bộ (fetch) gần nhất.

## 3. Phân Biệt Triết Lý git revert và git reset

Đây là hai tư duy xử lý lịch sử hoàn toàn trái ngược nhau trong Git mà bạn cần nắm chắc để áp dụng đúng môi trường:

```text
Lịch sử ban đầu:
A ---> B ---> C (Lỗi) ---> D (Đang đứng ở đây)

Kịch bản 1: git revert C
Lịch sử mới:
A ---> B ---> C ---> D ---> C' (Commit mới xóa bỏ thay đổi của C)
(Tư duy "Tiến về phía trước" - Phù hợp khi code ĐÃ push lên Server)

Kịch bản 2: git reset --hard B
Lịch sử mới:
A ---> B (Nhánh bị kéo thụt lùi về B, xóa sổ hoàn toàn C và D)
(Tư duy "Quay ngược thời gian" - Chỉ dùng khi code CHƯA push)
```

## 4. Phân Biệt Ký Hiệu Chỉ Định Cha ^ (Caret) và ~ (Tilde)

Khi cần trỏ về các commit trong quá khứ để khôi phục dữ liệu, Git cung cấp hai ký hiệu mạnh mẽ nhưng rất dễ gây nhầm lẫn:

```text
                      A (Nhánh chính nhận merge)
                     / \
                    B   C (Nhánh phụ gộp vào)
                     \ /
                      M (Merge Commit)
```

* **^ (Dấu mũ) - Đi ngang chọn Cha:** Dùng để chọn cha thứ mấy của một Merge Commit (commit có từ 2 cha trở lên).
  * `M^1` hoặc `M^`: Quay về commit cha thứ nhất A (thuộc nhánh chính nhận merge, ví dụ product).
  * `M^2`: Quay về commit cha thứ hai C (thuộc nhánh phụ gộp vào, ví dụ dev).
* **~ (Dấu ngã) - Đi lùi thẳng đứng:** Dùng để đi lùi về quá khứ theo trục dọc thời gian (thế hệ).
  * `M~1`: Quay về 1 commit trước đó (bằng với `M^1`).
  * `M~2`: Quay về 2 commit trước đó dọc theo nhánh chính (A -> cha của A).

## 5. Chuyên Đề: Kỹ Thuật "Phẫu Thuật" Revert Duy Nhất 1 File

**Tình huống thực tế:** Bạn có 3 commit theo thứ tự thời gian: 1 -> 2 -> 3.

Trong đó, **Commit 2** bị lỗi file a.js (commit này sửa nhiều file khác nhau). Bạn chỉ muốn hủy thay đổi của file a.js trong commit 2.

### Kịch bản A: Dùng git restore --source=2^ a.js (Ghi đè file từ quá khứ)

* **Cơ chế:** Git sao chép file a.js sạch từ Commit 1 (2^) đè trực tiếp vào thư mục hiện tại của bạn.
* **Vị trí HEAD:** Vẫn đứng im ở **Commit 3**.
* **Ưu điểm:** Cực nhanh, gọn.
* **Hạn chế lớn:** Nếu **Commit 3** cũng có sửa file a.js (các dòng code mới, chuẩn), hành động đè này sẽ **xóa sạch** luôn cả những thay đổi tốt ở Commit 3. Bạn sẽ bị mất code oan uổng!

### Kịch bản B: Dùng git revert -n 2 kết hợp lọc file (Phẫu thuật tinh xảo - KHUYÊN DÙNG)

* **Cơ chế:** Sử dụng thuật toán 3 bên của Git để chỉ bóc tách phần thay đổi lỗi của commit 2 ra khỏi file a.js, giữ lại các thay đổi chuẩn của commit 3.
* **Vị trí HEAD:** Vẫn đứng im ở **Commit 3** (nhờ flag -n kìm phanh không tạo commit mới).

**Quy trình 3 bước thực hiện:**

```bash
# Bước 1: Tính toán code đảo ngược của commit 2 nhưng KHÔNG commit ngay (-n)
git revert -n 2

# Bước 2: Khôi phục tất cả các file khác (ví dụ b.js) về trạng thái HEAD hiện tại (không revert tụi này)
git restore --staged --worktree b.js

# Bước 3: Bây giờ chỉ còn duy nhất file a.js bị revert trong Staging Area, tiến hành commit
git commit -m "Chỉ revert file a.js từ commit 2"
```

## 6. PHÂN TÍCH CHUYÊN SÂU CÁC LỆNH TƯƠNG TÁC HỆ THỐNG

Dưới đây là phân tích chi tiết cơ chế vận hành của 11 lệnh thao tác thực tế, được phân nhóm theo bản chất phân vùng tác động.

### PHÂN KHÚC I: THAO TÁC TRÊN VÙNG ĐỆM (STAGING AREA)

*Nhóm lệnh này giải quyết vấn đề khi bạn lỡ tay chạy `git add` nhầm file, hoặc muốn đưa file ra khỏi trạng thái "chuẩn bị commit" (màu xanh lá) về lại trạng thái "chỉ chỉnh sửa" (màu đỏ).*

#### Lệnh 1 & 3: git reset file_name / git restore --staged file_name

* **Bản chất cơ chế:** Git sẽ tìm đến snapshot của commit HEAD hiện tại, lấy trạng thái của file_name tại đó và **ghi đè** lên vùng Staging Area.
* **Tác động phân vùng:**
  * *Working Directory:* **Giữ nguyên 100%**. Code bạn đang viết dở trong file hoàn toàn không bị mất hay thay đổi.
  * *Staging Area:* Thay đổi trạng thái file từ "To be committed" (Xanh lá) thành "Not staged for commit" (Màu đỏ).
  * *Local Repo (HEAD):* Giữ nguyên.
* **Mức độ rủi ro:** **0% (Tuyệt đối an toàn)**. Không thể mất code khi dùng lệnh này.
* **Thực tế áp dụng:** Bạn sửa 5 file, gõ nhầm `git add .` để đưa cả 5 file vào Staging. Sau đó bạn nhận ra file config.env chứa mật khẩu cá nhân không nên commit. Hãy gõ `git restore --staged config.env` để rút nó ra ngoài an toàn.

#### Lệnh 2: git reset (Unstage toàn bộ)

* **Bản chất cơ chế:** Giống hệt lệnh trên nhưng áp dụng trên phạm vi toàn bộ thư mục gốc. Nó đồng bộ hóa (sync) toàn bộ trạng thái của Staging Area về khớp hoàn toàn với HEAD.
* **Tác động phân vùng:** Làm trống hoàn toàn Staging Area.
* **Mức độ rủi ro:** **0% (An toàn)**.

### PHÂN KHÚC II: HỦY THAY ĐỔI TRÊN THƯ MỤC LÀM VIỆC (WORKING DIRECTORY)

*Nhóm lệnh này can thiệp trực tiếp vào code bạn đang gõ trên IDE. Hãy cực kỳ cẩn thận.*

#### Lệnh 4 & 5: git restore file_name / git restore . (Xóa code chưa commit)

* **Bản chất cơ chế:** Git lấy phiên bản file sạch từ Staging Area (nếu đã từng add) hoặc từ HEAD (nếu chưa từng add) và **ghi đè thô bạo** lên file vật lý trong thư mục làm việc của bạn.
* **Tác động phân vùng:**
  * *Working Directory:* **Bị ghi đè hoàn toàn**. Mọi thay đổi bạn vừa gõ thêm bằng tay kể từ commit/add gần nhất sẽ bị xóa bỏ.
  * *Staging Area & Local Repo:* Giữ nguyên.
* **Mức độ rủi ro:** 🔥 **CỰC KỲ NGUY HIỂM (Mất code vĩnh viễn)**. Vì những đoạn code bạn vừa gõ chưa bao giờ được lưu vào cơ sở dữ liệu của Git (chưa bao giờ commit), một khi đã ghi đè bằng restore, **không có cách nào cứu lại được**.
* **Thực tế áp dụng:** Bạn đang thử nghiệm một thuật toán mới trong file a.js, sau 1 tiếng code bạn thấy nó quá rối và không chạy được. Bạn muốn xóa hết toàn bộ đống rác này đi để viết lại từ đầu trạng thái sạch của commit trước: Gõ `git restore a.js`.

### PHÂN KHÚC III: RESET LỊCH SỬ LOCAL (SỨC MẠNH CỦA SOFT - MIXED - HARD)

*Nhóm lệnh dùng để xử lý khi commit đã được tạo thành công ở máy local (Local Repo) nhưng bạn muốn "quay xe".*

```text
Trạng thái ban đầu:
Commit 1 ---> Commit 2 (HEAD đang đứng ở đây)
```

#### Lệnh 6: git reset --soft HEAD~1 (Hủy commit, giữ Staging)

* **Bản chất cơ chế:** Git chỉ dịch chuyển duy nhất con trỏ HEAD và nhánh hiện tại lùi lại 1 bước về Commit 1. Nó không đụng vào Staging Area và Working Directory.
* **Tác động phân vùng:**
  * *Working Directory:* Giữ nguyên code bạn đã viết.
  * *Staging Area:* Giữ nguyên. Toàn bộ các file của Commit 2 bị hủy hiện tại vẫn nằm sẵn ở vùng chờ màu xanh lá.
  * *Local Repo:* HEAD lùi về Commit 1.
* **Mức độ rủi ro:** **0% (An toàn tuyệt đối)**.
* **Thực tế áp dụng:** Bạn vừa gõ `git commit -m "feat: add api"`. Ngay sau khi nhấn Enter, bạn nhận ra mình viết sai chính tả trong commit message hoặc quên chưa thêm file README.md. Hãy gõ `git reset --soft HEAD~1`, sửa lại lỗi, rồi commit lại.

#### Lệnh 7: git reset HEAD~1 (Mặc định - mixed)

* **Bản chất cơ chế:** Dịch chuyển con trỏ HEAD lùi về Commit 1 **VÀ** đồng bộ hóa Staging Area về khớp với Commit 1.
* **Tác động phân vùng:**
  * *Working Directory:* Giữ nguyên code của bạn (file hiển thị màu đỏ - unstaged).
  * *Staging Area:* Bị xóa bỏ trạng thái chờ commit.
  * *Local Repo:* HEAD lùi về Commit 1.
* **Mức độ rủi ro:** **0% (An toàn)**. Code của bạn vẫn nguyên vẹn trên màn hình.
* **Thực tế áp dụng:** Bạn đã gom quá nhiều tính năng lộn xộn vào chung một commit Commit 2. Bạn muốn rã nó ra để chia thành 3 commit nhỏ hơn, gọn gàng hơn. Hãy dùng lệnh này để đưa toàn bộ code về trạng thái thô (unstaged), sau đó `git add` chọn lọc từng file để commit riêng lẻ.

#### Lệnh 8: git reset --hard HEAD~1 (Xóa sổ lịch sử và code)

* **Bản chất cơ chế:** Dịch chuyển HEAD lùi về Commit 1, đồng thời **ép phẳng cả Staging Area và Working Directory** về trạng thái của Commit 1.
* **Tác động phân vùng:** Cả 3 phân vùng đồng loạt bị kéo lùi về quá khứ.
* **Mức độ rủi ro:** 🔥 **CỰC KỲ NGUY HIỂM (Mất sạch code mới viết)**.
* **🚨 TUYỆT CHIÊU CỨU MẠNG (Git Reflog):** Nếu bạn lỡ tay gõ `git reset --hard HEAD~1` và đứng tim vì nhận ra mất sạch code của Commit 2. Đừng hoảng sợ! Hãy gõ ngay:

```bash
git reflog
```

Git sẽ hiển thị nhật ký di chuyển của con trỏ HEAD (kể cả những commit đã bị "xóa"). Tìm dòng có nội dung kiểu `d9cce18 HEAD@{1}: commit: [feat] add api`. Copy mã hash `d9cce18` và gõ:

```bash
git reset --hard d9cce18
```

Phép màu xảy ra! Lịch sử và code của bạn sẽ quay trở lại nguyên vẹn như chưa từng có cuộc chia ly.

### PHÂN KHÚC IV: ĐỒNG BỘ CƯỠNG CHẾ & DỌN DẸP RÁC

#### Lệnh 9: git fetch origin && git reset --hard origin/branch_name (Đồng bộ sạch sẽ)

* **Bản chất cơ chế:**
  1. `git fetch origin` tải toàn bộ lịch sử chuẩn nhất từ Server về máy (lưu trong kho ẩn).
  2. `git reset --hard` bẻ gãy hoàn toàn nhánh local hiện tại, bắt nó phải trỏ vào vị trí chính xác của nhánh trên server (origin/branch_name), đồng thời dọn sạch code local.
* **Mức độ rủi ro:** 🔥 **Nguy hiểm cho code chưa commit**. Sẽ xóa sạch mọi thay đổi chưa lưu ở local để đồng bộ với server.
* **Thực tế áp dụng:** Bạn thực hiện rebase lỗi, hoặc merge linh tinh khiến nhánh local rối tung lên và không thể push được nữa. Thay vì ngồi gỡ từng lỗi conflict, bạn quyết định "đập đi xây lại", lấy code chuẩn trên GitHub làm mốc. Đây chính là lệnh bạn cần.

#### Lệnh 10: git clean -fd (Dọn dẹp triệt để)

* **Bản chất cơ chế:** Đi xuyên qua thư mục dự án và xóa thẳng tay mọi file (-f) và thư mục (-d) không nằm trong danh sách quản lý của Git.
* **Mức độ rủi ro:** 🔥 **Nguy hiểm**.
* **💡 Pro-tip:** Trước khi chạy, luôn luôn chạy thử ở chế độ "Dry Run" (chạy thử nghiệm) để xem Git sẽ xóa những gì bằng lệnh:

```bash
git clean -nd
```

Khi màn hình liệt kê đúng các file rác (như file log, file build .class, dist/), bạn mới chạy `git clean -fd` để dọn dẹp thật.

### PHÂN KHÚC V: HOÀN TÁC CỘNG TÁC (PUBLIC BRANCH)

#### Lệnh 11: git revert commit_id (Đảo ngược văn minh)

* **Bản chất cơ chế:** Git tính toán toán học để lấy nghịch đảo của các thay đổi trong commit_id, sau đó áp dụng nó vào code hiện tại và **tự động tạo một commit mới** ở đầu dòng lịch sử.
* **Tác động phân vùng:** Tiến lên phía trước, tạo commit mới ở Local Repo, cập nhật Working Directory và Staging.
* **Mức độ rủi ro:** **0% (An toàn nhất khi làm việc nhóm)**.
* **Thực tế áp dụng:** Code lỗi đã push lên nhánh main chung của cả công ty. Bạn không thể reset vì sẽ làm lệch lịch sử của 50 lập trình viên khác. Hãy dùng git revert để hủy code lỗi một cách êm thấm, lịch sử Git vẫn tiến lên một bước dài sạch sẽ.
