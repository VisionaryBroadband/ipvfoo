# Security Audit — IPvFoo Extension

Date: 2026-09-16

**Verdict: Clean.** No RCE vectors, no phone-home telemetry, no third-party dependency exposure.

## Remote code execution

- No `eval()`, `new Function()`, `innerHTML`/`outerHTML`, or `document.write` anywhere in the codebase.
- All DOM construction uses safe `createElement`/`textContent`/`appendChild`.
- No `content_scripts` are declared in any manifest — the extension never injects scripts into web pages.
- `importScripts()` is used only to load its own local files (`iputil.js`, `common.js`), never a remote URL.

## Phone-home / data exfiltration

- No `fetch()` or `XMLHttpRequest` calls exist in the entire codebase.
- IP/domain data collected via `webRequest`/`webNavigation` is kept only in `chrome.storage.session` (RAM-backed, local) and `chrome.storage.sync` (Chrome's own account-sync, used only for your lookup-provider preference and NAT64 prefixes — never for browsing data).
- The only outbound network activity is user-initiated: right-clicking a selected domain/IP opens a new tab to a lookup provider (bgp.he.net, info.addr.tools, or ipinfo.io — user-selectable, including a custom URL). This requires an explicit click; nothing fires automatically.
- The bundled `misc/privacy_policy.txt` makes this same claim ("all information is kept in RAM... never transmitted over the network"), and it matches what the code actually does.

## Supply chain / third-party dependencies

- No `package.json`, lockfile, `node_modules`, or bundled/minified third-party JS anywhere in the repo.
- Build process (`Makefile`) just zips the local `src/` directory into `.xpi`/`.zip` — no dependency fetching or npm install step.
- No externally-hosted `<script src="https://...">` tags in any HTML file; all scripts are local files shipped in the extension.

## Permissions (expected, not a red flag)

`<all_urls>` host permission plus `webRequest`/`webNavigation` are required for its stated purpose (reading IP/protocol info for every connection on every page). `storage`, `contextMenus`, and `offscreen` (Chrome-only, for dark-mode detection) are minimal and used exactly as declared. No `contentSettings`, `cookies`, `history`, `tabs` (beyond query), `scripting`, or `management` — permissions are notably narrow for what the extension does.

## Minor observation (not a vulnerability)

`options.js` builds a lookup URL from user input but validates it against a strict `^https:\/\/[^/$]+\/.*\$` regex before use (`common.js:102`), so a malicious custom provider string can't become a `javascript:` URI — it's properly constrained.
