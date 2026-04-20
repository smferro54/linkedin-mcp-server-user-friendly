# CodeMie MCP Connection Sheet

**Status**: Template (fill in your values after edge/tunnel is set up)

## Connection Parameters

After running ngrok (see "Edge Setup" below), fill these in:

| Field | Value | Example |
|-------|-------|---------|
| **MCP Server URL** | `https://<ngrok-domain>/mcp` | `https://abc-123-456-789.ngrok.io/mcp` |
| **Authentication Username** | `codemie` (or your choice) | `codemie` |
| **Authentication Password** | (long random string) | `your-secure-password` |
| **Port** | 443 (implicit in HTTPS) | (auto from HTTPS) |
| **SSL/TLS Mode** | Standard HTTPS | Automatic |

## CodeMie UI Configuration (4 steps)

1. **Get your ngrok URL** from the tunnel output (see Quick Start step 4 above).
2. **Open CodeMie** and go to Settings → Add MCP Connection (or equivalent).
3. **Fill in four fields:**
   - **Endpoint URL**: `https://abc-123-456-789.ngrok.io/mcp` (your ngrok URL)
   - **Username**: `codemie`
   - **Password**: `your-secure-password`
   - **SSL/TLS**: Standard HTTPS (default)
4. **Click "Test Connection"** → Should show "Session established" or similar.
5. **Click "List Tools"** or make a test call to confirm MCP tool discovery works.

## MCP Session Behavior (Technical)

CodeMie will:
1. Send POST to `https://<your-domain>/mcp` with body `{"jsonrpc":"2.0","id":1,"method":"initialize",...}`
2. Receive a session ID in the response (e.g., `Mcp-Session-Id: abc123...`)
3. **Must reuse this session ID in all subsequent tool calls** as the `Mcp-Session-Id` HTTP header.

If CodeMie does not preserve the session header, remote calls may fail with auth or state errors. If this happens, see troubleshooting below.

## Edge Setup: ngrok (Recommended)

**Why ngrok?** Free tier includes HTTPS + Basic Auth. One command to run. Perfect for non-technical users.

### Quick Start (3 steps)

1. **Install ngrok**
   ```bash
   # macOS
   brew install ngrok
   
   # Linux
   wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-x86_64.tgz
   tar xzf ngrok-v3-stable-linux-x86_64.tgz
   sudo mv ngrok /usr/local/bin/
   ```

2. **Create a free ngrok account** (https://dashboard.ngrok.com/signup)

3. **Authenticate ngrok**
   ```bash
   ngrok config add-authtoken <your-auth-token>
   # (Copy token from https://dashboard.ngrok.com/auth)
   ```

4. **Start the tunnel with Basic Auth**
   ```bash
   ngrok http 8000 --basic-auth="codemie:your-secure-password"
   ```

   You'll see:
   ```
   Session Status:            online
   Account:                   your-email
   Version:                   3.x.x
   Region:                    us-west
   Latency:                   45ms
   Web Interface:             http://127.0.0.1:4040
   
   Forwarding:                https://abc-123-456-789.ngrok.io -> http://127.0.0.1:8000
   ```

5. **Copy your public HTTPS URL**
   - Your public URL: `https://abc-123-456-789.ngrok.io/mcp`
   - Username: `codemie`
   - Password: `your-secure-password`

### Local Network Alternative (Same Wi-Fi)

If CodeMie is on the same Wi-Fi, you can skip ngrok and use your laptop's local IP:

```bash
# Find your laptop's IP
ifconfig | grep "inet " | grep -v 127.0.0.1

# Example: 192.168.1.50
# CodeMie connects to: http://192.168.1.50:8000/mcp
# No auth needed on local network (but still recommended)
```

### Advanced: Cloudflare Tunnel or VPS (Optional)

If you outgrow ngrok or want more control, see [ADVANCED_EDGE_OPTIONS.md](ADVANCED_EDGE_OPTIONS.md) for Cloudflare Tunnel and VPS reverse proxy setups.

## Preflight Checklist

- [ ] LinkedIn session is logged in locally (`ls ~/.linkedin-mcp/profile/` shows files).
- [ ] Local HTTP MCP server is running on localhost:8000 (`curl http://127.0.0.1:8000/mcp` responds).
- [ ] ngrok is running and shows your public HTTPS URL.
- [ ] ngrok Basic Auth is configured with username and password.
- [ ] CodeMie client supports MCP streamable-http protocol (standard for MCP clients).

## Troubleshooting

### ngrok won't start

```bash
# Check ngrok is installed
ngrok --version

# Check auth token is set
cat ~/.ngrok2/ngrok.yml | grep authtoken

# If missing, re-run:
ngrok config add-authtoken <your-token>
```

### CodeMie fails to connect

1. **Check ngrok is running**: Look for the Forwarding line in terminal.
2. **Check credentials**: Username/password must match what you passed to ngrok.
3. **Test the URL directly**:
   ```bash
   curl -u codemie:your-secure-password https://abc-123-456-789.ngrok.io/mcp
   ```
   Should respond (not error out).

4. **Check local server is still running**: `ps aux | grep linkedin_mcp_server`

### CodeMie initializes but tools fail

1. Confirm CodeMie preserves `Mcp-Session-Id` header on tool calls (MCP protocol requirement).
2. Check MCP server logs for session errors.
3. If CodeMie doesn't support MCP protocol natively, use [MCP Inspector](#fallback-mcp-compatible-bridge-client) as a test.

## Fallback: MCP Inspector Bridge

If CodeMie doesn't work, test with MCP Inspector to isolate the issue:

```bash
bunx @modelcontextprotocol/inspector

# Point to: https://abc-123-456-789.ngrok.io/mcp
# Enter username/password when prompted
```

If Inspector works but CodeMie doesn't, it's a CodeMie client limitation (not MCP server).

## Next Steps

1. **Install ngrok** (if you haven't already) — see Quick Start step 1.
2. **Sign up for ngrok** (free account) — https://dashboard.ngrok.com/signup
3. **Authenticate ngrok** — `ngrok config add-authtoken <your-token>` — step 3 above.
4. **Start the tunnel** — `ngrok http 8000 --basic-auth="codemie:your-secure-password"` — step 4 above.
5. **Copy the public HTTPS URL** from the ngrok terminal output.
6. **Configure CodeMie UI** with URL, username, password (see "CodeMie UI Configuration" above).
7. **Test connection** in CodeMie → should list tools and connect successfully.
8. **Use MCP tools** in CodeMie (search profiles, companies, jobs, etc.).

**Note**: Keep the ngrok terminal running while CodeMie is in use. To stop ngrok, press Ctrl+C.
