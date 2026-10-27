# Android Toolkit

A desktop dashboard for managing Android phones, Android TV and Fire TV devices through **Android Debug Bridge (ADB)**. Built with Electron, React and TypeScript.

![License: MIT](https://img.shields.io/badge/license-MIT-green)
![Version](https://img.shields.io/badge/version-2.0.0-blue)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)

> **Status:** v2.0.0 source rebuild. The Windows installer has not been built and the GitHub Actions workflows have not been run yet. See [Known limitations](#known-limitations).

---

## Features

| Module | What it does |
|---|---|
| **ADB Connection** | Wireless pairing, connect / disconnect, list devices, recent IP addresses |
| **APK Tools** | Install a single APK, batch-install every APK in a folder, one-click installs of selected apps |
| **TV Tools** | Screensaver timeout settings, Fire TV debloat list |
| **ReVanced Tools** | Helpers for installing MicroG, ReVanced Manager and patchable APKs |
| **System Tools** | Device info (battery, memory, CPU, Wi-Fi, disk, OS version), screenshot, terminal shortcut |
| **Command output panel** | Live output of every action inside the app |

## Tech stack

- **Electron** (main process, IPC) with a context-isolated preload bridge
- **React 17 + TypeScript** with Material UI
- **Webpack** (Electron React Boilerplate layout)
- **electron-builder** for Windows (NSIS), macOS and Linux (AppImage) packages
- **Jest + Testing Library** for tests

## How it works

```
 React UI (renderer)
      │  window.api.adbCommand / shellCommand / coms
      ▼
 preload.js  (contextBridge, no Node access in the UI)
      │  ipcRenderer.send
      ▼
 ipcHandler.ts  (main process)
      ├─ validates commands (allow-list, address and pairing checks)
      ├─ downloads APKs to the per-user app data folder
      └─ runs ADB (executeCmd.ts) and replies to the UI
```

## Requirements

- Node.js 20 LTS and npm
- Android platform-tools (ADB). The packaged Windows app downloads them on first run. In development, `adb` must be on your `PATH`.
- A device with **Developer options** and **USB debugging** (or **Wireless debugging**) enabled

## Getting started

```bash
git clone https://github.com/harpreetsingh171190-cmyk/Android-Toolkit.git
cd Android-Toolkit
npm install
npm start
```

### Build an installer

```bash
npm run package-win      # Windows (run on Windows)
npm run package          # current OS
```

The output is written to `release/build/`. Build on the operating system you are targeting.

### Build with GitHub Actions

1. Push the project to your GitHub repository.
2. Open **Actions → Build Windows installer → Run workflow**.
3. Download the `Android-Toolkit-Windows-Installer` artifact when it finishes.

> Commit `package-lock.json` after your first `npm install` so CI can use reproducible installs.

## Connection checklist

1. Enable Developer options and USB debugging on the device.
2. Connect by USB, or enable Wireless debugging and pair first.
3. Accept the debugging prompt on the device.
4. Click **List Devices** in the app to confirm the connection before installing anything.

## Project structure

```
src/
  main/            Electron main process
    api/           IPC handlers
    utils/         ADB execution, downloads, batch install, updater
    constants/     Paths, URLs, platform settings
  renderer/        React UI (components, theme, styles)
  __tests__/       Jest tests
.erb/              Webpack and build scripts
.github/workflows/ CI: quality check, installer build, release publish
assets/            Icons and ASCII art
```

## Security design

- The UI has no direct Node.js access. It talks to the main process through `contextBridge` only.
- `shellChannel` accepts only a fixed set of actions (`powershell`, `terminal`, `update`).
- ADB commands from the UI are checked against an allow-list and a blocked-character list. `adb connect` addresses and pairing codes are validated.
- `adb pair` runs with `spawn` and sends the code on stdin, so no shell is involved.
- Production builds ship with a Content Security Policy.
- APKs are saved in the per-user app data folder, not in a system folder.
- The in-app updater stays disabled until `repository.url` in `package.json` points to your own repository.

## Known limitations

These come from a code review of this source. Details are in `Android-Toolkit-Review.md` and the analysis notebook.

- **Electron 25 is end-of-life.** Upgrade it before publishing a release.
- **Third-party APK links are not verified.** Several links in `src/main/constants/index.ts` point to Google Drive, a temporary MediaFire URL, forum attachments or mirror sites. Check each link yourself, and prefer official GitHub releases. Downloads currently have no hash or file-type check.
- **Commands run through a shell string** (`exec`). The validator blocks common shell characters but also rejects some valid file names (for example `R&D.apk`). Moving to `execFile` with argument arrays is the planned fix.
- The Fire TV **debloat button runs 60 `pm disable-user` commands at once** with no confirmation. Use it only on devices you own and understand.
- On first Windows launch, the ADB server may start before the platform-tools download finishes.
- Only one smoke test exists. Unit tests for the command validators are on the roadmap.
- The Linux/macOS updater runs `install.sh` from the repository's `main` branch. Replace it with signed release assets before distributing the app.

## Roadmap

- [ ] Replace unverified APK links; add SHA-256 checks and download validation
- [ ] Switch to `execFile` with structured commands
- [ ] Add a confirmation step and sequential run for debloat
- [ ] Unit tests for validators and URL handling
- [ ] Upgrade Electron, React and build tooling
- [ ] Verify and merge the CI workflows; commit the lockfile
- [ ] Signed releases and a safer updater

## Analysis notebook

`Android_Toolkit_Analysis.ipynb` reads the project zip and reports file and line counts, dependencies, IPC channels, ADB commands, download-link sources and a test of the command validator. It works in Google Colab (it asks you to upload the zip) or in local Jupyter with the zip next to the notebook. It reads source code only and never runs ADB or opens any link.

## Responsible use

Use this tool only on devices you own or are authorized to manage. Some actions (debloat, system settings) can disable device features, so back up important data first. Third-party apps and links can change or disappear; download software only from sources you trust.

## Credits

This project is based on the open-source **Android Toolkit** by **Anthony Gress**, released under the MIT License. The v2.0.0 rebuild updates the dashboard UI and applies review fixes. The original copyright notice is kept in [LICENSE](LICENSE).

## License

[MIT](LICENSE)
