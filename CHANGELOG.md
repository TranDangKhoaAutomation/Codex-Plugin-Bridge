# Changelog

Phiên bản theo [Semantic Versioning](https://semver.org/lang/vi/).

## 0.9.14 (2026-09-14)

### Phát hành Windows standalone

- Phát hành `PluginBridge-Setup-0.9.14.exe` dạng **single-file EXE**; máy người dùng không cần cài Python hoặc pip để chạy Plugin Bridge.
- Runtime/assets cần thiết của Plugin Bridge được nhúng trong EXE; Local provider không yêu cầu Codex CLI hoặc Claude Code.
- Chuẩn hóa executable cài đặt thành `%LOCALAPPDATA%\PluginBridge\plugin-bridge.exe`.
- Repository phát hành dùng `plugin-bridge.exe`; bỏ binary cũ `codex-plugin-bridge.exe`.

### Sửa lỗi cài đặt / nâng cấp Windows

- Dừng và chờ tiến trình Plugin Bridge đang cài trước khi thay executable.
- Nếu cần, force-stop có giới hạn đúng executable cài đặt thay vì kill rộng.
- Thay file qua temporary file + atomic replace.
- Retry các lỗi khóa file Windows tạm thời, gồm `WinError 5` và `WinError 32`.
- Nếu Windows vẫn giữ file quá thời gian cho phép, trả lỗi cài đặt có kiểm soát thay vì để `PermissionError` chưa xử lý làm crash launcher.
- Hỗ trợ migrate cấu hình từ thư mục legacy `%LOCALAPPDATA%\CodexPluginBridge` khi cấu hình mới chưa tồn tại.

### Verification

- Ruff format/lint: pass.
- mypy: pass.
- Python compile check: pass.
- 410 pytest tests: pass.
- Source/release secret scan: pass.
- Standalone EXE smoke: pass.
- Self-install single-EXE smoke: pass.
- Upgrade smoke từ v0.9.10 lên v0.9.14: pass.
- Port-conflict smoke: pass.
- Managed-package smoke: pass.
- SHA-256 của `PluginBridge-Setup-0.9.14.exe`: `e36bb3d26a331a2cf4e2b19d6968ee1a7ddc9c64c05b665a00ac26f2022e831c`.

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
