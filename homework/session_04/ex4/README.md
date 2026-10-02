# Báo Cáo Thực Hành: Quản Lý Tệp Tin Bỏ Qua (.gitignore) và Sửa Lịch Sử (Amend)

- **Học viên:** An Hải Dũng
- **Email:** anhaidung2k6@gmail.com
- **Tài khoản GitHub:** AnHaiDung
- **Bài tập:** Session 04 - Bài 4 (`homework/session_04/ex4/`)

---

## 1. Mục tiêu bài thực hành
- **Cấu hình tệp tin ẩn `.gitignore`:** Tự động bỏ qua các tệp tin chứa thông tin nhạy cảm (secrets, API keys, credentials) và các tệp tin rác của hệ thống hoặc môi trường phát triển.
- **Gỡ bỏ tệp tin khỏi cache theo dõi của Git an toàn (`git rm --cached`):** Hiểu rõ cơ chế tách biệt giữa bộ nhớ đệm (Staging Area/Index) và thư mục làm việc cục bộ (Working Directory).
- **Chỉnh sửa lịch sử commit gần nhất (`git commit --amend`):** Loại bỏ tệp tin nhạy cảm đã commit nhầm ra khỏi lịch sử commit gần nhất và viết lại thông điệp commit chuẩn mực, bảo đảm an toàn thông tin trước khi đẩy lên remote repository.

---

## 2. Bối cảnh & Ràng buộc kỹ thuật

### Bối cảnh
Trong quá trình khởi tạo và phát triển dự án, học viên vô tình đưa tệp tin `credentials.txt` chứa mật khẩu cơ sở dữ liệu và API key vào Staging Area và đã thực hiện lệnh commit lên Git. Cần phải:
1. Gỡ bỏ tệp tin `credentials.txt` này khỏi sự theo dõi của Git.
2. Giữ nguyên vẹn tệp tin `credentials.txt` vật lý trên ổ đĩa để tiếp tục sử dụng cho việc phát triển cục bộ.
3. Thiết lập `.gitignore` để Git không bao giờ theo dõi lại tệp tin này trong tương lai.
4. Sửa lại commit gần nhất bằng `--amend` để loại bỏ hoàn toàn tệp nhạy cảm khỏi snapshot của commit đó và cập nhật lại thông điệp commit sạch sẽ.

### Ràng buộc
- **Tuyệt đối không dùng lệnh xóa vật lý** trên hệ điều hành (`rm`, `del`, `Remove-Item`...).
- Tệp tin `credentials.txt` phải được giữ lại nguyên vẹn tại thư mục làm việc cục bộ (`homework/session_04/ex4/credentials.txt`).

---

## 3. Cơ sở lý thuyết

### 3.1. Phân biệt `git rm` và `git rm --cached`
Trong mô hình kiến trúc 3 vùng của Git:
- **Working Tree (Thư mục làm việc):** Nơi chứa các tệp tin vật lý trên đĩa cứng.
- **Staging Area / Index (Bộ nhớ đệm):** Nơi chuẩn bị các thay đổi cho commit tiếp theo.
- **Git Repository (Lịch sử commit):** Nơi lưu trữ vĩnh viễn các snapshot đã commit.

Khi cần xóa file:
- `git rm <file>`: Xóa file khỏi cả **Staging Area** lẫn **Working Tree** (file vật lý trên đĩa cứng bị xóa mất).
- `git rm --cached <file>`: Chỉ xóa file khỏi **Staging Area (Index)**, chuyển trạng thái theo dõi của file từ *Tracked* thành *Untracked*, đồng thời **giữ nguyên 100% tệp tin vật lý trên ổ đĩa**. Đây là giải pháp an toàn tuyệt đối khi cần sửa sai việc commit nhầm tệp tin cấu hình nhạy cảm.

### 3.2. Vai trò của `.gitignore`
- Khi một file ở trạng thái *Untracked*, lệnh `git status` sẽ luôn cảnh báo file đó chưa được theo dõi. Nếu lập trình viên gõ `git add .`, file đó lại bị đưa vào Staging Area.
- Tệp `.gitignore` chỉ thị cho Git bỏ qua hoàn toàn các tệp tin hoặc thư mục khớp với mẫu (pattern) đã khai báo.
- **Lưu ý quan trọng:** `.gitignore` chỉ có tác dụng đối với các tệp tin chưa được theo dõi (*Untracked*). Đối với các tệp tin đã được theo dõi (*Tracked*), bắt buộc phải dùng `git rm --cached` trước thì `.gitignore` mới bắt đầu phát huy tác dụng.

### 3.3. Cơ chế hoạt động của `git commit --amend`
- Khi sử dụng tùy chọn `--amend`, Git sẽ:
  1. Lấy trạng thái hiện tại trong Staging Area (bao gồm cả các file được thêm mới, sửa đổi hoặc xóa bằng `git rm --cached`).
  2. Tạo ra một commit object mới thay thế hoàn toàn cho commit đỉnh hiện tại (`HEAD`).
  3. Cập nhật lại thông điệp commit (commit message) mới nếu được chỉ định.
- Nhờ `--amend`, tệp tin nhạy cảm bị loại bỏ trực tiếp khỏi snapshot của commit gần nhất, giúp lịch sử Git sạch sẽ như thể tệp đó chưa từng được commit.

---

## 4. Quá trình thực hiện chi tiết

### Bước 1: Mô phỏng tình huống commit nhầm tệp `credentials.txt`
Khởi tạo dự án và tạo tệp chứa thông tin nhạy cảm `credentials.txt`:
```bash
git init -b main
# Tạo file mã nguồn app.py và file nhạy cảm credentials.txt
git add .
git commit -m "Initial commit with project files and credentials.txt"
```

Trạng thái commit ban đầu:
```text
[main (root-commit) e512aba] Initial commit with project files and credentials.txt
 2 files changed, 17 insertions(+)
 create mode 100644 homework/session_04/ex4/app.py
 create mode 100644 homework/session_04/ex4/credentials.txt
```
*(File `credentials.txt` đã vô tình bị lưu vào cơ sở dữ liệu commit của Git).*

---

### Bước 2: Gỡ bỏ tệp tin khỏi cache theo dõi của Git một cách an toàn
Sử dụng cờ `--cached` để xóa file khỏi chỉ mục của Git mà không xóa file trên đĩa cứng:
```bash
git rm --cached homework/session_04/ex4/credentials.txt
```

**Kết quả thực thi:**
```text
rm 'homework/session_04/ex4/credentials.txt'
```

Kiểm tra sự tồn tại vật lý của tệp tin trên ổ cứng:
```powershell
Test-Path homework/session_04/ex4/credentials.txt
# Output: True (File vật lý vẫn tồn tại nguyên vẹn!)
```

Kiểm tra trạng thái `git status` tại thời điểm này:
```text
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	deleted:    homework/session_04/ex4/credentials.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	homework/session_04/ex4/credentials.txt
```
> Lúc này Git đã đánh dấu xóa `credentials.txt` trong Staging Area và đưa file vật lý vào danh sách `Untracked files`.

---

### Bước 3: Cấu hình tệp tin `.gitignore`
Tạo tệp `.gitignore` trong thư mục `homework/session_04/ex4/.gitignore` (và thư mục gốc repo) để Git tự động bỏ qua `credentials.txt` vĩnh viễn:
```gitignore
# Ignore sensitive credentials
credentials.txt
*.env
.env
*.pem
*.key
secrets/
```

Sau khi tạo `.gitignore`, kiểm tra lại `git status`:
```text
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	deleted:    homework/session_04/ex4/credentials.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.gitignore
	homework/session_04/ex4/.gitignore
```
> `credentials.txt` đã biến mất hoàn toàn khỏi danh sách `Untracked files` nhờ quy tắc trong `.gitignore`.

---

### Bước 4: Sửa đổi commit gần nhất bằng `git commit --amend`
Đưa tệp `.gitignore` và báo cáo `README.md` vào Staging Area:
```bash
git add .
```

Thực hiện lệnh commit với tùy chọn `--amend` để gộp các thay đổi (gỡ bỏ file nhạy cảm, thêm `.gitignore`) và viết lại thông điệp commit:
```bash
git commit --amend -m "feat(ex4): configure .gitignore and remove sensitive credentials from git tracking"
```

---

## 5. Kết quả kiểm tra & Xác nhận

### 5.1. Kiểm tra trạng thái làm việc (`git status`)
Lệnh kiểm tra:
```bash
git status
```

**Kết quả hiển thị:**
```text
On branch main
nothing to commit, working tree clean
```
- Tệp `credentials.txt` không còn xuất hiện trong danh sách theo dõi của Git (không ở dạng Staged hay Modified, hay Untracked).
- Tệp tin vật lý `credentials.txt` vẫn tồn tại an toàn trên máy cục bộ.

### 5.2. Kiểm tra lịch sử commit (`git log -n 1`)
Lệnh kiểm tra:
```bash
git log -n 1
```

**Kết quả hiển thị:**
```text
commit 74c830ca5f588dfdf08ffe52de4596abd79e6c86 (HEAD -> main)
Author: AnHaiDung <anhaidung2k6@gmail.com>
Date:   Fri Oct 2 15:58:27 2026 +0700

    feat(ex4): configure .gitignore and remove sensitive credentials from git tracking
```

Kiểm tra danh sách các file trong commit mới:
```bash
git show --stat
```
Trong commit mới chỉ bao gồm:
- `homework/session_04/ex4/app.py`
- `homework/session_04/ex4/.gitignore`
- `homework/session_04/ex4/README.md`
- `.gitignore`
*(Tuyệt đối không còn sự hiện diện của `credentials.txt` trong commit này).*

---

## 6. Tổng kết & Best Practices về bảo mật trong Git
1. **Luôn cấu hình `.gitignore` ngay khi khởi tạo dự án:** Định nghĩa sẵn các file nhạy cảm (`.env`, `credentials.txt`, `*.key`) trước khi thực hiện commit đầu tiên.
2. **Sử dụng `git rm --cached` đúng lúc:** Khi lỡ đưa file không mong muốn vào Git, `--cached` là công cụ an toàn nhất để loại bỏ khỏi tracking mà không làm mất dữ liệu cục bộ.
3. **Chỉ dùng `git commit --amend` cho commit cục bộ (Local):** `--amend` thay đổi mã hash của commit (viết lại lịch sử). Chỉ nên sử dụng khi commit chưa được đẩy (`push`) lên remote repository chia sẻ để tránh xung đột với các thành viên khác trong nhóm.
