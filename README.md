# CloudMonster by Leakbase (MEGAX)

Проверка облачных аккаунтов: **login + инвентарь файлов**, вывод в **`Содержимое`**.  
Десктоп: **`export-mega-gui.exe`** (Rust / egui).

> Используйте только там, где у вас есть явное разрешение на тестирование.

---

## Быстрый старт

```powershell
cd G:\cloudmonster
$env:CARGO_TARGET_DIR = "G:\cloudmonster\target"   # опционально: артефакты на диск G:
cargo build -p export-mega-gui --release
.\target\release\export-mega-gui.exe
```

1. **Checker** → **Cloud service** → файл combo → **Start**  
2. Обычный формат: **`email:password`** (одна строка — один аккаунт)  
3. Результаты: папка вывода → **`Содержимое`** → `N_FILES_…txt`

**Профили:** Settings → Profiles → `%LOCALAPPDATA%\CloudMonster\profiles.json`

### Для пользователя без исходников

| Компонент | Назначение |
|-----------|------------|
| `export-mega-gui.exe` | CloudMonster GUI |
| [curl-impersonate](https://github.com/lwthiker/curl-impersonate/releases) (`win\bin`) | TLS/HTTP для облачных чекеров |
| [VC++ Redistributable x64](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist) | runtime |

Переменные: **`MEGA_CURL_ROOT`** или **`MEGA_CURL_BIN`**.  
Подробнее: **[docs/CLOUDMONSTER_USER_MANUAL.ru.md](docs/CLOUDMONSTER_USER_MANUAL.ru.md)** и вкладка **Help** в GUI.

---

## Облачные сервисы (32)

В GUI в списке **Ready** — все, у кого `CloudProvider::is_checker_ready()` (сейчас **без** Proton Drive и ToffeeShare).

Полная таблица (mail:pass vs token/cookie, листинг файлов, ограничения):

**[docs/CLOUD_MODULES.ru.md](docs/CLOUD_MODULES.ru.md)**

Кратко по enum и URL: `crates/core/src/cloud.rs`  
Маршрутизация: `crates/net/src/cloud/mod.rs` → `download_cloud_account`

| Группа | Сервисы |
|--------|---------|
| Основные combo | MEGA, Filen, Degoo, MediaFire, 4shared, Blomp, Files.fm, TeraBox, PixelDrain, Raindrop, InternXt, OneDrive, WPS, Sync.com, Koofr, Box, FEX, Icedrive, pCloud, WorkUpload, … |
| Особый формат пароля | **file.io** → `fileio:API_KEY`; **MobiDrive** → `mobidrive:…`; **UpNote** → `upnote:TOKEN`; **SecureShare** → `secureshare:COOKIE` |
| Login без полного листинга | **RemNote** (Firebase), **Acronis** (password grant), **Obsidian** (`ob` CLI / `OBSIDIAN_AUTH_TOKEN`) |
| Не combo-облако | **ToffeeShare** (P2P), **Proton Drive** (SRP/2FA — в разработке) |

---

## Структура кода (cloud)

```text
crates/core/src/cloud.rs              # CloudProvider, specs, is_checker_ready
crates/net/src/cloud/
  common.rs, config.rs, curl_http.rs  # общий pipeline, curl-impersonate
  dispatch_download.rs, download.rs
  providers/                          # один .rs = один сервис Platforms
    mod.rs, README.md
    four_shared.rs, terabox.rs, mediafire.rs, …
crates/pma-gui/src/megax.rs           # CloudMonster UI
apps/export-mega-gui/                 # точка входа exe
docs/CLOUD_MODULES.ru.md            # статус модулей
tools/REMNOTE-FIREBASE.md             # RemNote API key
```

### Pipeline

```text
megax.rs → pma-app → pma-net::download_cloud_account
  → providers::<service>::check_account
  → common::run_checker → login + inventory → Содержимое/*.txt
```

В файле результата: `DATA = email:pass | FILES = … | USED QUOTA = … | TARIFF = …` и список имён (до ~20 000 строк).

---

## Заметки по сервисам

| Сервис | Важно |
|--------|--------|
| **MEGA** | Upload только MEGA; combo также `email password` |
| **4shared** | Листинг через direct; при geo `dc.id` — прогрев `dc*.4shared.com`; датацентр-IP часто режет API |
| **TeraBox** | Passport + RSA/AES; модуль `providers/terabox.rs`. Share-ссылки — отдельные проекты (напр. [terabox-gateway](https://github.com/saahiyo/terabox-gateway), cookie `ndus`, не mail:pass) |
| **OneDrive** | Microsoft; 2FA / rate limit → auth или unreachable |
| **WorkUpload** | `email:password` + PoW captcha (не хвост `:https://…` в combo) |
| **RemNote** | Нужен **`REMNOTE_FIREBASE_API_KEY`** — см. `tools/REMNOTE-FIREBASE.md` |
| **InternXt** | Шифрование пароля (PBKDF2) |
| **ADrive** | WebDAV; free часто 401 |

---

## Прокси (Checker)

- **Direct** — без прокси (для отладки 4shared и др.)  
- Файл прокси / single proxy / SOCKS5  
- **Download Settings:** фильтры расширений, размер, потоки  

---

## Сборка и тесты

```powershell
cargo build -p pma-net
cargo build -p export-mega-gui --release
```

Локальный smoke (combo **только** в env, не в git):

```powershell
$env:FOUR_SHARED_TEST_COMBO = "email:password"
cargo test -p pma-net four_shared_local_smoke -- --ignored --nocapture
```

Другие ignored-тесты: `degoo_local_smoke`, `onedrive_local_smoke`, `internxt_local_smoke`, `remnote_local_smoke`, `terabox_local_smoke`, …

---

## UI и layout

| Что | Где |
|-----|-----|
| Интерфейс | `crates/pma-gui/src/megax.rs` |
| Layout | `tools/megax-layout.json`, `tools/apply_megax_layout.ps1` |
| Figma (editable) | [MEGAX Checker GUI](https://www.figma.com/design/iHwbtsMaNCiQFNgpp5NHiJ/MEGAX-Checker-GUI-editable) |

---

## Прочие GUI в workspace

Помимо CloudMonster, в репозитории есть отдельные `apps/export-*-gui` (phpMyAdmin, GitLab, Jenkins, …). См. корневой `Cargo.toml` → `[workspace.members]`.
