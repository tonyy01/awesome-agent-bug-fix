# WorkBuddy + OpenCode Go: 400 Missing x-opencode-session

## Problem

WorkBuddy fails to connect to OpenCode Go backend with:

```
400 Request is missing x-opencode-session and cannot be routed efficiently
```

Happens with models pointing to `opencode.ai/zen/go/v1/chat/completions` (e.g., mimo-v2.5, glm-5.3-flash).

## Environment

- **Tool:** WorkBuddy (custom model configuration)
- **Backend:** OpenCode Go (opencode.ai)
- **Date:** 2026-09-06 onwards

## Fix

Run a local proxy that injects the required headers.

### 1. Create the proxy script

Save the following as `opencode_go_proxy.py` (any directory, e.g., `~/opencode-go-proxy/`):

```python
#!/usr/bin/env python3
# OpenCode Go header proxy for WorkBuddy
# Fixes 400 error: "missing x-opencode-session"
import http.client
import http.server
import json
import os
import socketserver
import uuid

# Bypass local proxy for upstream connection
os.environ["no_proxy"] = "opencode.ai"
os.environ["NO_PROXY"] = "opencode.ai"

UPSTREAM_HOST = "opencode.ai"
UPSTREAM_PORT = 443
UPSTREAM_BASE = "/zen/go/v1"
LISTEN_HOST = "127.0.0.1"
LISTEN_PORT = 18904

# Fixed session ID for routing affinity + prompt caching
SESSION_ID = str(uuid.uuid4())
CLIENT = "workbuddy"

class Handler(http.server.BaseHTTPRequestHandler):
    protocol_version = "HTTP/1.1"

    def _forward(self, method):
        length = int(self.headers.get("Content-Length", 0) or 0)
        body = self.rfile.read(length) if length else b""

        path = self.path.split("?", 1)[0]
        if not path.startswith(UPSTREAM_BASE):
            path = UPSTREAM_BASE + (path if path.startswith("/") else "/" + path)

        # Forward headers, inject required ones
        fwd = {}
        for k, v in self.headers.items():
            if k.lower() in ("host", "connection", "content-length",
                             "transfer-encoding", "proxy-authorization",
                             "proxy-connection"):
                continue
            fwd[k] = v
        fwd["x-opencode-session"] = SESSION_ID
        fwd["x-opencode-client"] = CLIENT
        fwd["User-Agent"] = "workbuddy/1.0"
        fwd["Content-Length"] = str(len(body))

        print(f"[go-proxy] {method} {path} (session {SESSION_ID[:8]})", flush=True)

        try:
            conn = http.client.HTTPSConnection(UPSTREAM_HOST, UPSTREAM_PORT, timeout=300)
            conn.request(method, path, body=body, headers=fwd)
            resp = conn.getresponse()
            status = resp.status
            data = resp.read()
            conn.close()
        except Exception as e:
            print(f"[go-proxy] upstream error: {e}", flush=True)
            self.send_response(502)
            self.send_header("Content-Type", "application/json")
            self.end_headers()
            self.wfile.write(json.dumps({"error": str(e)}).encode())
            return

        self.send_response(status)
        for k, v in resp.getheaders():
            if k.lower() in ("transfer-encoding", "connection", "content-length"):
                continue
            self.send_header(k, v)
        self.send_header("Content-Length", str(len(data)))
        self.end_headers()
        if data:
            self.wfile.write(data)

    def do_POST(self):
        self._forward("POST")

    def do_GET(self):
        self._forward("GET")

    def do_OPTIONS(self):
        self.send_response(204)
        self.end_headers()

    def log_message(self, *a):
        pass

class ThreadingHTTPServer(socketserver.ThreadingMixIn, http.server.HTTPServer):
    daemon_threads = True

if __name__ == "__main__":
    print(f"[go-proxy] listening http://{LISTEN_HOST}:{LISTEN_PORT} session={SESSION_ID}", flush=True)
    print(f"[go-proxy] forwarding -> https://{UPSTREAM_HOST}{UPSTREAM_BASE}", flush=True)
    ThreadingHTTPServer((LISTEN_HOST, LISTEN_PORT), Handler).serve_forever()
```

### 2. Create start.bat (optional)

Save as `start.bat` in the same directory:

```bat
@echo off
cd /d "%~dp0"
echo Starting OpenCode Go header proxy on 127.0.0.1:18904 ...
python opencode_go_proxy.py
pause
```

### 3. Run the proxy

- Option A: Double-click `start.bat`
- Option B: Run directly:
  ```bash
  cd /path/to/directory/containing/script
  python opencode_go_proxy.py
  ```

### 4. Update WorkBuddy config

Edit `~/.workbuddy/models.json`:
- Change model `url` to `http://127.0.0.1:18904/chat/completions`
- Keep `apiKey` unchanged

### 5. Restart WorkBuddy

Or reselect the model in the UI.

## Why This Works

OpenCode Go requires `x-opencode-session` header for routing. WorkBuddy doesn't send it. The proxy injects the header before forwarding.

## Notes

- Keep the proxy running while using WorkBuddy with OpenCode Go models.
- Uses fixed UUID for session header.