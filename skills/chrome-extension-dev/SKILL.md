---
name: chrome-extension-dev
description: >
  Develops Chrome Extensions (Manifest V3) using vanilla JavaScript without
  build tools, package managers, or TypeScript. Use when the user wants to
  create a new Chrome extension, add features to an existing one, refactor
  extension code, or troubleshoot Chrome WebExtension API issues. Applies to
  requests involving manifest.json, service workers, content scripts, popup UI,
  message passing, storage, and chrome.* APIs.
---

# Chrome Extension Developer

Read first: also load `agentic-code-local`. This skill adds Chrome-extension rules on top of it. On conflict, this skill wins. If still unclear, ask the user.

Ready-to-use patterns (manifest, service worker, content script, popup, storage, message flow, permission checklist) are in [`references/chrome-extension.md`](references/chrome-extension.md). Read only the section you need.

## Core Constraints

Never use: NPM, package.json, node_modules, build tools (Webpack, Vite, esbuild), TypeScript, external packages, dist/ or build/ folders.

Always use: vanilla JavaScript, Manifest V3, files split by responsibility. No build step: manifest + JS files, load unpacked in Chrome.

If a task needs installation or compilation, stop and ask the user.

## Project Structure

Start flat for a small extension:

```
my-extension/
├── manifest.json
├── background.js   (service worker)
├── content.js      (page injection)
├── popup.html
├── popup.js
└── icons/
```

When responsibilities grow, split into subfolders per entry point (`background/`, `content/`, `popup/`) instead of growing one file.

## Rules

- The service worker has no DOM, `window` or `document`.
- Use `chrome.storage.local`, never `localStorage`.
- Content scripts run isolated from page JS. Use `window.postMessage` to talk to the page.
- Guard against double injection and use the message-passing shape from the reference file.
- Put an architecture note at the top of each entry-point file: its modules and the data flow.

## Splitting files

- **background**: set `"type": "module"` on the `background` key in `manifest.json`, then use `import`/`export`.
- **content scripts**: list several files in `content_scripts[].js`. They run in order in one shared scope, so keep shared state under one object (for example `window.__ext`).
- **popup**: `<script type="module" src="popup.js">` and `import` from other files.

## Gotchas

- A bad `import` path or `content_scripts[].js` path fails silently at runtime. Check paths and reload after every split.
- Content scripts cannot use static `import`. Dynamic `import()` needs `web_accessible_resources`, which exposes the file to web pages. Avoid it.
- Service worker memory does not survive a restart (about 30s idle). Keep state in `chrome.storage`, not in module variables.

## Setup Checklist

Before writing code, confirm: what the extension does, which tabs or pages it affects, whether it needs a popup, whether it needs storage. Write only the files that are needed.
