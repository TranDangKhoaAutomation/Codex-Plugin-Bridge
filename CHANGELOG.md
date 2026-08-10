# Changelog

Phiên bản theo [Semantic Versioning](https://semver.org/lang/vi/).

## 0.7.0 (2026-08-11)

### Thêm

- **Tự cập nhật (self-update)** cho bản đóng gói Windows EXE:
  - Kiểm tra GitHub Releases khi khởi động (nền, không chặn).
  - Tải EXE mới có kiểm tra SHA-256 từ `SHA256SUMS.txt`, cài thay thế và khởi động lại.
  - Quay lại bản trước (rollback) khi gặp sự cố.
  - Chỉ hoạt động khi chạy bản EXE đã đóng gói; chạy từ source không gọi GitHub.
  - Mất mạng chỉ chuyển trạng thái thành `failed` và ghi log — không crash, không treo.
- Thẻ **Cập nhật ứng dụng** trên dashboard local: xem phiên bản hiện tại, phiên bản mới,
  kiểm tra cập nhật, cập nhật và quay lại bản trước.
- Tùy chọn `--no-update-check` để tắt kiểm tra nền.

### Bảo mật

- Toàn bộ tải về chỉ chấp nhận HTTPS origin thuộc allowlist (GitHub/distribution), verify
  SHA-256 trước khi thay file, swap EXE bằng PowerShell launcher sau khi bridge thoát.

### Khác

- Cập nhật tài liệu và checksum cho bản phát hành 0.7.0.
</content>