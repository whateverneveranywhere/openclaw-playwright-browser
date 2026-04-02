# openclaw-playwright-browser

Extracted browser control and Playwright automation code from [openclaw/openclaw](https://github.com/openclaw/openclaw).

This fork keeps **only** the browser automation surface -- the Playwright session management, CDP integration, Chrome MCP bridge, browser tool actions, and the diff screenshotter -- stripped of all other channels, providers, and application code.

## What's here

```
extensions/
  browser/          # Main browser control plugin
    index.ts        # Plugin entry point (registers tool, CLI, gateway method, service)
    src/
      browser/
        pw-session.ts               # Playwright session lifecycle (Browser/Context/Page)
        pw-tools-core.ts            # Barrel: all browser tool operations
        pw-tools-core.interactions.ts  # click, type, fill, drag, select, hover, press, evaluate
        pw-tools-core.snapshot.ts      # ARIA/AI snapshots, screenshots, viewport resize
        pw-tools-core.state.ts         # offline, headers, geolocation, device emulation
        pw-tools-core.downloads.ts     # download interception and save
        pw-tools-core.storage.ts       # cookies and localStorage
        pw-tools-core.responses.ts     # network request/response inspection
        pw-tools-core.activity.ts      # console messages and page errors
        pw-tools-core.trace.ts         # HAR/trace recording
        pw-ai.ts                       # Playwright _snapshotForAI() integration
        pw-role-snapshot.ts            # Role-based snapshots with ref tracking
        chrome.ts                      # Chrome executable discovery and launch
        chrome-mcp.ts                  # Chrome DevTools MCP server bridge
        cdp.ts / cdp.helpers.ts        # Raw CDP communication
        navigation-guard.ts            # SSRF protection for navigation
        server.ts / server-context.ts  # HTTP control server
        client.ts / client-actions.ts  # Client API for browser operations
        profiles.ts                    # Multi-profile management
        routes/                        # HTTP route handlers (snapshot, act, tabs, storage)
      cli/                   # CLI commands (browser start/stop/open/snapshot/etc.)
      config/                # Browser config resolution
      gateway/               # Gateway integration
      infra/                 # Network, filesystem, port utilities
  diffs/             # Diff viewer plugin (uses Playwright for screenshot rendering)
    src/
      browser.ts             # PlaywrightDiffScreenshotter (chromium.launch for rendering)

Dockerfile.sandbox-browser   # Headless Chromium container for sandboxed browser
```

## Key patterns

### Session management (`pw-session.ts`)

Connects to Chrome via CDP WebSocket, creates Playwright `Browser` objects, manages page lifecycle. Tracks console messages, errors, and network requests per page. Caches ARIA role refs for stable element identification across snapshots.

### Tool operations (`pw-tools-core.*.ts`)

Each file exports functions that take a CDP URL + target ID and perform browser operations:

- **interactions** -- `clickViaPlaywright()`, `typeViaPlaywright()`, `fillFormViaPlaywright()`, `dragViaPlaywright()`, etc. Supports batch operations with abort.
- **snapshots** -- ARIA tree via CDP accessibility API, AI snapshots via `_snapshotForAI()`, role-based snapshots with ref maps, screenshots.
- **state** -- device emulation, geolocation, offline mode, custom headers, locale/timezone.
- **storage** -- cookie get/set/clear, localStorage get/set/clear.
- **network** -- response body retrieval, request inspection.

### Chrome MCP bridge (`chrome-mcp.ts`)

Launches `chrome-devtools-mcp` as a stdio MCP server to connect to a user's real Chrome session (the "user" profile), as opposed to the isolated "openclaw" profile which uses direct Playwright control.

### Navigation guard (`navigation-guard.ts`)

SSRF protection layer that validates URLs before navigation and checks redirect chains after navigation completes.

### HTTP control server (`server.ts`, `routes/`)

Express-based local server exposing browser operations over HTTP with bearer token auth, CSRF protection, and CORS.

## Dependencies

- `playwright-core` 1.58.2 -- Core Playwright API (no bundled browsers)
- `@modelcontextprotocol/sdk` -- MCP client for Chrome DevTools MCP

## License

[MIT](LICENSE) (same as upstream)
