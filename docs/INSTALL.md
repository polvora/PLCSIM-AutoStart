# Installation

## Requirements
- Windows 10 / Windows Server 2016 or newer (x64)
- Siemens **S7-PLCSIM Advanced** (tested with V20). The installer finds its API DLL by itself.
- .NET Framework 4.x (already part of Windows)

## Install
1. Download the project: green **Code → Download ZIP**. You get **`PLCSIM-AutoStart-main.zip`**, with the
   ready-to-run `PlcsimAutoStart.exe` inside. Extract it.
2. **Double-click `Install.cmd`** and accept the administrator prompt (UAC). It finds the PLCSIM DLL, opens
   the port in the firewall, creates `appconfig.txt`, and installs the **Windows Service "PLCSIM
   AutoStart"**, which you can start and stop from `services.msc`.

   > Because the file comes from the internet, Windows may first show a warning ("Windows protected your
   > PC" or "Open File - Security Warning"). Choose **More info → Run anyway**.

   Instead of `Install.cmd` you can run `.\scripts\install.ps1` from an elevated PowerShell:
   - `-LocalOnly`: the page is reachable only from this machine.
   - `-AsTask`: install a Scheduled Task instead of the service.
3. Open **http://localhost:8090**, or `http://<this-machine-ip>:8090` from another machine.

To manage it (or use `services.msc`):
```powershell
Start-Service "PLCSIM AutoStart"
Stop-Service  "PLCSIM AutoStart"
Get-Content .\autostart.log -Tail 30 -Wait
```

## How the service works
PLCSIM Advanced only does networking inside the session of a **signed-in Windows user**. A normal service
runs in the hidden SYSTEM session, where PLCSIM networking fails with `-48 CommunicationInterfaceNotAvailable`.

So the service only **starts the app for you**. It runs as LocalSystem, so it appears in `services.msc` and
you can start and stop it there. When it starts, it launches the web app in the session of the user signed
in at the machine's console, with that user's administrator rights.

That means a user must be signed in **at the console**. A disconnected Remote Desktop session doesn't
count: in that case the service waits and the page stays down (see
[TROUBLESHOOTING](TROUBLESHOOTING.md#the-page-doesnt-open-after-restarting-the-service)). For a machine
that must recover by itself after a reboot, turn on auto-logon (below).

`-AsTask` doesn't use the service: a Scheduled Task starts the app when the user signs in.

## Unattended boot (auto-logon)
PLCSIM only runs while a user is signed in, so a machine that must recover by itself after a reboot needs
auto-logon. **The installer offers it** and asks for the account's password. To set it up later:
```powershell
.\scripts\setup-autologon.ps1
```
At boot, Windows signs that account in by itself, and the service starts the app. The script uses
Sysinternals **Autologon** (password stored encrypted) if it finds it. Otherwise it uses the registry, which
stores the password **in plain text**, so use that only on a secured machine. Read the warning at the top of
the script first.

## Network or this machine only
By default the page is reachable from the network (`http_prefix = http://+:8090/`, plus a URL reservation
and a firewall rule). To make it reachable only from this machine, install with `-LocalOnly`. There is no
login, so keep it on a trusted network.

## Uninstall
```powershell
.\scripts\uninstall.ps1
```
It removes the service (or task), the firewall rule and the URL reservation. It leaves your files,
settings, logs and workspaces. If you set up auto-logon, undo it with Sysinternals Autologon, or set
`AutoAdminLogon` to `0` under `HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`.
