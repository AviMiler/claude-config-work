# Chrome Extension Reference (Manifest V3, Vanilla JavaScript)

## Minimal manifest.json

```json
{
  "manifest_version": 3,
  "name": "My Extension",
  "version": "1.0.0",
  "description": "What this extension does",
  "permissions": ["storage", "activeTab"],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "run_at": "document_start"
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_title": "My Extension"
  },
  "icons": {
    "16": "icons/16.png",
    "48": "icons/48.png",
    "128": "icons/128.png"
  }
}
```

## Service Worker (background.js)

Service workers run when needed (no DOM access):

```javascript
// Listen for messages from content script or popup
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === 'GET_DATA') {
    chrome.storage.local.get('myData', (result) => {
      sendResponse(result.myData);
    });
    return true; // Required for async sendResponse
  }
});

// Listen for tab updates
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete') {
    chrome.tabs.sendMessage(tabId, { type: 'PAGE_LOADED' });
  }
});
```

## Content Script (content.js)

Runs in page context (has DOM access, isolated from page JS):

```javascript
// Guard against double-injection
if (window.__myExtLoaded) return;
window.__myExtLoaded = true;

// Listen for messages from background
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === 'PAGE_LOADED') {
    console.log('Background says page loaded');
  }
});

// Send message to background
chrome.runtime.sendMessage({ type: 'GET_DATA' }, (response) => {
  console.log('Got data:', response);
});

// Communicate with page JS via window.postMessage
window.postMessage({ type: 'FROM_EXTENSION', data: 'hello' }, '*');

// Listen from page JS
window.addEventListener('message', (event) => {
  if (event.source !== window) return;
  if (event.data.type === 'FROM_PAGE') {
    console.log('Page says:', event.data.data);
  }
});
```

## Popup (popup.html + popup.js)

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { width: 300px; font-family: sans-serif; }
    button { padding: 10px; width: 100%; cursor: pointer; }
  </style>
</head>
<body>
  <h2>My Extension</h2>
  <button id="btnClick">Click Me</button>
  <div id="result"></div>
  <script src="popup.js"></script>
</body>
</html>
```

```javascript
// popup.js
document.getElementById('btnClick').addEventListener('click', () => {
  chrome.runtime.sendMessage({ type: 'GET_DATA' }, (response) => {
    document.getElementById('result').textContent = JSON.stringify(response);
  });
});
```

## Storage API

Always use `chrome.storage`, never `localStorage`:

```javascript
// Save
chrome.storage.local.set({ myKey: 'myValue' });

// Read
chrome.storage.local.get(['myKey'], (result) => {
  console.log('Stored value:', result.myKey);
});

// Remove
chrome.storage.local.remove(['myKey']);

// Clear all
chrome.storage.local.clear();
```

## Common Patterns

### Message Flow: popup → background → content

```javascript
// popup.js
chrome.runtime.sendMessage({ 
  type: 'INJECT_SCRIPT',
  script: 'alert("Hello!")' 
}, response => console.log(response));

// background.js
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === 'INJECT_SCRIPT') {
    chrome.tabs.query({ active: true, currentWindow: true }, tabs => {
      chrome.tabs.sendMessage(tabs[0].id, msg, sendResponse);
    });
    return true;
  }
});

// content.js
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === 'INJECT_SCRIPT') {
    eval(msg.script); // Or use Function() for safety
    sendResponse({ success: true });
  }
});
```

### Query Active Tab

```javascript
chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
  const activeTab = tabs[0];
  console.log('Active tab URL:', activeTab.url);
});
```

### Permission Checklist

Only request what you need:

- `storage` — Use chrome.storage.local/sync
- `activeTab` — Access the current tab (requires user gesture)
- `tabs` — Query/manage all tabs
- `scripting` — Inject scripts dynamically
- `alarms` — Schedule tasks in background
- `contextMenus` — Add right-click menu items

Host permissions (`host_permissions`) example:
```json
"host_permissions": ["https://example.com/*", "https://example.org/*"]
```

## Common Gotchas

1. **Service worker sleeps** — never store data in variables. Always use storage.
2. **Content script isolation** — cannot access page's `window` variables. Use `window.postMessage`.
3. **Async operations** — return `true` from `onMessage` listener if calling `sendResponse` async.
4. **No DOM in background** — use `chrome.tabs.executeScript` or `chrome.tabs.sendMessage` instead.
5. **CSP restrictions** — inline `<script>` tags don't work; load from files only.
