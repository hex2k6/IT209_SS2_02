# Bài 2: Khởi tạo user thường và cấu hình Sudoers

> **Trạng thái: Bản hướng dẫn và báo cáo đang thực hiện, chưa có log kiểm tra bài 2.**
> Các lệnh dưới đây là quy trình dự kiến; chưa được ghi nhận là đã chạy thành công. Chỉ cập nhật kết quả sau khi thực hành.

## 1. Thông tin

- Họ tên: **Bui Van Phuong**
- GitHub: [hex2k6](https://github.com/hex2k6)
- Môi trường đã xác minh ở bài 1: VPS AZVPS, Ubuntu 22.04.5 LTS.
- Tài khoản quản trị hiện dùng: `phuong`.
- Tài khoản cần tạo: `devops`.
- AZVPS được sử dụng thay DigitalOcean theo xác nhận của học viên về sự cho phép của giảng viên.

## 2. Mục tiêu

Tạo tài khoản làm việc thường `devops`, cấp quyền quản trị qua nhóm `sudo`, cài khóa công khai được root cho phép vào tài khoản mới và kiểm tra đăng nhập SSH bằng khóa từ Windows. Chỉ dùng `sudo` khi cần thao tác quản trị.

## 3. Kiểm tra điều kiện ban đầu trên VPS

Giữ phiên `phuong` hiện tại mở trong suốt quá trình. Chạy:

```sh
whoami
id devops
sudo -v
sudo test -s /root/.ssh/authorized_keys && echo "ROOT_KEY_PRESENT" || echo "ROOT_KEY_MISSING_OR_CHECK_FAILED"
```

- Nếu `id devops` báo không có user, có thể tạo user mới.
- Nếu user đã tồn tại, kiểm tra tài khoản trước khi thay đổi; không tạo lại hoặc ghi đè cấu hình SSH.
- Chỉ tiếp tục sao chép khi lệnh kiểm tra root key thành công.
- Bài 1 chưa xác minh tạo SSH key thành công. Nếu thiếu key hoặc không có private key tương ứng trên Windows, cần hoàn tất thiết lập cặp khóa và public key của root trước. Một file authorized_keys có sẵn không chứng minh học viên giữ private key tương ứng.
- Không gửi mật khẩu, private key hoặc token vào báo cáo/chat.

## 4. Tạo user và cấp quyền sudo

Chạy trên VPS từ tài khoản `phuong` có quyền sudo:

```sh
sudo adduser devops
sudo usermod -aG sudo devops
id devops
```

Khi `adduser` hỏi mật khẩu, đặt mật khẩu mới riêng cho `devops` và nhập lại. Không đưa mật khẩu vào lệnh hoặc báo cáo. Các trường thông tin tùy chọn có thể để trống.

Tùy chọn `-aG` thêm nhóm sudo mà giữ các nhóm bổ sung hiện có. Sử dụng nhóm sudo có sẵn của Ubuntu; không cần cấu hình NOPASSWD hay sửa trực tiếp /etc/sudoers.

## 5. Sao chép authorized_keys từ root

**Chỉ áp dụng sau khi xác nhận root có public key đúng và devops là tài khoản mới, chưa có authorized_keys cần giữ lại.** Nếu tài khoản đã có cấu hình SSH, dừng để kiểm tra trước khi sao chép.

```sh
sudo install -d -m 700 -o devops -g devops /home/devops/.ssh
sudo install -m 600 -o devops -g devops /root/.ssh/authorized_keys /home/devops/.ssh/authorized_keys
sudo stat -c '%a %U:%G %n' /home/devops/.ssh /home/devops/.ssh/authorized_keys
```

Chỉ sao chép danh sách public key được phép đăng nhập, không sao chép private key của root. Dùng đường dẫn tuyệt đối `/root/.ssh/authorized_keys` vì phiên hiện tại là `phuong`; `~/.ssh` trong phiên này trỏ tới thư mục của phuong.

**Kết quả mong đợi, chưa phải log thực tế:**

```text
700 devops:devops /home/devops/.ssh
600 devops:devops /home/devops/.ssh/authorized_keys
```

## 6. Kiểm tra từ Windows

Mở PowerShell mới trên Windows; giữ phiên quản trị cũ mở. Chỉ sử dụng đường dẫn dưới đây nếu khóa đã được tạo đúng trên Windows và public key tương ứng đã được cài ở bước trước:

```powershell
ssh -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no -i "$env:USERPROFILE\.ssh\azvps_ed25519" devops@<IP_VPS>
```

Thay `<IP_VPS>` bằng IP thật. Các tùy chọn trên ngăn chuyển sang xác thực mật khẩu máy chủ để kiểm tra đúng SSH key. Passphrase bảo vệ private key, nếu có, vẫn có thể được yêu cầu.

Sau khi đăng nhập bằng devops, chạy:

```sh
whoami
id
sudo -k
sudo whoami
```

Kết quả cần đạt:

- `whoami` trả về `devops`.
- `id` có nhóm `sudo`.
- `sudo whoami` yêu cầu mật khẩu của **devops** theo cấu hình mặc định và trả về `root`.
- Dấu nhắc yêu cầu mật khẩu sudo là bình thường; không đồng nghĩa đăng nhập SSH bằng mật khẩu.

## 7. Bằng chứng thực hành

Chưa có bằng chứng bài 2. Bổ sung ảnh hoặc log thật cho:

1. Tạo tài khoản và kết quả `id devops`.
2. Quyền và chủ sở hữu .ssh / authorized_keys.
3. Lệnh đăng nhập bằng khóa từ Windows và phiên devops thành công.
4. `whoami` và `sudo whoami` trong phiên devops.

Không thay log còn thiếu bằng kết quả mong đợi.

## 8. Checklist nộp bài

- [ ] Đã tạo devops.
- [ ] Đã thêm devops vào nhóm sudo.
- [ ] Đã xác minh public key của root tương ứng với private key trên Windows.
- [ ] Đã sao chép authorized_keys từ root.
- [ ] .ssh là 700, authorized_keys là 600, chủ sở hữu devops:devops.
- [ ] Đăng nhập SSH bằng khóa với tài khoản devops thành công.
- [ ] sudo whoami trả về root.
- [ ] Đã bổ sung bằng chứng thực tế và cập nhật trạng thái báo cáo.

Đường dẫn bài: [homework/session_02/ex2/](https://github.com/hex2k6/IT209_SS2_02/tree/main/homework/session_02/ex2).
