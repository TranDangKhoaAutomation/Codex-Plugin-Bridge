# Codex Plugin Bridge

> Cầu nối MCP an toàn trên Windows giữa **ChatGPT**, **Codex CLI** / **Claude Code** và các workspace cục bộ.

**Version:** `0.7.0` · **Nền tảng:** Windows · **Giấy phép:** MIT

**Trang tải bản phát hành:** <https://github.com/TranDangKhoaAutomation/Codex-Plugin-Bridge/releases>

Codex Plugin Bridge biến máy Windows thành execution gateway cục bộ cho ChatGPT thông qua giao thức **MCP (Model Context Protocol)**. Dự án tách việc thực thi thành hai lớp độc lập:

- **Local Coder tools** — thao tác nhỏ, deterministic: liệt kê thư mục, đọc/tìm kiếm văn bản, chỉnh sửa chính xác và Git status/diff.
- **Codex / Claude task runner** — công việc lớn: đọc nhiều file, refactor đa file, build, test, migration, debugging hoặc tiếp tục một session đang dở.

Bridge chỉ bind HTTP server lên **loopback** (`127.0.0.1`), có dashboard quản trị local-only, hỗ trợ **Cloudflare Tunnel** và **zrok Reserved Name** để đưa MCP endpoint ra HTTPS, đồng thời áp dụng nhiều lớp kiểm soát quyền trước khi truy cập filesystem hoặc CLI.

---

## Kiến trúc

```text
ChatGPT / MCP client
        │ HTTPS + MCP Streamable HTTP
        ▼
Cloudflare Tunnel / zrok / Local only
        │
        ▼
Codex Plugin Bridge — 127.0.0.1:8765
        ├── MCP JSON-RPC  /mcp/
        ├── Dashboard local  /setup
        ├── Local Coder Tools
        └── Task Runner (bất đồng bộ)
                  │
                  ▼
          Codex CLI hoặc Claude Code
                  │
                  ▼
          Project trên Windows
```

MCP protocol version hiện dùng: `2025-06-18`.

## Tính năng chính

- **MCP Streamable HTTP JSON-RPC** — ChatGPT kết nối như một MCP server HTTP.
- **Standalone Windows executable** — đóng gói bằng PyInstaller, không cần cài Python.
- **Tự cập nhật trong bản EXE** — kiểm tra GitHub Releases khi khởi động, tải EXE có kiểm tra SHA-256, cài và khởi động lại; chạy từ source không gọi mạng.
- **Dashboard quản trị** tại `http://127.0.0.1:8765/setup` (chỉ truy cập từ máy local).
- **Local Coder tools** có giới hạn: đọc, tìm kiếm, ghi, chỉnh sửa chính xác và Git status/diff.
- **Task runner bất đồng bộ** — có task ID, events, session ID và hủy (cancel) process tree.
- **Hỗ trợ Codex CLI và Claude Code (Claude CLI)** — chọn CLI provider ngay trên dashboard.
- **Access profile:** `safe`, `extended`, `trusted-full-access`.
- **Phân quyền riêng:** Local Coder permission và task permission tách biệt.
- **Xác thực:** Bearer token phát triển, OAuth development server, và chế độ external OAuth resource-server.
- **Tunnel providers:** Cloudflare Quick Tunnel, Cloudflare Managed Tunnel, zrok2 Reserved Name và Local only.
- **cloudflared được pin version** và kiểm tra SHA-256 trước khi chạy; zrok2 cũng được pin và verify hash.
- **Quản lý log JSON** có redaction bí mật và rotation.
- **Chặn an toàn đường dẫn:** traversal, UNC/absolute path, symlink/junction/reparse point và các đường dẫn chứa bí mật (`.env`, `.ssh`, credential, secret, key...).

## Yêu cầu hệ thống

### Dùng bản phát hành (EXE)

- **Windows** (10/11).
- **Internet** nếu dùng Cloudflare Tunnel hoặc cần tải `cloudflared` / `zrok` / `ngrok`.
- **Codex CLI** chỉ bắt buộc khi dùng task tools của Codex. **Claude Code** chỉ bắt buộc khi chọn Claude làm CLI provider.
- Codex CLI / Claude Code phải **đăng nhập** để chạy task thực tế.

Bản phát hành **không yêu cầu người dùng cuối cài Python**.

### Phát triển từ source

- Python `>=3.9`.
- PowerShell trên Windows.
- Git (khuyến nghị).
- Codex CLI / Claude Code nếu cần test task runner thực tế.

Runtime dependencies trong `pyproject.toml`:

```text
cryptography==43.0.3
PyJWT==2.9.0
```

Dev dependencies: `mypy==1.11.2`, `pyinstaller==6.10.0`, `pytest==8.3.3`, `ruff==0.6.9`.

## Tải và chạy EXE

### Tải

Tải bản phát hành mới nhất từ **Releases** của repository này:

- `codex-plugin-bridge.exe` — file EXE độc lập (chính).
- `cloudflared.exe` — binary tunnel đã pin (cần đặt cạnh EXE để dùng Cloudflare Tunnel).
- `cloudflared-release.json` / `zrok-release.json` — metadata pin version/hash.
- `install.ps1` / `uninstall.ps1` — script cài đặt / gỡ cài đặt.
- `README.md` / `LICENSE` / `THIRD_PARTY_NOTICES.md` — tài liệu.
- `SHA256SUMS.txt` — kiểm tra toàn vẹn file.

> Kiểm tra hash trước khi chạy (ví dụ):
> ```powershell
> Get-FileHash codex-plugin-bridge.exe -Algorithm SHA256
> ```

### Chạy trực tiếp

```powershell
.\codex-plugin-bridge.exe
```

Lần chạy đầu tiên EXE sẽ **tự cài đặt theo user** vào:

```text
%LOCALAPPDATA%\CodexPluginBridge
```

rồi khởi chạy lại và tự mở dashboard `http://127.0.0.1:8765/setup`.

Muốn chạy portable (không self-install):

```powershell
.\codex-plugin-bridge.exe --portable --no-browser
```

### Tự cập nhật

Chỉ bản đóng gói EXE kiểm tra cập nhật từ GitHub Releases. Khi khởi động, bridge kiểm tra
nền và hiển thị phiên bản mới trên dashboard tại thẻ **Cập nhật ứng dụng**. Bấm **Cập
nhật** sẽ tải EXE có kiểm tra SHA-256, đóng bridge, thay EXE rồi chạy lại. Nếu mất mạng,
trạng thái chuyển thành *failed* và ghi log — không crash, không treo. Chạy từ source
không gọi GitHub.

Tắt kiểm tra nền khi cần:

```powershell
.\codex-plugin-bridge.exe --no-update-check
```

### Cài đặt bằng install.ps1

```powershell
.\install.ps1
```

Các tùy chọn:

```powershell
.\install.ps1 -NoAutostart            # không bật tự chạy cùng Windows
.\install.ps1 -NoShellIntegration     # không tạo shortcut / startup
.\install.ps1 -NoLaunch               # cài xong không tự mở dashboard
.\install.ps1 -InstallDirectory "$env:LOCALAPPDATA\CodexPluginBridge"
```

Installer mặc định: tạo HKCU startup entry, Start Menu shortcut, chạy bridge và mở dashboard. Bản release chạy nền khi Windows khởi động — không mở cửa sổ terminal.

### Kiểm tra bridge đang chạy

```powershell
Get-NetTCPConnection -LocalPort 8765 -ErrorAction SilentlyContinue
```

Mở dashboard tại:

```text
http://127.0.0.1:8765/setup
```

## Cài và kiểm tra Codex CLI

### Cài đặt

Sử dụng npm hoặc trình cài đặt chính thức của OpenAI:

```powershell
npm install -g @openai/codex
```

hoặc tải bản phát hành Codex CLI từ trang chính thức.

### Đăng nhập

```powershell
codex login
```

Codex phải đăng nhập để bridge chạy task thực tế.

### Kiểm tra từ dashboard

Mở `http://127.0.0.1:8765/setup`, xem trạng thái **Codex CLI**:

- `Chưa cài` — bridge chưa tìm thấy Codex; cài Codex hoặc đặt biến `CODEX_BRIDGE_CODEX_PATH`.
- `Sẵn sàng — <version>` — đã cài và đăng nhập.
- `Cần đăng nhập` — chạy `codex login`.

### Kiểm tra bằng MCP tool

Từ ChatGPT, gọi tool `check_codex_status` để xem trạng thái cài đặt và xác thực Codex CLI.

## Cài và kiểm tra Claude Code (Claude CLI) nếu chọn provider

Bridge có thể dùng **Claude Code** làm CLI provider cho task runner thay vì Codex CLI.

### Cài đặt

```powershell
npm install -g @anthropic-ai/claude-code
```

hoặc theo hướng dẫn chính thức của Anthropic.

### Chọn Claude làm CLI provider

1. Mở dashboard `http://127.0.0.1:8765/setup`.
2. Tại thẻ **CLI tác vụ**, chọn **Claude Code** trong mục "CLI đang dùng".
3. Chọn quyền tác vụ (`disabled` / `read-only` / `workspace-write`).
4. Bấm **Lưu CLI và quyền**.

Trạng thái Claude Code hiển thị ngay trên dashboard (tương tự Codex CLI): cài/chưa cài, sẵn sàng hay cần đăng nhập.

> Lưu ý: mỗi máy chọn một CLI provider. Đổi provider áp dụng ngay cho task runner; các Local Coder tools vẫn hoạt động độc lập.

## Kết nối MCP với ChatGPT

### Bước 1 — Chạy bridge

Chạy EXE (hoặc để tự chạy cùng Windows), mở `http://127.0.0.1:8765/setup`.

### Bước 2 — Chọn chế độ xác thực và tunnel

Trên dashboard:

1. **Public Tunnel:** chọn provider (Cloudflare Quick Tunnel / zrok Reserved Name / Local only).
2. **Chế độ xác thực:** `development-static-bearer` (Bearer token) hoặc `oauth-development` (dùng ChatGPT connector).

### Bước 3 — Lấy MCP URL

- **Local URL:** `http://127.0.0.1:8765/mcp/`
- **Remote URL:** URL từ tunnel + `/mcp/`, ví dụ `https://<tên>.trycloudflare.com/mcp/` hoặc `https://<tên>.share.zrok.io/mcp/`.

URL xuất hiện trên dashboard (có nút copy).

### Bước 4 — Thêm MCP server trong ChatGPT

Trong ChatGPT, thêm MCP server HTTP với:

- **Type:** `http`
- **URL:** Remote MCP URL (hoặc Local URL nếu ChatGPT chạy trên cùng máy).
- **Header:** `Authorization: Bearer <token>` (khi dùng chế độ Bearer).

Hoặc dùng **plugin / OAuth connector**: khi ChatGPT kết nối plugin, yêu cầu cấp quyền xuất hiện trên dashboard local — bấm **Cho phép** cho yêu cầu bạn vừa mở.

> Xác thực và scopes:
>
> | Scope | Quyền |
> |---|---|
> | `codex.read` | read/list/search/status/diff/task status/events |
> | `codex.write` | local write/edit tools |
> | `codex.task` | start/continue/cancel task |

## Tunnel

Bridge hỗ trợ nhiều provider, chỉ **một provider hoạt động tại một thời điểm**. Khi đổi provider, bridge dừng provider cũ trước khi chạy provider mới.

### Cloudflare Quick Tunnel

- Không cần tài khoản hay domain.
- URL dạng `https://<ngẫu nhiên>.trycloudflare.com` — **tạm thời**, có thể đổi khi tạo lại tunnel hoặc restart.
- Phù hợp development/testing.

### Cloudflare Managed Tunnel

- Dùng cho deployment có activation (tunnel token, public URL cố định).
- Tunnel token được bảo vệ bằng **Windows DPAPI** và truyền cho `cloudflared` qua child environment, không nằm trên command line.
- Port conflict trả `PORT_IN_USE`; bridge không âm thầm đổi port.

### zrok2 Reserved Name

- Dành cho dùng lâu dài với URL cố định: `https://<reserved-name>.share.zrok.io`.
- Bridge tự cài bản zrok2 đã pin và kiểm tra hash khi thiếu.
- Nhập **enable token** một lần (không được lưu vào config hay log) và chọn reserved name.

### Local only

- Không chạy public tunnel; chỉ dùng Local MCP URL `http://127.0.0.1:8765/mcp/`.

### cloudflared / zrok pin và verify

- `cloudflared` pin version và kiểm tra **SHA-256 trước khi chạy**.
- Khi bản đóng gói không có, bridge tải asset từ HTTPS allowlist rồi verify trước khi thay file đích.
- Retry policy: `1, 2, 4, 8, 15, 30, 60` giây; sau 10 lần fail liên tiếp, supervisor chuyển sang trạng thái `failed`.

## Sử dụng cơ bản

### Ví dụ prompt với ChatGPT

**Chỉ đọc project:**

```text
Dùng @Codex Plugin Bridge, chỉ đọc:
đọc project hiện tại, tóm tắt cấu trúc và entry point chính.
Không tạo, sửa, đổi tên hoặc xóa file.
```

**Kiểm tra trusted roots:**

```text
Dùng @Codex Plugin Bridge.
Gọi local_list_roots và cho tôi biết các trusted roots hiện tại.
Chỉ đọc.
```

**Chỉnh sửa nhỏ có kiểm chứng:**

```text
Dùng @Codex Plugin Bridge.
Đọc file config/example.json, sửa chính xác timeout từ 30 thành 60,
đọc lại file và kiểm tra local_git_diff.
Không sửa file khác.
```

**Task lớn bằng Codex / Claude:**

```text
Dùng @Codex Plugin Bridge.
Chạy task để phân tích toàn bộ project, sửa lỗi test đang fail,
chạy lại test liên quan và kiểm tra diff trước khi hoàn thành.
Không thay đổi file ngoài project.
```

### Danh sách MCP tools

| Tool | Mục đích |
|---|---|
| `local_list_roots` | liệt kê trusted roots + root IDs |
| `check_codex_status` | trạng thái Codex CLI (cài đặt / auth) |
| `check_claude_status` | trạng thái Claude Code CLI (cài đặt / auth) |
| `local_project_context` | thông tin project (instructions/manifests) |
| `local_list_directory` | liệt kê thư mục có giới hạn |
| `local_read_text_file` | đọc UTF-8 theo offset/max chars |
| `local_search_text` | tìm văn bản trong selected root |
| `local_write_text_file` | tạo/ghi văn bản khi write mode cho phép |
| `local_edit_text_file` | chỉnh sửa chính xác có kiểm chứng |
| `local_git_status` | Git status có giới hạn |
| `local_git_diff` | Git diff chưa stage |
| `run_codex_task` | bắt đầu task (Codex hoặc Claude tùy provider) |
| `continue_codex_session` | tiếp tục đúng session |
| `get_codex_task_status` | đọc task state/result |
| `get_codex_task_events` | đọc incremental bounded events |
| `cancel_codex_task` | hủy task/process tree |

Task lifecycle:

```text
queued → starting → running → completed
                         ├→ failed
                         ├→ cancelled
                         └→ timed_out
```

### Workspace mặc định

```text
<Documents Windows>\Codex
```

Override khi cần:

```powershell
$env:CODEX_BRIDGE_WORKSPACE = "D:\MyWorkspace"
```

## Quyền truy cập và access profile

Trên dashboard local (`/setup`):

### Access profile (được làm ở đâu)

- **SAFE** — chỉ workspace mặc định `<Documents>\Codex`.
- **EXTENDED** — thêm các thư mục tuyệt đối do user cấu hình (không phải reparse point). Root được expose bằng ID `root-0`, `root-1`, ...
- **TRUSTED FULL ACCESS** — toàn bộ máy cho task runner; phải nhập chính xác `TRUSTED FULL ACCESS` khi nâng quyền. Mặc định **tắt**.

### Command permission (được làm gì)

- **Local MCP tools:** `disabled` / `read-only` / `workspace-write`.
- **CLI tasks:** `disabled` / `read-only` / `workspace-write`.

> Access profile trả lời "được làm ở đâu"; command permission trả lời "được làm gì". Các thiết lập này **chỉ thay đổi tại máy Windows**; remote MCP không thể tự nâng quyền.

## Biến môi trường

| Biến | Tác dụng |
|---|---|
| `CODEX_BRIDGE_WORKSPACE` | override workspace mặc định |
| `CODEX_BRIDGE_PORT` | override local port (`1..65535`) |
| `CODEX_BRIDGE_LOCAL_TOOLS_MODE` | mode khởi tạo Local Coder cho config mới |
| `CODEX_BRIDGE_CODEX_PATH` | override Codex executable path |
| `CODEX_BRIDGE_CLAUDE_PATH` | override Claude executable path |
| `CODEX_BRIDGE_CLOUDFLARED_PATH` | override cloudflared executable |
| `CODEX_BRIDGE_CLOUDFLARED_SHA256` | override expected cloudflared hash |

## Lỗi thường gặp

### Codex / Claude CLI

| Mã lỗi | Ý nghĩa | Cách xử lý |
|---|---|---|
| `CODEX_NOT_FOUND` / `CLAUDE_NOT_FOUND` | không tìm thấy CLI | cài CLI hoặc đặt `CODEX_BRIDGE_CODEX_PATH` / `CODEX_BRIDGE_CLAUDE_PATH` |
| `CODEX_NOT_AUTHENTICATED` / `CLAUDE_NOT_AUTHENTICATED` | chưa đăng nhập | chạy `codex login` hoặc `claude` và đăng nhập |
| `CODEX_AUTH_EXPIRED` | auth/session hết hạn | đăng nhập lại |
| `CODEX_QUOTA_EXHAUSTED` | hết quota | chờ/reset quota; Local Coder vẫn độc lập |
| `CODEX_RATE_LIMITED` | rate limit | giảm tần suất và retry sau |
| `CODEX_SESSION_NOT_FOUND` | session/task không tồn tại | kiểm tra đúng ID |
| `CODEX_START_FAILED` | không start được process | kiểm tra executable/PATH/quyền |
| `CODEX_PROCESS_FAILED` | CLI process lỗi | đọc task events/result |
| `CODEX_TIMEOUT` | task quá hạn | giảm scope hoặc chia task |
| `CODEX_CANCELLED` | task đã cancel | start task mới nếu cần |
| `CODEX_TASKS_DISABLED` | task mode disabled | đổi permission tại dashboard local |

### Path / access

| Mã lỗi | Nguyên nhân thường gặp |
|---|---|
| `ROOT_NOT_ALLOWED` | root chưa trust hoặc root ID sai |
| `ACCESS_ROOT_INVALID` | trusted root không hợp lệ |
| `REPARSE_POINT_BLOCKED` | symlink/junction/reparse point |
| `SECRET_PATH_BLOCKED` | path thuộc secret policy |
| `NOT_A_DIRECTORY` / `NOT_A_FILE` | list/read trên sai loại đối tượng |
| `BINARY_FILE` | file binary (có NUL trong sample) |
| `NOT_UTF8` | file text không phải UTF-8 |

### Local permission

| Mã lỗi | Ý nghĩa |
|---|---|
| `LOCAL_TOOLS_DISABLED` | Local Coder disabled |
| `WRITE_NOT_ALLOWED` | Local Coder chưa ở `workspace-write` |

### MCP / OAuth

| Mã lỗi | Ý nghĩa |
|---|---|
| `MCP_UNAUTHORIZED` | Bearer/OAuth token thiếu hoặc sai |
| `OAUTH_INSUFFICIENT_SCOPE` | token thiếu required scope |
| `OAUTH_CONFIG_INVALID` | issuer/audience/JWKS/resource sai |
| `OAUTH_PUBLIC_URL_UNAVAILABLE` | Quick Tunnel chưa có public URL |
| `OAUTH_INVALID_RESOURCE` | resource không match bridge |
| `OAUTH_REDIRECT_REJECTED` | redirect URI không hợp lệ |

### `PORT_IN_USE`

Configured port đang bị process khác chiếm:

```powershell
Get-NetTCPConnection -LocalPort 8765 -ErrorAction SilentlyContinue
```

Managed deployment không tự đổi port âm thầm.

### Tunnel degraded/failed

Kiểm tra dashboard `/setup` và:

```text
%LOCALAPPDATA%\CodexPluginBridge\logs\cloudflared.log
```

Nguyên nhân thường gặp: mất Internet/DNS, cloudflared sai hash, download bị chặn, managed credential sai, port conflict hoặc retry policy đã exhausted.

## Bảo mật

- **Loopback-first:** HTTP server bind loopback; truy cập public đi qua outbound tunnel, không mở service inbound trực tiếp.
- **Local-only management:** `/setup`, `/setup.json`, `/tunnel/status`, `/admin/*` từ chối request được forwarded từ tunnel.
- **MCP authentication + least privilege:** mọi request MCP phải có Bearer hợp lệ; OAuth enforce scope theo tool.
- **Filesystem boundary:** Local Coder không nhận arbitrary absolute path; phải qua canonicalization, boundary, reparse và secret checks.
- **Secret redaction:** log và task history redact Bearer token, tunnel token, API key, client secret, access token, password và private-key marker.
- **DPAPI:** managed tunnel token được mã hóa gắn với Windows user context trước khi persist.
- **Verified cloudflared/zrok:** binary chỉ chạy khi hash khớp expected SHA-256; download chỉ chấp nhận HTTPS origin được allowlist.
- **Bounded operations:** read/write/list/search/event/history đều có limit để tránh tiêu thụ tài nguyên không kiểm soát.
- **Full access có chủ ý:** `trusted-full-access` yêu cầu xác nhận tại máy local khi elevation, không phải default, và không loại bỏ local path/secret filtering.

### Dữ liệu runtime

```text
%LOCALAPPDATA%\CodexPluginBridge
├── config.json          # cấu hình (Bearer, quyền, roots, activation đã bảo vệ)
├── logs\bridge.log      # log ứng dụng/security/task
├── logs\cloudflared.log # log tunnel
├── history\             # task metadata/events
├── oauth\               # OAuth development state
├── bin\cloudflared.exe  # binary tunnel đã verify
└── tunnel-state.json
```

## Gỡ cài đặt

Từ installed directory:

```powershell
.\uninstall.ps1
```

Giữ application files:

```powershell
.\uninstall.ps1 -KeepApplicationFiles
```

Uninstaller: remove HKCU startup entry, Start Menu shortcut, dừng bridge/cloudflared process tương ứng, có thể xóa application files. Giữ lại user configuration và task history theo thiết kế.

## Giấy phép và third-party

Phát hành theo **MIT License**.

```text
Copyright (c) 2026 Trần Đăng Khoa / TranDangKhoaAutomation
```

Release sử dụng/liên quan đến:

- **cloudflared** — Apache License 2.0.
- **zrok** — Apache License 2.0.
- **PyJWT** — MIT License.
- **cryptography** — Apache License 2.0 hoặc BSD License.
- **PyInstaller** — GPL 2.0 với exception cho bundled applications.

## Tác giả

**Trần Đăng Khoa**

GitHub: <https://github.com/TranDangKhoaAutomation>

> Khuyến nghị khi bắt đầu: dùng **SAFE + read-only**, kiểm tra `local_list_roots` và `check_codex_status`, sau đó chỉ nâng quyền khi workflow thực sự cần.
