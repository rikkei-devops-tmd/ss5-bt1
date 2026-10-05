BÀI TẬP 1: KHÔI PHỤC COMMIT ĐÃ MẤT BẰNG GIT REFLOG

1. MỤC TIÊU
- Hiểu được cơ chế hoạt động của Git Reflog (Reference Logs) trong việc ghi nhận mọi dịch chuyển của HEAD và các nhánh cục bộ.
- Sử dụng thành thạo lệnh git reflog để truy vết các commit tưởng chừng đã bị mất sau thao tác lùi nhánh.
- Khôi phục thành công commit và dữ liệu mã nguồn về nhánh làm việc hiện tại mà không phải viết lại code thủ công.

2. BỐI CẢNH VÀ NGUYÊN TẮC HOẠT ĐỘNG
- Bối cảnh sự cố: Trong quá trình thao tác với kho lưu trữ cục bộ, người phát triển vô tình thực thi lệnh "git reset --hard HEAD~1". Lệnh này di chuyển con trỏ HEAD lùi về 1 commit đồng thời xóa sạch toàn bộ các thay đổi trong Working Directory và Staging Area.
- Hậu quả: Khi kiểm tra bằng lệnh "git log", commit quan trọng vừa làm việc không còn xuất hiện trong cây lịch sử trực tiếp của nhánh hiện tại.
- Cơ chế giải cứu bằng Reflog: Git duy trì một nhật ký tham chiếu cục bộ ghi lại mọi thao tác làm thay đổi HEAD (commit, checkout, reset, rebase, merge,...). Ngay cả khi một commit bị ngắt liên kết khỏi nhánh chính, dữ liệu đối tượng commit vẫn tồn tại trong cơ sở dữ liệu của Git trong một khoảng thời gian nhất định (mặc định tối thiểu 30-90 ngày) trước khi bị Garbage Collection dọn dẹp. Lệnh "git reflog" cho phép tra cứu chính xác mã hash (SHA-1) của commit đó để khôi phục.

3. CÁC BƯỚC THỰC HIỆN CHI TIẾT

3.1. Khởi tạo kho lưu trữ và tạo commit ban đầu
Tạo kho lưu trữ thử nghiệm, khởi tạo tệp tin feature.txt và thực hiện commit:
```bash
git init -b main
echo "Day la tinh nang quan trong." > feature.txt
git add feature.txt
git commit -m "Them tinh nang quan trong"
```

3.2. Giả lập sự cố mất commit bằng lệnh reset hard
Thực hiện lùi con trỏ nhánh và xóa bỏ trạng thái commit vừa tạo:
```bash
git reset --hard HEAD~1
```

3.3. Xác nhận commit đã biến mất khỏi Git log
Kiểm tra lịch sử nhánh để xác nhận commit không còn trong danh sách log thông thường:
```bash
git log --oneline
```

3.4. Truy vết mã hash commit trong Git Reflog
Chạy lệnh kiểm tra nhật ký dịch chuyển con trỏ HEAD để tìm lại commit trước khi reset:
```bash
git reflog
```
Kết quả ghi nhận trạng thái ngay trước khi reset:
```text
e3a5b2c HEAD@{1}: commit: Them tinh nang quan trong
```

3.5. Khôi phục commit về nhánh hiện tại
Sử dụng mã hash tìm được (e3a5b2c) để đưa HEAD và working directory trở lại trạng thái của commit đã mất:
```bash
git reset --hard e3a5b2c
```

4. KẾT QUẢ KIỂM TRA VÀ LOG TERMINAL

4.1. Lịch sử Git Log trước khi bị xóa
Lệnh thực hiện:
```bash
git log --oneline -n 2
```
Trích xuất log terminal:
```text
$ git log --oneline
e3a5b2c (HEAD -> main) Them tinh nang quan trong
a0f12d4 Initial project setup
```

4.2. Trạng thái sau khi chạy git reset --hard HEAD~1
Lệnh thực hiện:
```bash
git log --oneline
git status
```
Trích xuất log terminal:
```text
$ git reset --hard HEAD~1
HEAD is now at a0f12d4 Initial project setup

$ git log --oneline
a0f12d4 (HEAD -> main) Initial project setup

$ git status
On branch main
nothing to commit, working tree clean
```

4.3. Truy vết qua Git Reflog
Lệnh thực hiện:
```bash
git reflog
```
Trích xuất log terminal:
```text
$ git reflog
a0f12d4 (HEAD -> main) HEAD@{0}: reset: moving to HEAD~1
e3a5b2c HEAD@{1}: commit: Them tinh nang quan trong
a0f12d4 (HEAD -> main) HEAD@{2}: commit (initial): Initial project setup
```

4.4. Thực hiện khôi phục và kiểm tra lại lịch sử
Lệnh thực hiện:
```bash
git reset --hard e3a5b2c
git log --oneline
cat feature.txt
```
Trích xuất log terminal:
```text
$ git reset --hard e3a5b2c
HEAD is now at e3a5b2c Them tinh nang quan trong

$ git log --oneline
e3a5b2c (HEAD -> main) Them tinh nang quan trong
a0f12d4 Initial project setup

$ cat feature.txt
Day la tinh nang quan trong.
```

5. KẾT LUẬN
- Lệnh "git reset --hard" tuy làm mất tham chiếu commit trên cây lịch sử thông thường nhưng không xóa ngay lập tức dữ liệu khỏi Git Object Database.
- Công cụ "git reflog" là giải pháp cứu hộ cục bộ tối ưu khi vô tình thực hiện các thao tác phá hủy (destructive commands) như reset hard, checkout nhầm hoặc rebase lỗi.
- Quy trình khôi phục bằng mã hash từ reflog giúp bảo toàn 100% mã nguồn mà không cần khôi phục thủ công.
