# Mobbin Docs (offline mirror)

Mirrored from `https://docs.mobbin.com` on 2026-08-06. Source of truth is always the live
site — re-run this capture if something here seems stale. Full site index used to build
this: `https://docs.mobbin.com/llms.txt`.

Mobbin is the industry-standard UI/UX design reference library (mobile apps, web apps,
websites). This mirror only covers the **MCP** and **API** docs — how to query Mobbin
programmatically — not the Mobbin web app itself.

---

## 1. Overview

There are two ways to access Mobbin programmatically:

|                    | MCP                             | API                          |
| ------------------ | -------------------------------- | ----------------------------- |
| **Best for**       | AI chat apps & coding agents     | Custom integrations           |
| **Offered as**      | Remote MCP server                | REST API                      |
| **Authorization**  | OAuth (automatic)                | API key (Bearer token)        |
| **Availability**   | Pro, Team & Enterprise plans      | Team & Enterprise plans       |

This project (`mobbin-ui-ref` skill) uses the **MCP** tools (`mcp__mobbin__search_screens`,
`mcp__mobbin__search_flows`, `mcp__mobbin__search_sections`) — already connected in this
Claude Code session, no setup needed here. The API section below is documented for
completeness (e.g. if building a separate integration outside Claude Code).

## 2. Rate Limits

| Surface | Scope         | Limit                        |
| ------- | ------------- | ----------------------------- |
| MCP     | Per user      | 60 requests / 60 seconds      |
| API     | Per workspace | 60 requests / 60 seconds      |

On exceeding the limit: HTTP `429 Too Many Requests` with a `Retry-After` header (seconds
to wait). Best practice: respect `Retry-After`; if retries keep failing, back off
exponentially with jitter.

---

## 3. MCP

### 3.1 Introduction

- Available on **Pro, Team, and Enterprise** plans.
- Lets AI tools/coding agents search real-world UI designs via natural language; returns
  screen images inline in tool responses.
- Transport: [Streamable HTTP](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports#streamable-http).
- Auth: OAuth — most MCP clients handle the browser-based authorization automatically, no
  manual API key/token setup.
- Server URL: `https://api.mobbin.com/mcp`

### 3.2 Features / Tools

| Tool               | What it does                                                       |
| ------------------- | -------------------------------------------------------------------- |
| `search_screens`    | Searches UI screens.                                                 |
| `search_flows`      | Searches multi-step user flows (e.g. onboarding, checkout).          |
| `search_sections`   | Searches website sections (e.g. pricing pages, footers).             |

**Interactive gallery**: in supported clients (ChatGPT, Claude Desktop/Web, Codex App, or
any client supporting the [MCP Apps spec](https://modelcontextprotocol.io/docs/extensions/apps)),
results render as an inline interactive gallery. Known issue: gallery may not render in
Claude Cowork / Claude Code within the Claude desktop app — the agent can still see and
use the results even if the gallery itself doesn't load.

### 3.3 Supported clients & setup

All clients authorize via OAuth — sign in with your Mobbin account when the browser
window opens. Common "try it" prompt across all clients:

```text
Search Mobbin for onboarding screens from banking apps
```

| Client | Setup summary |
| --- | --- |
| **Claude Desktop and Web** | Connector in Claude's directory: Customize → Connectors → Add → Browse connectors → **Mobbin** → Connect → sign in. Same steps also cover Claude Cowork and Claude Code on desktop. Team/Enterprise: admin must add Mobbin org-wide first (Organization settings → Connectors → Add → All available → Mobbin → "Add to your team"); non-admins may need to request access. |
| **Claude Code CLI** | Recommended: set up via Claude Desktop/Web first — CLI shares connectors automatically. CLI-only (not recommended): `claude mcp add mobbin --scope user --transport http https://api.mobbin.com/mcp`, then `claude /mcp` → select **mobbin** → **Authenticate** → sign in in the browser window that opens. |
| **ChatGPT** | Open the [Mobbin app page](https://chatgpt.com/apps/mobbin/asdk_app_69fdb9081018819193707354f21b366e) → Connect → "Sign in with Mobbin" → authorize. |
| **Codex App** | Shares credentials with ChatGPT — set up ChatGPT first, Codex app gets access automatically. |
| **Codex CLI** | `codex mcp add mobbin --url https://api.mobbin.com/mcp`, then `codex mcp login mobbin` → browser opens → sign in. |
| **Cursor** | Recommended: click the [pre-filled deeplink](cursor://anysphere.cursor-deeplink/mcp/install?name=Mobbin&config=eyJ0eXBlIjoiaHR0cCIsInVybCI6Imh0dHBzOi8vYXBpLm1vYmJpbi5jb20vbWNwIn0=) → Install → find Mobbin → Connect → sign in. Manual: Settings → Tools & MCPs → Installed MCP Servers → New MCP server, paste into `mcp.json`: `{"mcpServers": {"Mobbin": {"type": "http", "url": "https://api.mobbin.com/mcp", "headers": {}}}}` → Connect → sign in. |
| **VS Code** | Via [GitHub MCP Registry](https://github.com/mcp/com.mobbin/mobbin) → Install MCP server → Install in VS Code → Install → Allow → sign in. |
| **v0** | Chat input **+** → MCPs → Add MCP → name "Mobbin", URL `https://api.mobbin.com/mcp` → auth method OAuth → Authorize → sign in. |
| **Replit** | Settings → Integrations → search "Mobbin" → Sign in → "Test & save" → "Authorize server" → sign in → redirected back to Replit. |
| **Lovable** | Settings → Connectors → Personal connectors → New MCP server → name "Mobbin", URL `https://api.mobbin.com/mcp` → sign in on first use. |
| **Magic Patterns** | Settings → Connectors → find Mobbin → Connect → "Connect with OAuth" → sign in → confirmation → Close. |
| **Manus** | Open the [Mobbin connector page](https://manus.im/app/plugins#connector_73ebc40f-2395-4c22-b28d-2421dcbdc9de) → Connect → sign in. |
| **Devin** | Devin MCP Marketplace → Settings → Connections → MCP servers → Mobbin → Access: Personal → "Install and enable" → confirm dialog → sign in. |
| **Antigravity 2.0 / Antigravity IDE** | Edit `~/.gemini/config/mcp_config.json`, add under `mcpServers`: `{"Mobbin": {"serverUrl": "https://api.mobbin.com/mcp"}}`. Then in-app: Settings → Customizations → Installed MCP Servers → Refresh → Authenticate next to Mobbin → sign in in browser → copy the auth code from the callback page → paste into the app → Submit → Refresh if needed. |
| **Other clients** | Any client supporting Streamable HTTP + OAuth: add a new HTTP MCP server pointing at `https://api.mobbin.com/mcp`; sign in on first use. |

### 3.4 Build your own integration

For developers building an app that calls Mobbin's MCP server on behalf of their own
users (each user authorizes with their own Mobbin account).

- Users need a Mobbin **Pro**, **Team**, or **Enterprise** plan. No separate integrator fee.
- Uses [Dynamic Client Registration (RFC 7591)](https://datatracker.ietf.org/doc/html/rfc7591)
  — no manual `client_id` provisioning or redirect URI whitelisting needed.

Flow:
1. Fetch protected resource metadata: `https://api.mobbin.com/.well-known/oauth-protected-resource/mcp`
2. Fetch the authorization server's `.well-known/oauth-authorization-server` for the full
   OAuth endpoint set (authorization, token, registration, JWKS, etc.).
3. Register your app via the DCR endpoint to obtain a `client_id`.
4. Redirect the user to the authorization endpoint with PKCE (`S256`) and the `openid` scope.
5. Exchange the authorization code for an access token + refresh token.
6. Call the MCP server at `https://api.mobbin.com/mcp` with a `Bearer` token.
7. Use the refresh token to get new access tokens as needed.

Full protocol: [MCP authorization spec](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization).
Recommended: use an MCP SDK that automates the OAuth flow —
[TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk),
[Python SDK](https://github.com/modelcontextprotocol/python-sdk).

### 3.5 Revoke access

1. Go to [Account Settings → MCP](https://mobbin.com/settings/mcp).
2. Under **Connected**, find the client.
3. Click the **...** menu → **Revoke access**.

Access is revoked immediately; reconnect any time by re-authorizing through the MCP client.

---

## 4. API (REST)

Available on **Team and Enterprise** plans only (not Pro).

### 4.1 Quick start

1. Get an API key: [Settings → API Keys](https://mobbin.com/settings/team/api-keys) in
   your Mobbin workspace → **Create API Key**. Key is scoped to the workspace — keep it
   secret, don't commit to source control.
2. Base URL: `https://api.mobbin.com`
3. Auth header: `Authorization: Bearer YOUR_API_KEY`

Example request:

```bash
curl -X POST https://api.mobbin.com/v1/screens/search \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "login screen with biometric authentication",
    "platform": "ios",
    "limit": 5
  }'
```

Response includes matching screens with image URLs, app names, and links back to Mobbin.

### 4.2 API Reference — `POST /v1/screens/search`

Search Mobbin for screens using natural language. Returns relevant screens with image
URLs, app names, and links to Mobbin.

**Request body** (`application/json`):

| Field | Type | Required | Default | Notes |
| --- | --- | --- | --- | --- |
| `query` | string (1–500 chars) | Yes | — | Describe **one screen** in plain language — the UI elements you'd see and how they relate. Be specific. Good: `"login screen with biometric authentication"`, `"checkout page with promo code field and Apple Pay button"`. Avoid: combining multiple screens/intents (search separately), negations (`"without ads"`), vague style words (`"modern"`, `"clean"`), disconnected keyword lists. Name a specific app to filter to it (e.g. `"Spotify now-playing screen"`). Don't put platform in the query — use the `platform` field. |
| `platform` | enum: `ios`, `web` | Yes | — | Platform to search. |
| `mode` | enum: `deep`, `standard`, `fast` | No | `deep` | `deep`: AI-powered pipeline that interprets intent and scores each candidate for relevance, keeping only strong matches — best for nuanced queries. `standard`: low-latency results. `fast`: deprecated alias for `standard`, will be removed — use `standard`. |
| `limit` | integer (1–100) | No | `20` | Max screens to return. |
| `image_quality` | enum: `optimized`, `high` | No | `optimized` | `optimized`: sized for AI agents / programmatic inspection. `high`: larger images for detailed visual inspection. |
| `exclude_screen_ids` | array of UUID strings (max 100) | No | `[]` | Screen IDs to exclude from results. |

**Response `200`** — `{ "screens": Screen[] }`

**`Screen` object**:

| Field | Type | Notes |
| --- | --- | --- |
| `id` | UUID | Unique screen identifier. |
| `image_url` | URI | **Deprecated** — use `image.url` instead. |
| `image` | `ScreenImage` | See below. |
| `mobbin_url` | URI | Link to view this screen on Mobbin. |
| `app_name` | string | Name of the app the screen belongs to. |
| `platform` | enum: `ios`, `web` | Platform of the app. |

**`ScreenImage` object**:

| Field | Type | Notes |
| --- | --- | --- |
| `url` | URI | Image URL for the screen. |
| `width` | integer | Exact delivered width in px. Present together with `height`; both may be absent if source dimensions are unknown. |
| `height` | integer | Exact delivered height in px. Same pairing rule as `width`. |
| `url_expires_at` | RFC 3339 datetime | When the image URL expires. |

**Error responses**: `400` invalid params, `401` missing/invalid API key, `403` Team/Enterprise
plan required, `404` not found, `429` rate limit exceeded, `500` internal error. All error
bodies: `{ "error": { "code": string, "message": string } }`.

Full machine-readable spec saved alongside this file: [`openapi.json`](openapi.json).

---

## Source pages mirrored

- `https://docs.mobbin.com/overview`
- `https://docs.mobbin.com/rate-limits`
- `https://docs.mobbin.com/mcp/introduction`
- `https://docs.mobbin.com/mcp/features`
- `https://docs.mobbin.com/mcp/build-an-integration`
- `https://docs.mobbin.com/mcp/disconnect`
- `https://docs.mobbin.com/mcp/clients/overview` (+ all 15 individual client pages)
- `https://docs.mobbin.com/api/quickstart`
- `https://docs.mobbin.com/api-reference/screens/search-screens-with-natural-language`
- `https://docs.mobbin.com/openapi.json`
