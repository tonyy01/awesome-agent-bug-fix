# Claude Desktop Cowork: RPC Pipe Closed

## Problem

Claude Desktop's Cowork workspace on Windows 11 fails to start with the error:

```
Failed to start Claude's workspace
Workspace unavailable. The isolated Linux environment failed to start (RPC pipe closed).
```

Any shell command (e.g., `ls -la /sessions/.../mnt/d/`) triggers this error.

## Environment

- **OS:** Windows 11 Pro
- **Claude Desktop:** 1.46388.4.0 (MSIX from Microsoft Store)
- **Virtualization:** Hyper-V, VirtualMachinePlatform, vmcompute all enabled

## Diagnosis Process

### Step 1: Check Claude Desktop logs

```
C:\Users\YE\AppData\Local\Claude\logs\
├── main.log
├── cowork_vm_node.log
├── ssh.log
└── unknown-window.log
```

Key finding in `cowork_vm_node.log`:
```
[info] [cleanupVMBundleIfUnsupported] yukonSilver not supported (status=unsupported)
```

### Step 2: Check CoworkVMService logs

```
C:\ProgramData\Claude\Logs\cowork-service.log
```

Key findings:
- Service starts successfully with `Enforce: true`
- Clients connect but disconnect after 3-5 seconds
- Error: `Failed to read request: failed to read length: EOF`

### Step 3: Identify the pattern

Every startup cycle shows:
```
[Server] Client connected
[Server] Client connected
[Server] Client connected
[Server] Failed to read request: failed to read length: EOF
Service stop requested
```

The client connects, the service tries to verify the client's signature, but the client disconnects before sending a complete request.

### Step 4: Find the root cause

The root cause is **MSIX packaging corrupting the Authenticode signature of `claude.exe`**:

1. `cowork-svc.exe` runs with `Enforce: true` (package identity verified)
2. On each client connection, it verifies the Authenticode signature of `claude.exe`
3. MSIX packaging modifies `claude.exe`, invalidating its embedded signature
4. Signature verification fails → client disconnects → pipe EOF

This is documented in:
- [Anthropic Issue #90283](https://claudeissues.com/issue/90283)
- [GitHub Issue #56195](https://github.com/anthropics/claude-code/issues/56195)

## Fix

### Workaround: Manual service start

```powershell
# 1. Stop the service
Stop-Service CoworkVMService -Force -ErrorAction SilentlyContinue

# 2. Copy cowork-svc.exe out of the package (use xcopy /G for EFS)
xcopy "C:\Program Files\WindowsApps\Claude_1.46388.4.0_x64__pzs8sxrjxfjjc\app\resources\cowork-svc.exe" "C:\Temp\" /G /Y

# 3. Verify signature
Get-AuthenticodeSignature "C:\Temp\cowork-svc.exe" | Select-Object Status
# Should show: Status = Valid

# 4. Run manually (in Administrator PowerShell)
& "C:\Temp\cowork-svc.exe"
# Wait for: "Service ready. Listening on \\.\pipe\cowork-vm-service"

# 5. Start Claude Desktop
```

### Why This Works

When running from outside the MSIX package:
- `Enforce: false` (no package identity)
- Signature verification is bypassed
- Client can connect successfully

### Notes

- This workaround must be repeated after each Claude Desktop update
- Requires Administrator PowerShell (for named pipe creation)
- If `Copy-Item` fails with "Cannot encrypt the specified file", use `xcopy /G`
- The bug is a regression: version 1.34493.1.0 worked, 1.37937.x and later fail

## Timeline

| Date | Version | Status |
|------|---------|--------|
| 2026-08-23 | 1.34493.1.0 | ✅ Working |
| 2026-08-27+ | 1.37937.x | ❌ Broken |
| 2026-09-05 | 1.46388.4.0 | ❌ Still broken |
| 2026-09-07 | 1.46388.4.0 | ✅ Workaround confirmed |

## Lessons Learned

1. **Always check logs before restarting** — the error message told us exactly where to look
2. **Understand the security model** — the fix only makes sense if you know why the signature check fails
3. **A 2-hour diagnosis can save months of workarounds** — now we know exactly what to do every time this breaks
4. **MSIX packaging has quirks** — be aware that packaging can break embedded signatures
