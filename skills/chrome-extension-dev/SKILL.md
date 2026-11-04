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

⚠️ Read First — also load `agentic-code-local` (general code
quality) and `code-standards` (general rules). This skill
adds Chrome-extension-specific rules and constraints on top of both —
where they conflict, this skill wins (if still unclear, ask the user).

Full copy-pasteable patterns (manifest.json, service worker, content
script, popup, storage, message-flow, permission checklist) live in
[`references/chrome-extension.md`](references/chrome-extension.md) — read
it now, before writing code. This file covers only the rules and gotchas;
that one covers the code.

## Core Constraints

**Never use:**
- NPM, package.json, node_modules
- Build tools (Webpack, Vite, esbuild)
- TypeScript
- External packages (lodash, axios, etc.)
- dist/ or build/ directories

**Always use:**
- Vanilla JavaScript only
- Manifest V3
- Files split by responsibility (subfolders like `background/`, `content/`,
  `lib/` are fine — no package.json or build step required for a nested
  layout)
- Maximum 500 lines per file (a ceiling, not a reason to wait before splitting)

## Project Structure

```
my-extension/
├── manifest.json
├── background.js       (service worker)
├── content.js          (page injection)
├── popup.html          (popup UI, if needed)
├── popup.js            (popup script, if needed)
└── icons/              (extension icons)
```

That flat layout is the starting point for a small extension. As responsibilities
grow, split into subfolders per entry point instead of growing one file:

```
my-extension/
├── manifest.json
├── background/
│   ├── index.js         (entry — "type": "module" in manifest.json)
│   ├── storage.js
│   └── messaging.js
├── content/
│   ├── main.js          (first file in content_scripts[].js)
│   └── utils.js
├── popup/
│   ├── popup.html
│   ├── popup.js         (entry — <script type="module">)
│   └── ui.js
└── icons/
```

No package.json. No build step either way. Manifest + JS files → drag into Chrome → works.

## Critical Rules

### Manifest V3 specifics
- Service worker cannot access DOM, window, or document
- Storage: use `chrome.storage.local` only, never `localStorage`
- Content script runs isolated from page JS — use `window.postMessage` for cross-realm communication

### Preventing double-injection, and message passing

Guard pattern and the full send/listen shape are in
`references/chrome-extension.md` — don't reproduce them here, use them
as written from that file.

### File size & splitting

Split by responsibility, not by line count — a 150-line file mixing storage,
messaging, and UI logic should still be split. Don't wait for 400+ lines as
an excuse to keep unrelated concerns in one file. Each of MV3's three
contexts has a build-tool-free way to split cleanly:

- **background.js**: set `"type": "module"` on the `background` key in
  `manifest.json`, then use plain `import`/`export` to split into files
  (e.g. `background/storage.js`, `background/messaging.js`). Chrome runs ES
  modules in service workers natively — no bundler needed.
- **content scripts**: list multiple files in `content_scripts[].js`
  (e.g. `["content/utils.js", "content/main.js"]`) — Chrome injects them in
  order into the same execution context, so no build step is required.
  Because they share global scope, namespace shared state under one object
  (e.g. `window.__ext = {...}`) instead of loose globals.
- **popup.html**: use `<script type="module" src="popup.js"></script>` and
  let `popup.js` `import` from other files — popup.html is a normal page,
  so ES modules work exactly as in any browser page.

Add an architecture note at the top of each entry-point file listing its
modules and the data flow between them:

```js
/**
 * ARCHITECTURE
 * background.js  — service worker, messaging hub, chrome.storage
 * content.js     — DOM injection, page interaction
 * popup.html/js  — UI for popup, sends messages to background
 *
 * DATA FLOW: popup → sendMessage → background → storage/tab action
 */
```

### Gotchas without a build step

Splitting files is safe, but these three issues are real and have nothing
to do with a bundler catching them for you:

- **Bad import paths fail silently at runtime, not at write time.** A typo
  in an `import` path or in `content_scripts[].js` isn't caught until you
  load the extension — often with no clear error, just a listener that
  never fires. Double-check paths, and reload+test after every split.
- **Content scripts can't use static `import`.** Files declared in
  `content_scripts` always run as classic scripts, never as modules — that's
  why they're split via the ordered `content_scripts[].js` array (shared
  global scope, namespaced under one object), not via ES modules. (Dynamic
  `import()` is technically possible there but requires listing the file in
  `web_accessible_resources`, exposing it to any web page — avoid it.)
- **The service worker's in-memory state doesn't survive a restart.** MV3
  unloads `background.js` after ~30s idle and reruns all its modules from
  scratch on the next event. A module like `background/storage.js` must
  persist anything that matters through `chrome.storage`, not module-level
  variables — splitting logic into its own file makes it easier to forget
  this and assume state "stuck around."

## System Permissions — Strictly Limited

⚠️ **This environment has very limited permissions.** Cannot:
- Install software or packages (no NPM, Homebrew, apt, choco, etc.)
- Run system commands that modify the computer
- Change system settings or registry
- Execute installers or setup files
- Access protected system directories

**This means all code must be:**
- **Self-contained** — no external dependencies, no build steps
- **Vanilla JavaScript** — runs directly in Chrome without compilation
- **Files only** — just manifest.json + .js files, drag into Chrome, done

If a task requires installation, compilation, or system changes, **stop and ask
the user** — it cannot be done in this environment.

## Setup Checklist

Before writing code:
1. Confirm extension purpose (what does it do?)
2. What tabs/pages does it affect?
3. Does it need popup UI? (Yes/No)
4. Does it need persistent storage? (Yes/No)

Write only the files that are actually needed.

**All code is self-contained — zero dependencies, zero build tools, zero setup.**
