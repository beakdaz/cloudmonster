# Mega v4.6 (Azazello1998) — reference from `D:\Mega\Mega_v4.6.exe_extracted`

Primary entry: **`Mega_v4.6.pyc`** (single large module, Python 3.13).  
Supporting: `PYZ.pyz_extracted/` (stdlib + deps only).

## Stack (not `mega-rs`)

| Piece | v4.6 | Our Rust port today |
|-------|------|---------------------|
| HTTP | **`curl_cffi.requests.AsyncSession`** impersonate **`chrome110`** + Chrome UA / Origin `https://mega.nz` | **`curl-impersonate`** subprocess (`curl_chrome110`) + hashcash in `third_party/mega-0.8` |
| API | POST JSON to **`https://g.api.mega.co.nz/cs`** (`API_URL`) | Same origin via `mega-rs` |
| Anti-bot | **`_request_with_hashcash`**: HTTP **402** + header **`X-Hashcash`**, solve and retry | Implemented in `curl_impersonate.rs` (402 + `X-Hashcash` retry) |
| Login | Custom **`AsyncMegaClient.login`**: `us0` / v2 PBKDF2, RSA, sid | `mega::Client::login` (same protocol family) |
| Concurrency | **`asyncio`**: `account_producer` → queue → **`account_consumer`**, **`asyncio.Semaphore`** passed as **`sem_login`** into **`relogin_account`** | `tokio` + `Semaphore(threads)` + **`download_mega_account` per task** |
| Proxy | UI **`proxy_mode`**: `no_proxy` / file / link (optional) | MEGA path: **no proxy wiring yet** |
| Results | **`Good.txt` / `Bad.txt` / `Banned.txt`**, stats line matches GUI | **`Good.txt`** + GUI Valid count |

## Check flow (checker)

1. **`run_processing_async(settings, …)`** — reads accounts, optional proxies, opens result buckets.
2. **`relogin_account(email, password, proxy_protocol, check_mode, sem_login, …)`**  
   - Optional proxy via **`parse_proxy_url`**.  
   - **`AsyncMegaClient.login`**.  
   - On failure, **`re.search`** on error text:  
     - **`(-16|-26|-28|-27)`** → banned-like  
     - **`(-9|-15|-14|-11|-2|-13)`** → bad credentials  
   - Success → **`'good'`**.
3. **`process_one_account`** — maps status to **`good` / `bad` / `banned` / `lowcost` / `not_found`**, writes lines, optional download.

## Why 50 threads without proxy can work in v4.6

- **Browser-like TLS** (curl_cffi) + **hashcash** → API answers with normal JSON instead of drowning in opaque HTTP failures.
- **`sem_login`** limits concurrent **login** sessions even when “Threads (Checker)” is 50.
- Most lines are **Banned/Bad** (cheap error codes), not full successful sessions.

## Rust checker setup (Windows)

1. **curl-impersonate 2.2.3** source tree: `D:\Mega\curl-impersonate-2.2.3`
   - Wrappers: `win\bin\curl_chrome110.bat`
   - You still need **`win\bin\curl-impersonate.exe`** (build with `win\build.bat`, or copy from an x86_64 Windows release tarball next to the `.bat` files).
2. Launch with **`run_export-mega-gui.bat`** (sets `MEGA_CURL_ROOT` / `MEGA_CURL_BIN`), or manually:
   ```bat
   set MEGA_CURL_ROOT=D:\Mega\curl-impersonate-2.2.3
   set MEGA_CURL_BIN=%MEGA_CURL_ROOT%\win\bin\curl_chrome110.bat
   ```
3. `update_export-mega-gui.bat` → run `export-mega-gui.exe`.

See **`setup_mega_curl.bat`** for a short checklist.

**Do not use** `curl-impersonate-v2.2.3.arm64-win32` on normal AMD64 PCs — wrong architecture. Prefer the 2.2.3 tree above or an **x86_64-win32** binary drop in `win\bin`.

Without curl-impersonate you will see **`maximum retries`** on every account (MEGA blocks plain TLS).

1. **HTTP layer**: Chrome impersonation + hashcash (port `_request_with_hashcash` / `_solve_hashcash`), or embed curl-impersonate / `rquest`-style client — **not** plain `reqwest` alone.
2. **Orchestration**: producer/consumer + login semaphore like v4.6.
3. **Buckets**: map API codes to Good/Bad/Banned like **`relogin_account`** regex + **`process_one_account`**.

Extract strings/constants from pyc:

```powershell
python -c "import marshal; f=open(r'D:\Mega\Mega_v4.6.exe_extracted\Mega_v4.6.pyc','rb'); f.read(16); c=marshal.load(f); print([x for x in c.co_consts if isinstance(x,str) and 'api' in x.lower()][:5])"
```
