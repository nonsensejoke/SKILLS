---
name: tinyfish-mcp
description: Set up and configure TinyFish MCP (web search and page fetch) for Hermes Agent via OAuth PKCE flow.
version: 1.0.0
author: hermes
license: MIT
metadata:
  hermes:
    tags: [MCP, OAuth, Web, Search, Fetch, TinyFish]
prerequisites:
  commands: [python3, firefox]
---

# TinyFish MCP Setup for Hermes Agent

TinyFish is a web toolkit for agents providing live web search and page content extraction. Search and fetch are free with no credits or cost.

## What TinyFish Provides

- **Search** — Query the live web and get structured, agent-ready results. Use for current information, news, prices, or anything that changes over time.
- **Fetch** — Pull the full content of any web page as clean extracted text. Use to read articles, docs, product pages, or any URL you need to reason over.

## Setup Procedure

### Step 1: Register an OAuth Dynamic Client

TinyFish uses OIDC dynamic client registration. Register a client that Hermes will use:

```bash
python3 << 'PYEOF'
import json, urllib.request

reg = json.dumps({
    "client_name": "hermes-agent",
    "redirect_uris": ["http://127.0.0.1:9999/callback"],
    "response_types": ["code"],
    "grant_types": ["authorization_code"],
    "token_endpoint_auth_method": "none"
}).encode()

req = urllib.request.Request('https://agent.tinyfish.ai/oauth/register', data=reg)
req.add_header('Content-Type', 'application/json')

resp = urllib.request.urlopen(req)
result = json.loads(resp.read())
print(json.dumps(result, indent=2))
PYEOF
```

Save the `client_id` from the response — it will be used in the next step.

Note: The existing client `kLBulPWh1DPMfnuk` (registered as `mcporter-cli`) also works and can be reused instead of registering a new one.

### Step 2: Generate PKCE Parameters and Auth URL

```bash
python3 << 'PYEOF'
import secrets, base64, hashlib, json
from urllib.parse import quote

# Change these to match your client registration
cid = 'vpjNizO2at9CQjG4'  # or kLBulPWh1DPMfnuk
ru = 'http://127.0.0.1:9999/callback'

cv = secrets.token_urlsafe(48)
cc = base64.urlsafe_b64encode(hashlib.sha256(cv.encode()).digest()).rstrip(b'=').decode()
state = secrets.token_urlsafe(32)

auth_url = 'https://agent.tinyfish.ai/oauth/authorize?response_type=code&client_id=' + cid + '&redirect_uri=' + quote(ru, '') + '&scope=' + quote('openid profile email offline_access') + '&code_challenge=' + cc + '&code_challenge_method=S256&state=' + state

cfg = json.dumps({"cv": cv, "state": state, "cid": cid, "ru": ru, "auth_url": auth_url}, indent=2)
print(cfg)

# Save for step 4
with open('/tmp/tf_oauth.json', 'w') as f:
    json.dump({"cv": cv, "state": state, "cid": cid, "ru": ru}, f)
PYEOF
```

Save the output JSON (especially `cv` and `state`) to `/tmp/tf_oauth.json`.

### Step 3: Start Callback Listener and Open OAuth URL

Start a local HTTP server to catch the OAuth redirect:

```bash
python3 << 'PYEOF'
import http.server, sys
from urllib.parse import urlparse, parse_qs

class H(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        p = urlparse(self.path)
        q = parse_qs(p.query)
        if p.path == '/callback':
            code = q.get('code', [None])[0]
            st = q.get('state', [None])[0]
            print('CAUGHT_CODE:' + str(code), flush=True)
            print('CAUGHT_STATE:' + str(st), flush=True)
            self.send_response(200)
            self.send_header('Content-Type', 'text/plain')
            self.end_headers()
            self.wfile.write(b'Auth complete! You can close this tab.')
        else:
            self.send_response(200)
            self.send_header('Content-Type', 'text/plain')
            self.end_headers()
            self.wfile.write(b'Listening...')
    def log_message(s, *a): pass

srv = http.server.HTTPServer(('127.0.0.1', 9999), H)
print('LISTENING on :9999', flush=True)
srv.serve_forever()
PYEOF
```

Then open the auth URL from Step 2 in the user's browser:

```bash
DISPLAY=:10.0 firefox --new-window 'AUTH_URL_HERE'
```

### Step 4: Exchange Auth Code for Tokens

After the user approves the OAuth flow (sign in + consent), they will be redirected to `http://127.0.0.1:9999/callback?code=***&state=STATE`. Capture the `code` value.

Then exchange it for tokens:

```bash
python3 << 'PYEOF'
import json, urllib.parse, urllib.request

with open('/tmp/tf_oauth.json') as f:
    cfg = json.load(f)

code = 'CODE_FROM_REDIRECT'  # Replace with actual code
cid = cfg['cid']
ru = cfg['ru']
cv = cfg['cv']
expected_state = cfg['state']

# Verify state
if received_state != expected_state:
    print('STATE MISMATCH - possible CSRF')
    exit(1)

# Exchange code for tokens
token_body = 'grant_type=authorization_code&client_id=' + cid + '&redirect_uri=' + urllib.parse.quote(ru, '') + '&code=' + code + '&code_verifier=' + urllib.parse.quote(cv, '')

req = urllib.request.Request('https://agent.tinyfish.ai/oauth/token', data=token_body.encode())
req.add_header('Content-Type', 'application/x-www-form-urlencoded')

resp = urllib.request.urlopen(req)
tokens = json.loads(resp.read())

# Save tokens (do NOT truncate them!)
with open('/tmp/tinyfish_tokens.json', 'w') as f:
    json.dump(tokens, f, indent=2)

print('Tokens received successfully')
print('access_token length: ' + str(len(tokens['access_token'])))
print('expires_in: ' + str(tokens['expires_in']))
PYEOF
```

### Step 5: Add TinyFish to Hermes Config

Add the MCP server to `~/.hermes/config.yaml` under the `mcp_servers` key with the Bearer token:

```bash
python3 << 'PYEOF'
import json, yaml

with open('/tmp/tinyfish_tokens.json') as f:
    tokens = json.load(f)

at = tokens['access_token']  # Full token, not truncated

config_path = '/home/raymond/.hermes/config.yaml'
with open(config_path) as f:
    config = yaml.safe_load(f)

if 'mcp_servers' not in config:
    config['mcp_servers'] = {}

config['mcp_servers']['tinyfish'] = {
    'url': 'https://agent.tinyfish.ai/mcp',
    'headers': {
        'Authorization': 'Bearer ' + at
    },
    'connect_timeout': 60,
    'timeout': 120
}

with open(config_path, 'w') as f:
    yaml.dump(config, f, default_flow_style=False, sort_keys=False)

print('TinyFish MCP server added to Hermes config')
PYEOF
```

### Step 6: Reload and Verify

After adding the config, Hermes needs to reload MCP servers. In an interactive session, use:

```
/reload-mcp
```

Or restart the agent. On next startup, Hermes will:
1. Connect to TinyFish via HTTP transport
2. Discover 21 tools (search, fetch, and related browser automation tools)
3. Register them with the prefix `mcp_tinyfish_*`

Verify with:

```bash
hermes mcp list
```

## Important Notes

- **Auth codes are single-use** — each code can only be exchanged once. If exchange fails, you must redo the OAuth flow from Step 2.
- **Do NOT truncate the access token** — Python print truncation (`v[:20]+'...'`) will save an unusable token to config. Always save the full token string.
- **Token expiration** — Access tokens expire in ~24 hours (86400s). The refresh token can be used to get new ones, but Hermes currently does not auto-refresh MCP tokens. If tools start returning 401, redo the OAuth flow.
- **Port conflicts** — The callback server uses port 9999. If that port is in use, change it in both the client registration `redirect_uris` and the HTTP server bind address.
- **Cloudflare bot detection** — Opening the OAuth URL in Hermes's built-in browser (Browserbase) may trigger Cloudflare verification. Use the local Firefox/browser instead.

## Troubleshooting

### 401 Unauthorized on MCP connect
The access token has expired. Redo the OAuth flow (Steps 2-5) to get a fresh token.

### "STATE MISMATCH" error
The state parameter in the redirect URL doesn't match what was generated. This is a security check — restart from Step 2.

### Port 9999 already in use
Kill the existing process or use a different port (e.g., 9998). Update the port in both the client registration and callback server.

### Cloudflare blocking the browser
The built-in browser may trigger bot detection. Use `DISPLAY=:10.0 firefox --new-window 'URL'` to open on the local desktop instead.

### MCP tools not appearing after config
- Check YAML indentation in `~/.hermes/config.yaml`
- Run `hermes mcp list` to see server status
- Look at startup logs for connection errors
- Use `/reload-mcp` in an active session

## Quick Reference

| Item | Value |
|------|-------|
| OAuth issuer | https://agent.tinyfish.ai |
| Auth endpoint | https://agent.tinyfish.ai/oauth/authorize |
| Token endpoint | https://agent.tinyfish.ai/oauth/token |
| Registration endpoint | https://agent.tinyfish.ai/oauth/register |
| MCP server URL | https://agent.tinyfish.ai/mcp |
| Scopes | openid, profile, email, offline_access |
| PKCE | S256 required |
| Config key | mcp_servers.tinyfish |
| Tool prefix | mcp_tinyfish_* |
| Token lifetime | ~24 hours |
