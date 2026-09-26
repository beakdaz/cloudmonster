# Decompiled / reference sources

Paths used when porting behavior to Rust. Keep these trees on disk; they are not required at runtime for the GUI/CLI.

| Root | Contents | Used for |
|------|----------|----------|
| `project_final/` | C# WPF **PhpMyAdmin Export** (de4dot + decompiler) | Primary port: parsers, HTTP, GUI flow |
| `project/` | Alternate de4dot output of the same family | Diff / proxy-pool files (`_3LBKzY64…`, `_G4HZl6x…`) |
| `D:\Mega\Mega_v4.6.exe_extracted\` | PyInstaller extract **Mega v4.6** (`Mega_v4.6.pyc`) | Reference for checker; Rust uses **`third_party/mega-0.8`** (hashcash + Chrome headers) |
| `D:\Mega\pyc\` | Older/alternate extract path | Same family if present |

## Opening both trees in Cursor

Use multi-root workspace:

`C:\Users\beakd\Documents\MEGA\1\MEGA.code-workspace`

(File → Open Workspace from File…)

## Optional junction (same repo folder)

To browse `D:\Mega` under this repo without a workspace file:

```bat
mklink /J "C:\Users\beakd\Documents\MEGA\1\phpmyadmin-rs\source_mega_d" "D:\Mega"
```

Then reference `D:\Mega\Mega_v4.6.exe_extracted\Mega_v4.6.pyc` and `MEGA_V46_REFERENCE.md`.
