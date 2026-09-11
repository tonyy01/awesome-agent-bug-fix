# Claude Desktop Cowork: RPC Pipe Closed

## Problem

Claude Desktop's Cowork workspace on Windows 11 fails with:

```
Failed to start Claude's workspace
Workspace unavailable. The isolated Linux environment failed to start (RPC pipe closed).
```

## Environment

- **OS:** Windows 11 Pro
- **Claude Desktop:** 1.37937.x, 1.46388.3.0, 1.46388.4.0 (MSIX from Microsoft Store)

## Fix

1. Stop the service:
   ```powershell
   Stop-Service CoworkVMService -Force -ErrorAction SilentlyContinue
   ```

2. Copy `cowork-svc.exe` out of the package (use `xcopy /G` for EFS):
   ```powershell
   xcopy "C:\Program Files\WindowsApps\Claude_1.46388.4.0_x64__pzs8sxrjxfjjc\app\resources\cowork-svc.exe" "C:\Temp\" /G /Y
   ```

3. Run it manually (Administrator PowerShell):
   ```powershell
   & "C:\Temp\cowork-svc.exe"
   ```
   Wait for: `Service ready. Listening on \\.\pipe\cowork-vm-service`

4. Start Claude Desktop.

## Why This Works

Running outside the MSIX package sets `Enforce: false`, bypassing signature verification that fails due to corrupted Authenticode signature.

## Notes

- Repeat after each Claude Desktop update.
- Requires Administrator PowerShell.
- If `Copy-Item` fails with "Cannot encrypt the specified file", use `xcopy /G`.