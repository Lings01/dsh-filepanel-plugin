# dsh-filepannel-plugin

> A workspace file panel for the DeepSeek Harness (DSH) — manage the files of the current workspace right inside the browser UI.

[English](README.en.md) | [中文](README.md)

A **dynamic Cordis plugin** for the DSH web shell. Hover the right edge of the screen and a file panel slides out, giving you browse / upload / download / preview / edit / search / archive — every file operation for the current workspace, all in the browser.

The UI uses **SVG line icons (no emoji)** and is fully themed with **DSH design tokens** (`--dsw-alias-*`), automatically adapting to light/dark themes.

## Features

| Category | Details |
| --- | --- |
| Entry | Right-edge hotzone + tab: hover 0.3s to slide out, leave 0.5s to auto-hide; drawer width is drag-resizable |
| Browsing | Breadcrumbs, parent directory, refresh, in-directory filter, directories first |
| Transfer | **Streaming upload** (1MB chunks, progress bar + speed + cancel, up to 2GB, atomic temp-file replacement), download, open in a new browser tab |
| Preview | PDF (in-iframe reader), images, text/code (line numbers + syntax highlight + edit with Ctrl/Cmd+S) |
| File ops | New file/folder, rename, delete (confirmed), copy path, move/copy to another directory (directory picker) |
| Batch | Multi-select → batch download / archive / move / copy / delete |
| Search | **Recursive workspace search**: file names + text content, one click to jump |
| Archive | Pack selected items to zip; one-click unzip of `.zip` files |
| Upload | Button picker + **drag & drop** straight into the panel |

## How it works

A **Host + Client** dynamic Cordis plugin:

- **Host half** (`src/host.js`):
  - Registers 15 Package-private RPC methods via `harness.handle` (list, search, read/write, move/copy, zip/unzip, streamed upload, …)
  - Registers the same-origin download route `/__dsh__/filepanel/download` on `webServer`, guarded by a one-time token + workspace containment, supporting `inline` preview (images/PDF) and attachment downloads
  - Every path is constrained inside the session workspace via `fs.contains`; writes honor the session sandbox policy (`workspace-write`)
- **Client half** (`src/client.js`):
  - Registers into the `shell.overlay` slot (a floating layer; no shipped UI is replaced)
  - React components + inline SVG icons + DSH theme tokens — no build step, no external dependencies

### RPC methods (`panel.*`)

| Method | Description |
| --- | --- |
| `list` | List a directory (name/type/size/absolute path) |
| `search` | Recursive search (names + content of text files ≤256KB; depth ≤8, ≤200 results) |
| `readText` / `writeText` | Read (≤512KB preview) / atomic write |
| `createDir` / `remove` / `rename` | mkdir / delete / rename |
| `move` / `copy` | Batch move / recursive copy (with name-conflict checks) |
| `zip` / `unzip` | Pack (`zip -r`) / unpack (`unzip -o`) |
| `uploadStart` / `uploadChunk` / `uploadAbort` | Streamed upload (base64 chunks to a temp file, atomic rename on finish) |
| `token` | One-time token for the download route |

## Install & use

Currently shipped as a **DSH dynamic plugin** (no deployment changes, no build):

1. Call `cordis_define` in a DSH session with the bodies of `src/host.js` and `src/client.js`.
2. Call `cordis_run` to activate (the Client half needs approval the first time).
3. Hover the **right edge of the screen** for a moment — the panel slides out; or click the folder tab on the edge.

> Note: dynamic plugins are process-local; they need to be redefined after a restart. To persist, mount the plugin in the deployment's Cordis composition (`cordis.yml`).

## Dependencies & limits

- Host services used: `fs`, `shell`, `webServer`, `sandboxPolicy` (standard DSH capabilities)
- Archiving requires the system `zip` / `unzip` commands
- Uploads up to 2GB; previews: text 512KB, images/PDF 256MB; global search limited to depth 8 / 4000 nodes / 200 results
- "Open with system app" is unavailable on headless servers (a friendly hint is shown — use in-panel preview or browser open instead)

## Development

```text
dsh-filepannel-plugin/
├── src/
│   ├── host.js        # Host half (RPC + download route)
│   └── client.js      # Client half (panel UI)
├── README.md          # 中文文档
├── README.en.md       # English docs
└── LICENSE            # MIT
```

Before opening a PR: no emoji, theme tokens only for colors, plain JavaScript (no TS/JSX/build).

## License

[MIT](LICENSE)
