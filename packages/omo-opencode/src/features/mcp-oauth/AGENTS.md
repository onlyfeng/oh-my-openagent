# src/features/mcp-oauth/ — OAuth 2.0 + PKCE + DCR for MCP Servers

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

Full OAuth 2.0 authorization flow for MCP servers requiring authentication. Implements PKCE (RFC 7636) and Dynamic Client Registration (DCR, RFC 7591). Used by `bunx oh-my-opencode mcp oauth login` (`src/cli/mcp-oauth/`) and the skill MCP manager.

Adapter shim: all 10 non-test `.ts` files are one-line re-exports of `@oh-my-opencode/mcp-client-core/mcp-oauth/*` (no local barrel). Edit `packages/mcp-client-core/src/mcp-oauth/`; the 7 co-located tests here pin adapter behavior.

## AUTHORIZATION FLOW

```
1. discovery.ts → fetch /.well-known/oauth-authorization-server
2. dcr.ts → Dynamic Client Registration (if server supports it)
3. oauth-authorization-flow.ts → generate PKCE verifier/challenge
4. callback-server.ts → local HTTP server on random port for redirect
5. Open browser → authorization URL
6. callback-server.ts → receive code + state
7. provider.ts → exchange code for token (with PKCE verifier)
8. storage.ts -> persist token under <opencode config dir>/mcp-oauth/
9. step-up.ts → handle step-up auth if initial token insufficient
```

## KEY FILES

| File | Purpose |
|------|---------|
| `oauth-authorization-flow.ts` | PKCE helpers: `generateCodeVerifier()`, `generateCodeChallenge()`, `buildAuthorizationUrl()` |
| `callback-server.ts` | Local HTTP redirect server — listens for OAuth callback |
| `provider.ts` | `OAuthProvider` — token exchange, refresh, revoke |
| `discovery.ts` | Fetch + parse OAuth server metadata from well-known endpoint |
| `dcr.ts` | Dynamic Client Registration — register this app with OAuth server |
| `step-up.ts` | Handle step-up authentication challenges |
| `storage.ts` | Persist tokens to `<opencode config dir>/mcp-oauth/{server-hash}.json`; reads legacy `mcp-oauth.json` |
| `refresh-mutex.ts` | Serializes concurrent token refreshes |
| `resource-indicator.ts` | RFC 8707 resource indicator handling (hash key includes resource) |
| `schema.ts` | Token / metadata schemas |

## PKCE IMPLEMENTATION

- Code verifier: 32 random bytes → base64url (no padding)
- Code challenge: SHA-256(verifier) → base64url
- Method: `S256`

## TOKEN STORAGE

Location: `getOpenCodeCliConfigDir()/mcp-oauth/`, one JSON file per MCP server (keyed by server host + resource hash).
Fields: `access_token`, `refresh_token`, `expires_at`, `client_id`.

## CLI COMMANDS

```bash
bunx oh-my-opencode mcp oauth login <server-name>    # Full PKCE flow
bunx oh-my-opencode mcp oauth logout <server-name>   # Revoke + delete token
bunx oh-my-opencode mcp oauth status [server-name]   # List stored tokens
```
