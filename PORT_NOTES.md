# Port Notes

## Current state (2026-09-20)

Rust workspace porting the decompiled C# WPF app **PhpMyAdmin Export** from `project_final/`, plus reference material for the **Mega v4.6** PyInstaller bundle under `D:\Mega\pyc\` (MEGA.nz client — see `SOURCES.md`).

## Source roots

| Path | Role |
|------|------|
| `project_final/` | Main C# reference for export modules |
| `project/` | Secondary C# tree (proxy pool, etc.) |
| `D:\Mega\pyc\` | `Mega_v4.6` — cloud upload/check/proxy (future port) |

Multi-root workspace: `..\MEGA.code-workspace` (adds `D:\Mega` beside `Documents\MEGA\1`).

| C# source | Rust module |
|-----------|-------------|
| `_PnNZuayyjAZTWlDfE5rUmZjqyYg.cs` (TargetCredential) | `pma-core::parser` |
| `_tzYCdx01jabRi9YINoZruASmPDF.cs` (PhpMyAdminClient) | `pma-net::client` |
| `_xULUmRrI4p3f9lVXdlb81HoogZB.cs` (BrowserHttpClient) | browser headers + random Chrome UA in `pma-net` |
| `_vQgh0sjaucl3pGnxc3VkckNe2Bc.cs` (MainViewModel) | `pma-app::process_targets_combined` |
| `_WzzQf3bdg5M1xPP01AThAAOmtij.cs` (LoginResult) | `LoginOutcome` + `ScanStatus::Captcha` |
| `_TYt5UiueQJPld0Vj3scfuVKB51s.cs` (thread pool) | `tokio` + `Semaphore` |
| WPF MainView | `phpmyadmin-rs-gui` (egui) |

## C# → Rust mapping

| Original concept | Rust crate / module |
|------------------|---------------------|
| Target list `url:user:pass` | `pma-core::parser` |
| Short-credential host filter | `Credential::should_skip_short_credentials` |
| Login (3 retries, cookies, Refresh) | `PhpMyAdminClient::login` |
| Server-wide SQL export | `PhpMyAdminClient::export_server_to_file` |
| Result buckets (`Good.txt`, …) | `pma-app::append_result_bucket` |
| Combined login+dump pass | `process` CLI / `process_targets_combined` |
| Separate scan then dump | `scan` + `dump` / `run` CLI |
| Desktop UI | `pma-gui` library; binaries below |
| All modes (picker) | `target/release/phpmyadmin-rs-gui.exe` (`crates/pma-gui`) |
| One mode per exe | `apps/export-<mode>-gui` → `target/release/export-<mode>-gui.exe` (14 modes incl. MEGA.nz) |

Build all GUI binaries: `build_gui_release.bat` or `update_gui_all.bat`.

Per-mode release rebuild (same as above, one exe): `update_phpmyadmin-rs-gui.bat`, `update_export-<mode>-gui.bat` (e.g. `update_export-mega-gui.bat`). Output: `target\release\`.

## Stack

| Concern | Choice |
|---------|--------|
| HTTP | `reqwest` (rustls, cookies) |
| Async runtime | `tokio` |
| CLI | `clap` subcommands: `scan`, `dump`, `run`, `process` |
| GUI | `eframe` + `egui` |

## Still not ported

- Proxy pool (`_3LBKzY64OpTHsVWd3iaEu1vldEe.cs`, `_G4HZl6xJwrcuDmIkPCZEv6KhSrN.cs`)
- Registry settings persistence
- LiveCharts statistics / ETA
- Full ~50 export form fields (subset used; enough for quick server export)
- Drag-and-drop file loading in GUI

## Usage (C# equivalent)

Original app: load list → Start → writes `Results/{timestamp}/Good.txt` + `{host}.sql`.

Rust equivalent:

```bash
cargo run -p phpmyadmin-rs -- process -i targets.txt -o .
```

## Secrets

No credentials are hardcoded. Read targets from list files; do not commit real target lists.
