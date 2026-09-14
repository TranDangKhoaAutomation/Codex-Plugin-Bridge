# Plugin Bridge

**Latest stable:** `v0.9.14` · **Windows 10/11** · **Standalone EXE**

Plugin Bridge connects ChatGPT Web to a Windows computer through MCP so ChatGPT can work with local projects, run deterministic local commands, build/test code, and optionally delegate tasks to Codex CLI or Claude Code.

## Download and install

For normal users, download this one file and double-click it:

**[Download PluginBridge-Setup-0.9.14.exe](https://github.com/TranDangKhoaAutomation/Codex-Plugin-Bridge/releases/download/v0.9.14/PluginBridge-Setup-0.9.14.exe)**

Release page: <https://github.com/TranDangKhoaAutomation/Codex-Plugin-Bridge/releases/tag/v0.9.14>

### No Python installation required

`PluginBridge-Setup-0.9.14.exe` is a PyInstaller **single-file Windows build**. Python, pip and the Plugin Bridge Python dependencies are bundled; the runtime assets needed by Plugin Bridge, including the bundled Cloudflare helper and local browser-extension assets, are embedded in the installer.

You do **not** need to install Python or pip before running Plugin Bridge.

Codex CLI and Claude Code are optional. Install one of them only if you deliberately choose that provider. The **Local · ChatGPT Web** provider does not require either CLI.

## First run

1. Download `PluginBridge-Setup-0.9.14.exe`.
2. Double-click the EXE.
3. It installs per-user to:

   ```text
   %LOCALAPPDATA%\PluginBridge\plugin-bridge.exe
   ```

4. Plugin Bridge starts in the background and opens the local setup dashboard:

   ```text
   http://127.0.0.1:8765/setup
   ```

5. Select `Local`, `Codex`, or `Claude`, configure the access profile and tunnel as required, then use the MCP URL shown on the dashboard.

A second copy named `plugin-bridge.exe` is also kept at the repository root for convenience, but the versioned Release asset above is the recommended download.

## WinError 5 / Access is denied

Older builds could fail while replacing a running executable under the legacy path:

```text
%LOCALAPPDATA%\CodexPluginBridge
```

The current installer path is `%LOCALAPPDATA%\PluginBridge`. For an existing current installation, the installer now:

1. asks the running bridge to stop;
2. waits for the installed executable to release;
3. if necessary, force-stops only the matching installed executable;
4. stages the new executable to a temporary file;
5. replaces it atomically;
6. retries transient Windows file-lock errors including **WinError 5** and **WinError 32**;
7. returns a controlled install timeout instead of exposing a raw unhandled `PermissionError` when Windows still refuses replacement.

Legacy configuration is migrated from the old `CodexPluginBridge` data directory when the new configuration does not yet exist.

## Release verification

The `v0.9.14` executable published here came from the verified Windows CI artifact. The verification pipeline completed:

- Ruff formatting check: pass
- Ruff lint: pass
- mypy strict type check: pass
- Python compile check: pass
- **410 pytest tests: pass**
- source credential/secret scan: pass
- standalone EXE build: pass
- `STANDALONE_EXE_LOCAL_FULL_SMOKE_PASS`
- `SELF_INSTALL_SINGLE_EXE_SMOKE_PASS`
- `UPGRADE_EXISTING_INSTALL_SMOKE_PASS` (v0.9.10 → v0.9.14)
- `PORT_CONFLICT_SMOKE_PASS`
- `MANAGED_PACKAGE_SMOKE_PASS`
- release secret scan: pass

### SHA-256

```text
e36bb3d26a331a2cf4e2b19d6968ee1a7ddc9c64c05b665a00ac26f2022e831c  PluginBridge-Setup-0.9.14.exe
```

The repository-root `plugin-bridge.exe` is the same verified executable bytes and has the same SHA-256.

To verify after download:

```powershell
Get-FileHash .\PluginBridge-Setup-0.9.14.exe -Algorithm SHA256
```

Expected result:

```text
E36BB3D26A331A2CF4E2B19D6968EE1A7DDC9C64C05B665A00AC26F2022E831C
```

## Portable mode

To run without self-installing:

```powershell
.\PluginBridge-Setup-0.9.14.exe --portable
```

For normal end users, use the default double-click installation instead.

## Notes

- Plugin Bridge itself is standalone. Internet access is still required for features that inherently use Internet services, such as a public tunnel.
- Codex CLI / Claude Code are optional external providers and are not required for Local mode.
- The management dashboard is local-only; remote ChatGPT access should use the MCP endpoint and the configured tunnel/authentication path.

## License

MIT. See `LICENSE` and `THIRD_PARTY_NOTICES.md`.
