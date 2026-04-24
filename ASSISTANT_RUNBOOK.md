# Assistant Runbook: Headless MCP + Cloudflare Quick Tunnel + CodeMie

Use this runbook when a coding assistant needs to set up this MCP server on a laptop and connect it to CodeMie.

Default runtime is headless. Only the one-time login step is interactive.

## Goal

Expose local MCP endpoint `http://127.0.0.1:8000/mcp` through Cloudflare Quick Tunnel and produce a working CodeMie JSON config.

## Constraints

1. Quick Tunnel is temporary and unauthenticated.
2. URL changes when tunnel restarts.
3. Keep setup to local laptop only (`127.0.0.1`).
4. `--login` opens a browser window; normal MCP runtime is headless unless `--no-headless` is used.

## Required Inputs

1. OS: `ubuntu`, `macos`, or `windows` (PowerShell)
2. Git repo path (or clone URL)
3. Confirmation that user can complete LinkedIn login in browser

## Success Criteria

1. Local initialize call succeeds on `http://127.0.0.1:8000/mcp`.
2. Public initialize call succeeds on `https://<random>.trycloudflare.com/mcp`.
3. CodeMie JSON generated with the active tunnel URL.

## Procedure

### 1) Install prerequisites

Ubuntu:

```bash
sudo apt update
sudo apt install -y git curl jq unzip python3 python3-pip
curl -LsSf https://astral.sh/uv/install.sh | sh
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

macOS:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install git curl jq uv
```

Windows (PowerShell):

```powershell
winget install --id Git.Git -e
winget install --id astral-sh.uv -e
```

If using Windows, use WSL version 2 (not WSL 1).

Open a new PowerShell window after install.

Validation:

```bash
uv --version
```

### 2) Clone and install dependencies

```bash
git clone https://github.com/smferro54/linkedin-mcp-server-user-friendly.git
cd linkedin-mcp-server-user-friendly
uv sync
```

Validation:

```bash
uv run python -c "import linkedin_mcp_server; print('ok')"
```

### 3) Create/refresh LinkedIn session (interactive)

```bash
uv run -m linkedin_mcp_server --login
```

Expected:

1. Browser opens.
2. User logs in and completes challenge prompts.
3. Command exits successfully.

### 4) Start local MCP server (Terminal A, headless)

```bash
uv run -m linkedin_mcp_server --transport streamable-http --host 127.0.0.1 --port 8000 --path /mcp
```

Keep Terminal A running.

Optional debug mode (visible browser):

```bash
uv run -m linkedin_mcp_server --no-headless --transport streamable-http --host 127.0.0.1 --port 8000 --path /mcp
```

### 5) Validate local MCP initialize

In Terminal B:

```bash
curl -s -X POST http://127.0.0.1:8000/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'
```

Windows (PowerShell) equivalent:

```powershell
$headers = @{
  "Content-Type" = "application/json"
  "Accept" = "application/json, text/event-stream"
}

$body = @{
  jsonrpc = "2.0"
  id = 1
  method = "initialize"
  params = @{
    protocolVersion = "2025-03-26"
    capabilities = @{}
    clientInfo = @{
      name = "test"
      version = "1.0"
    }
  }
} | ConvertTo-Json -Depth 10

Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/mcp" -Headers $headers -Body $body
```

Expected:

JSON response containing `"result"` and `"serverInfo"`.

### 6) Install cloudflared

Ubuntu:

```bash
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloudflare-main.gpg
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt update
sudo apt install -y cloudflared
```

macOS:

```bash
brew install cloudflare/cloudflare/cloudflared
```

Windows (PowerShell):

```powershell
winget install --id Cloudflare.cloudflared -e
```

Validation:

```bash
cloudflared --version
```

### 7) Start Cloudflare Quick Tunnel (Terminal C)

```bash
cloudflared tunnel --url http://127.0.0.1:8000
```

Capture tunnel URL from output, for example:

`https://example-name.trycloudflare.com`

Construct MCP URL:

`https://example-name.trycloudflare.com/mcp`

### 8) Validate public MCP initialize

In Terminal B:

```bash
curl -s -X POST https://example-name.trycloudflare.com/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'
```

Windows (PowerShell) equivalent:

```powershell
$headers = @{
  "Content-Type" = "application/json"
  "Accept" = "application/json, text/event-stream"
}

$body = @{
  jsonrpc = "2.0"
  id = 1
  method = "initialize"
  params = @{
    protocolVersion = "2025-03-26"
    capabilities = @{}
    clientInfo = @{
      name = "test"
      version = "1.0"
    }
  }
} | ConvertTo-Json -Depth 10

Invoke-RestMethod -Method Post -Uri "https://example-name.trycloudflare.com/mcp" -Headers $headers -Body $body
```

Expected:

JSON response containing `"result"` and `"serverInfo"`.

### 9) Produce CodeMie JSON

Use this template with active tunnel URL:

```json
{
  "command": "npx",
  "url": null,
  "args": [
    "-y",
    "mcp-remote",
    "https://example-name.trycloudflare.com/mcp"
  ],
  "headers": {},
  "env": {},
  "type": null,
  "auth_token": null,
  "single_usage": false,
  "tools": null,
  "audience": null
}
```

### 10) Take the JSON and paste it to Codemie

## Failure Handling

1. `cloudflared: command not found`
- Install cloudflared, then rerun step 7.

2. `Not Acceptable: Client must accept both application/json and text/event-stream`
- Add header `Accept: application/json, text/event-stream`.

3. Public URL returns error/timeout
- Check Terminal A (MCP server) is still running.
- Check Terminal C (cloudflared) is still running.
- Restart tunnel and update URL in CodeMie JSON.

4. LinkedIn tools fail after initialize
- Re-run `uv run -m linkedin_mcp_server --login`.

5. `TargetClosedError: Target page, context or browser has been closed`
- If using `--no-headless`, the debug browser was likely closed manually.
- Restart MCP server in headless mode and retry.
- If still failing, re-run `uv run -m linkedin_mcp_server --login` and restart MCP.

6. Browser does not start in WSL / missing Linux browser libraries
- Ensure Windows is using WSL 2.
- In WSL (Ubuntu), install required browser dependencies:

```bash
sudo apt update && sudo apt install -y \
  libnss3 libatk1.0-0 libatk-bridge2.0-0 libcups2 libdrm2 \
  libxkbcommon0 libxcomposite1 libxdamage1 libxrandr2 \
  libgbm1 libasound2t64 libpangocairo-1.0-0 libpango-1.0-0
```

7. cloudflared error about ping group range (for example: Group ID 1000 is not between ping group range)
- cloudflared may fail to ping when the current user group is outside `ping_group_range`.
- Check your group ID:

```bash
id -g
cat /proc/sys/net/ipv4/ping_group_range
```

- Temporary fix (until reboot):

```bash
sudo sysctl -w net.ipv4.ping_group_range="0 2147483647"
```

- Persistent fix:

```bash
echo 'net.ipv4.ping_group_range = 0 2147483647' | sudo tee /etc/sysctl.d/99-cloudflared-ping.conf
sudo sysctl --system
```

- Retry the tunnel command after applying the change:

```bash
cloudflared tunnel --url http://127.0.0.1:8000
```

## Operational Notes

1. Quick Tunnel has no built-in auth.
2. Share URL only with intended tester.
3. Stop tunnel after demo.
4. For authenticated/stable endpoint, move to Cloudflare Named Tunnel + Cloudflare Access.

## Stop Commands

Use `Ctrl+C` in Terminal A and Terminal C.
