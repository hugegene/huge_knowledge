# OAuth: HTTPS Website vs MCP Server

They look similar at the spec level but the actor model is completely different.

## The actors

| Role | Website OAuth | MCP OAuth |
|---|---|---|
| **User** | The human in front of a browser | The human in front of claude.ai |
| **Client** | The browser (rendering JS your site sent it) | The LLM client backend (claude.ai's server) |
| **Resource server** | Your website's API (often same origin as the UI) | Your MCP server (just JSON-RPC, no UI) |
| **Auth server** | Auth0 / Google / your own IdP | WorkOS / Stytch / etc. |
| **User-agent** | Browser handles redirects, cookies, JS | claude.ai infra handles HTTP, no cookies, no UI |

The single biggest difference: in the website flow, **the same browser** drives the whole dance, including the OAuth redirects AND the final API calls. In MCP, the human's browser does the redirect dance one time during connector setup — but **all subsequent API calls come from claude.ai's backend infrastructure**, not from your browser. The human is out of the loop after first sign-in.

## Flow A — OAuth for an HTTPS website

```
You          Browser                Your website                Your API              IdP (Google/etc)
 │             │                        │                          │                     │
 │ visit site  │                        │                          │                     │
 ├────────────►│                        │                          │                     │
 │             │ GET / (no session)     │                          │                     │
 │             ├───────────────────────►│                          │                     │
 │             │ 302 → /login           │                          │                     │
 │             │◄───────────────────────│                          │                     │
 │             │                        │                          │                     │
 │             │ GET /login             │                          │                     │
 │             ├───────────────────────►│                          │                     │
 │             │ HTML "sign in"         │                          │                     │
 │             │◄───────────────────────│                          │                     │
 │ click login │                        │                          │                     │
 ├────────────►│                        │                          │                     │
 │             │ 302 → IdP/authorize    │                          │                     │
 │             │  ?client_id=<PRE-REGISTERED>                      │                     │
 │             │  &redirect_uri=<KNOWN, allowlisted on IdP>        │                     │
 │             │  &state=...&PKCE                                  │                     │
 │             ├───────────────────────────────────────────────────────────────────────►│
 │             │                                                                        │
 │             │ IdP shows login UI (Google sign-in)                                    │
 │ enter creds │                                                                        │
 ├────────────►│                                                                        │
 │             │                                                                        │
 │             │ 302 → <your redirect_uri>?code=...                                     │
 │             │◄───────────────────────────────────────────────────────────────────────│
 │             │                                                                        │
 │             │ GET /oauth/callback?code=...                                           │
 │             ├───────────────────────►│                                               │
 │             │                        │ POST IdP/token                                │
 │             │                        │   { code, client_id, client_secret, PKCE }    │
 │             │                        ├──────────────────────────────────────────────►│
 │             │                        │ { access_token, refresh_token, id_token }     │
 │             │                        │◄──────────────────────────────────────────────│
 │             │                        │                                               │
 │             │                        │ Server SETS A SESSION COOKIE                  │
 │             │                        │   Server stores user identity in DB           │
 │             │                        │   Tokens kept SERVER-SIDE                     │
 │             │                        │                                               │
 │             │ 302 → /  +  Set-Cookie: session=...                                    │
 │             │◄───────────────────────│                                               │
 │             │                                                                        │
 │             │ ── you're now signed in ──                                             │
 │             │                                                                        │
 │             │ fetch('/api/whatever')                                                 │
 │             │   browser auto-attaches Cookie: session=...                            │
 │             ├──────────────────────────────────────────────►│                        │
 │             │ session looked up in DB                       │                        │
 │             │ → user identity → query data                  │                        │
 │             │◄──────────────────────────────────────────────│                        │
 │             │ JSON response                                                          │
```

Key properties:

- **Client is pre-registered** with the IdP. `client_id` + `client_secret` are baked into your server's config.
- **Redirect URI is allowlisted** on the IdP side (`https://yoursite.com/oauth/callback`).
- **Token lives server-side**, browser just gets an opaque session cookie.
- **State is mutable.** Server has a session table; logout deletes a row.
- **Subsequent API calls** ride on the cookie that the browser auto-attaches. Your server hits its session DB to identify the user.

## Flow B — OAuth for an MCP server

```
You         Browser            claude.ai backend           Your MCP server         IdP (WorkOS)
 │             │                     │                          │                     │
 │  in claude.ai chat UI, paste MCP URL into "Add connector"    │                     │
 ├────────────►│                     │                          │                     │
 │             ├────────────────────►│                          │                     │
 │             │                     │ POST /mcp (no token)     │                     │
 │             │                     ├─────────────────────────►│                     │
 │             │                     │ 401 + WWW-Authenticate:  │                     │
 │             │                     │   resource_metadata="..."│                     │
 │             │                     │◄─────────────────────────│                     │
 │             │                     │                          │                     │
 │             │                     │ GET /.well-known/oauth-protected-resource      │
 │             │                     ├─────────────────────────►│                     │
 │             │                     │ { authorization_servers: ["<WorkOS issuer>"] } │
 │             │                     │◄─────────────────────────│                     │
 │             │                     │                                                │
 │             │                     │ GET <issuer>/.well-known/oauth-authorization-server
 │             │                     ├──────────────────────────────────────────────►│
 │             │                     │ { authorization_endpoint, token_endpoint,     │
 │             │                     │   registration_endpoint, jwks_uri, ... }      │
 │             │                     │◄──────────────────────────────────────────────│
 │             │                     │                                               │
 │             │                     │ POST <registration_endpoint>  ← DYNAMIC       │
 │             │                     │   { redirect_uris: ["https://claude.ai/.../cb"],
 │             │                     │     client_name: "Claude" }                   │
 │             │                     ├──────────────────────────────────────────────►│
 │             │                     │ 201 { client_id: "client_01KTEFE..." }        │
 │             │                     │◄──────────────────────────────────────────────│
 │             │                     │                                               │
 │             │ Anthropic redirects YOUR browser to WorkOS authorize endpoint       │
 │             │◄────────────────────│                                               │
 │             │ 302 → <issuer>/oauth2/authorize?client_id=<JUST_REGISTERED>...      │
 │             ├──────────────────────────────────────────────────────────────────────►
 │             │                                                                     │
 │             │ WorkOS shows Google sign-in                                         │
 │ enter creds │                                                                     │
 ├────────────►│                                                                     │
 │             │                                                                     │
 │             │ 302 → https://claude.ai/.../callback?code=...                       │
 │             │◄────────────────────────────────────────────────────────────────────│
 │             │                                                                     │
 │             │ GET claude.ai callback   │                                          │
 │             ├────────────────────►│                                               │
 │             │                     │ POST <token_endpoint>                         │
 │             │                     │   { code, client_id, PKCE verifier }          │
 │             │                     ├──────────────────────────────────────────────►│
 │             │                     │ { access_token: "<JWT>",                      │
 │             │                     │   refresh_token, expires_in }                 │
 │             │                     │◄──────────────────────────────────────────────│
 │             │                     │                                               │
 │             │                     │ Anthropic STORES the JWT, scoped to this      │
 │             │                     │ connector for this user                       │
 │             │                     │ Your MCP server stores NOTHING                │
 │             │                     │                                               │
 │             │ ── connector is now active ──                                       │
 │             │                                                                     │
 │  later, in a chat, you ask: "use trademoomoo to ..."                              │
 │             │                     │                                               │
 │             │                     │ POST /mcp                                     │
 │             │                     │   Authorization: Bearer <JWT>                 │
 │             │                     │   { jsonrpc, method: "tools/call", ... }      │
 │             │                     ├─────────────────────────►│                    │
 │             │                     │                          │                    │
 │             │                     │ Server validates JWT FRESH on every call:     │
 │             │                     │   - signature against WorkOS JWKS (cached)    │
 │             │                     │   - iss matches MCP_OAUTH_ISSUER              │
 │             │                     │   - exp not in past                           │
 │             │                     │   - aud (skipped in your config)              │
 │             │                     │ NO session lookup. NO DB. Just crypto math.   │
 │             │                     │                                               │
 │             │                     │                          → tool executes      │
 │             │                     │ JSON-RPC result          │                    │
 │             │                     │◄─────────────────────────│                    │
```

Key properties:

- **Client is dynamically registered** at runtime via DCR. claude.ai didn't exist as a pre-known client to WorkOS until the moment you added the connector.
- **Redirect URI is dynamic** too — claude.ai tells WorkOS its callback URL during DCR, not via a dashboard config.
- **Token lives in claude.ai's storage**, scoped to (this user, this connector). Your MCP server holds zero session state.
- **State is essentially immutable on your side.** You can't "log someone out" from your MCP server because there's no session to invalidate. You can revoke at the IdP (kill the user in WorkOS), which makes future tokens fail validation.
- **Every API call re-validates the JWT** from scratch using cached public keys. No DB hit per request.

## The contrasts that actually matter

| Dimension | Website OAuth | MCP OAuth |
|---|---|---|
| **Who holds the token** | Your server (session DB), browser gets a cookie | claude.ai's backend; never touches your server |
| **Client registration** | Pre-registered, static `client_id`/`client_secret` | Dynamic (DCR), happens at runtime |
| **Discovery** | None — the URL the user typed *is* the entry point | Required — `WWW-Authenticate` header → resource metadata → auth server metadata |
| **Session model** | Stateful — DB-backed session, cookie auth on each request | Stateless — JWT validated cryptographically per call |
| **Redirect URI** | Hardcoded in IdP dashboard | Sent during DCR, varies per client |
| **Logout / revoke** | Delete the session row, clear cookie | Can't do it server-side; revoke at IdP (kills future tokens) |
| **Token in transit** | Browser cookie (auto) or `Authorization` header (manual JS) | Always `Authorization: Bearer <JWT>` in JSON-RPC POST |
| **CSRF concerns** | Major — cookies are ambient, need anti-CSRF tokens | None — Bearer tokens aren't ambient credentials |
| **User-perceived sign-in trigger** | Click "log in" button on your site | Add the connector (one-time), then never again unless token expires |
| **PKCE** | Optional/recommended for SPAs | Mandatory per MCP spec |
| **Resource binding (RFC 8707)** | Rarely used — token is for "the site" generally | Required by spec — token bound to the specific MCP server URL via `aud` claim |
| **Why your server is simpler** | Has to manage sessions, logout, CSRF, etc. | Just validates JWT signature on each call — nothing else |

## The mental model shift

**Website OAuth:** "User logs into my service. I now know who they are and remember them between requests."

**MCP OAuth:** "User signed into my service ONCE, weeks ago. Every incoming request is a fresh stranger holding a JWT that claims to be them. I check the cryptographic proof every time. I don't care who they are between requests."

This is why an MCP server has zero session code, zero auth middleware beyond the JWT verifier, and works fine on a Cloud Run service that scales from 0 instances. The "stateful session" half of typical auth code simply doesn't exist for an MCP server — the IdP and the JWT do all the work.

It's also why the failure modes are different:

- Website OAuth: lost session, expired cookie, CSRF, "remember me", "log out everywhere"
- MCP OAuth: expired token (refresh automatically), JWKS cache stale (auto-fetches), audience mismatch, DCR not enabled
