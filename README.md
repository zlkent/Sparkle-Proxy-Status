# Sparkle Proxy Status

Chrome MV3 extension for Sparkle. It shows whether the current tab is proxied, which rule matched, and the proxy chain returned by Sparkle's local read-only Extension API.

## What It Does

- Checks the current tab URL against Sparkle's local Extension API
- Shows proxied/direct status in the popup
- Displays matched rule, chain, and last update time
- Supports background prefetch and badge status
- Uses a local Bearer token for read-only authorization

## Repository Layout

This repository is intentionally minimal:

- `extension/`: Chrome extension source

The following directories are local-only and are not meant to be published from this repository:

- `docs/`
- `scripts/`
- `upstream-sparkle/`

## Load the Extension Locally

1. Open `chrome://extensions`
2. Turn on `Developer mode`
3. Click `Load unpacked`
4. Select the `extension/` directory in this repository

## Configure Sparkle

In Sparkle:

1. Open `设置 -> 更多设置 -> 浏览器扩展 API`
2. Enable the Extension API
3. Confirm the port, usually `14123`
4. Copy the token
5. Set allowed origin to `chrome-extension://<your-extension-id>` if you want origin allowlisting

## Configure the Extension

Open the extension options page and fill in:

- `Extension API Base URL`: `http://127.0.0.1:14123`
- `Bearer Token`: paste the token from Sparkle without adding `Bearer `

Then click `测试连接`.

## Package for Chrome Web Store

Chrome Web Store expects the zip root to contain `manifest.json` directly. Do not zip the whole repository root.

### Option 1: Finder / Explorer

Zip the contents inside `extension/`, not the `extension/` folder itself.

The final zip should contain files like:

- `manifest.json`
- `popup.html`
- `popup.js`
- `options.html`
- `options.js`
- `service_worker.js`

### Option 2: Command Line

From the repository root:

```bash
cd extension
zip -r ../extension.zip .
```

Then upload `extension.zip` to Chrome Web Store.

## Submit to Chrome Web Store

1. Go to the Chrome Web Store Developer Dashboard
2. Create a new item or open the existing item
3. Upload the packaged zip
4. Fill in the store listing, screenshots, description, and privacy information
5. Submit for review

## Release Checklist

Before uploading:

1. Confirm Sparkle's Extension API is enabled
2. Confirm the token works
3. Confirm popup and options page both load correctly
4. Confirm the current tab can show proxied/direct status
5. Bump `extension/manifest.json` version before each store upload

## Development Notes

- Host permission is limited to `http://127.0.0.1/*`
- The extension only reads local Sparkle state
- Authorization is sent as `Authorization: Bearer <token>`
