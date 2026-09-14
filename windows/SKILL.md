---
name: windows-daemons
description: >
  Windows desktop daemon lessons: Shell_NotifyIconW E_FAIL traps, headless
  message pump lifecycles, and Task Manager Startup registry splits. Use when
  writing background daemons, system tray apps, or autostart tools on Windows.
---

# Windows Desktop Daemon Lessons

Lessons from building autonomous background daemons on Windows 11.

## 1. Shell_NotifyIconW fails with 0x80004005 in console processes

Bug: Calling `Shell_NotifyIconW(NIM_ADD, &nid)` from a background Rust/console
binary failed with `2147500037` (`0x80004005` E_FAIL). Crates like `tray-icon`
and `tray-item` crashed on startup when invoked headless.

**Fix:** Windows Shell Notify APIs require an explicit Win32 message pump
running on an STA thread. In raw Win32, call `RegisterClassW` + a hidden window
message pump; in .NET/WinForms, wrap in `[STAThread]` and `ApplicationContext`.

**Check:** Run the daemon under a non-interactive/hidden window launcher — tray
icon must register and return success, not E_FAIL.

## 2. Headless NotifyIcon silently exits without ApplicationContext

Bug: A background tray daemon launched via `-WindowStyle Hidden` appeared in
process list for 2 seconds and exited without an error or Event Viewer entry.
Garbage collection swept the tray handle because no main window was open.

**Fix:** Pass an explicit `ApplicationContext` to `Application.Run(context)`. Do
not rely on parameterless `Application.Run()` when the process has no visible
Form window.

**Check:** `Get-Process <name>` 10 seconds after hidden launch must show active
PID with stable working set.

## 3. Task Manager Startup tab ignores HKCU Run without StartupApproved

Bug: Adding an executable to `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
ran the app on boot, but the app was completely invisible in Windows 11 Task
Manager's "Startup apps" tab.

**Fix:** Task Manager requires a companion entry in
`HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\StartupApproved\Run`
with binary value `02 00 00 00 00 00 00 00 00 00 00 00`. Update both registry
paths during install/setup.

**Check:** Open Task Manager → Startup apps; verify entry displays with status
"Enabled" rather than missing.
