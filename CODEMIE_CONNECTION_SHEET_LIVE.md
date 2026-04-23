# CodeMie MCP Connection Sheet – Live Setup

**Status:** ✅ Ready to connect  
**Created:** 2026-04-23  
**Tunnel Type:** Cloudflare Quick Tunnel (ephemeral, no uptime guarantee)

## Connection Parameters

| Parameter | Value |
|-----------|-------|
| **MCP Endpoint URL** | `https://calendars-kills-booking-skin.trycloudflare.com/mcp` |
| **Authentication** | None (Quick Tunnel has no auth) |
| **Protocol** | MCP streamable-http (SSE-based) |
| **Session Header** | `Mcp-Session-Id` (must be preserved by client) |

## CodeMie MCP Configuration

Paste this JSON into CodeMie's MCP configuration:

```json
{
  "command": "npx",
  "args": [
    "-y",
    "mcp-remote",
    "https://calendars-kills-booking-skin.trycloudflare.com/mcp"
  ],
  "env": {}
}
```

**Alternative:** If CodeMie has a dedicated URL field, use just:
```
https://calendars-kills-booking-skin.trycloudflare.com/mcp
```

## Preflight Checklist

- [ ] Cloudflare tunnel running on laptop: `cloudflared tunnel --url http://127.0.0.1:8000`
- [ ] MCP server running locally: `uv run -m linkedin_mcp_server --transport streamable-http --host 127.0.0.1 --port 8000 --path /mcp`
- [ ] LinkedIn session logged in (run `uv run -m linkedin_mcp_server --login` once)
- [ ] Public URL responds: Test with curl below

## Validation Command

Run this from any terminal to verify the endpoint is live:

```bash
curl -s -X POST https://calendars-kills-booking-skin.trycloudflare.com/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'
```

**Expected output:** MCP JSON response with `"result"` and `"serverInfo"` fields.

## Important Notes

1. **Keep both processes running:**
   - Local MCP server (Terminal 1)
   - Cloudflare tunnel (Terminal 2)

2. **URL is ephemeral:** If you restart the tunnel or laptop, the URL changes. Update CodeMie with the new URL.

3. **No authentication:** Quick Tunnel has no built-in auth. Keep the URL private.

4. **Session continuity:** CodeMie MUST preserve the `Mcp-Session-Id` header in responses for stateful operations.

5. **Authentication model:** This MCP server uses browser-session cookies, not OAuth tokens. You must log in once locally with `--login` flag.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `curl` returns HTML/login page | MCP server not running locally. Start it: `uv run -m linkedin_mcp_server --transport streamable-http --host 127.0.0.1 --port 8000 --path /mcp` |
| `curl` times out | Cloudflare tunnel not running. Start it: `cloudflared tunnel --url http://127.0.0.1:8000` |
| CodeMie says "not initialized" after restart | URL changed on tunnel restart. Copy new URL from tunnel output and update CodeMie. |
| Tools fail but initialize works | Session header not preserved. Verify CodeMie sends `Mcp-Session-Id` in all requests. |
| "Not Acceptable" error | Missing Accept header. CodeMie must send: `Accept: application/json, text/event-stream` |

## Next Steps

1. Copy the MCP configuration JSON above into CodeMie
2. Run the validation command to confirm connectivity
3. Test one read-only tool in CodeMie (e.g., `get_person_profile`)
4. If stable, consider creating a Cloudflare account + named Tunnel for production use

## Fallback Options

If Quick Tunnel is blocked by your network:

1. **ngrok** (org may block as P2P): https://ngrok.com/ → requires auth token
2. **Cloudflare Named Tunnel** (stable, auth-capable): https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/
3. **Corporate reverse proxy** (internal only): Contact your IT team for API Gateway/NGINX setup

---

**URL valid until:** Tunnel restart or laptop reboot (temporary)  
**Session created:** 2026-04-23 by [user]  
**Feedback:** If issues arise, validate locally first (`curl http://127.0.0.1:8000/mcp`), then test public URL.
