# Troubleshooting

Look at the log first. It is `autostart.log`, next to `PlcsimAutoStart.exe`:
```powershell
Get-Content .\autostart.log -Tail 40 -Wait
```
The Windows Service writes its own log, `service.log`, in the same folder.

## The page doesn't open after restarting the service
`service.log` shows *"Web app not started yet: waiting for a user signed in at the machine's console"*.
Older versions write *"Waiting for an interactive session … (no user logged on (err 1008))"*.

The service starts the web app only in the session of the user signed in at the machine's console. If
that user's session is a disconnected Remote Desktop session, the service keeps waiting and the page stays
down. Restart the machine: auto-logon signs the user in at the console, and the service starts the app.

## The header says "PLCSIM not responding"
- Is **S7-PLCSIM Advanced** installed?
- The app must run in the session of a signed-in user, not as the hidden SYSTEM account. If you installed it
  some other way, reinstall with `Install.cmd`.
- Does the log say `API DLL: NOT FOUND`? Set `api_dll_path` in `appconfig.txt` (or the `PLCSIM_API_DLL`
  environment variable) to the full path of `Siemens.Simatic.Simulation.Runtime.Api.x64.dll`, then restart
  the service.

## `-48 CommunicationInterfaceNotAvailable` or `NetInterfaces = 0`
PLCSIM can't do networking from the hidden SYSTEM account; it needs a signed-in user.
- Reinstall with `Install.cmd`.
- For unattended boot, turn on auto-logon (`scripts\setup-autologon.ps1`).
- If networking stays stuck, end `s7opnsim.exe` and `S7SimHost.exe` (or restart the machine) to release
  the PLCSIM virtual switch.

## A PLC powers on but stays in STOP ("has no program")
The PLC has no program. **Download your project to it from TIA Portal.** With `storage_layout = default`
the program is saved on disk and is still there after a reboot.

## The page doesn't let me power on another PLC
The **Max. powered on at once** limit is reached. Power a PLC off, or raise the limit.

Before you raise it, make sure the machine can run that many PLCs at once. On a 2-vCPU VM, a second PLC
froze the whole machine. See [Capacity](../README.md#capacity-how-many-plcs-this-machine-can-run).

## The machine froze after powering on another PLC
Power that PLC off from the page. On a 2-vCPU VM the page barely responded, but powering the PLC off from it
worked and the machine recovered. Then set **Max. powered on at once** back to a number the machine can run.

If the page doesn't respond at all, restart the machine from outside it (for example from the Azure
portal).

## The official PLCSIM control panel says "no connection to PLCSIM Advanced"
While this tool is connected to PLCSIM, the control panel can't connect. Click **Enter maintenance mode**
on the page: it disconnects this tool from PLCSIM without stopping the service. **It powers off the PLCs
this tool powered on.** Click **Resume** when you are done.

You don't need maintenance mode to:
- download a project from TIA Portal over TCP/IP: that works while this tool runs;
- create or delete a PLC: use **+ New PLC** and **Delete** on the page.

## "PLCSIM already has saved data under the name …" when creating a PLC
PLCSIM saves each PLC's data in a folder named after the PLC, and that folder is shared by all workspaces.
A folder with that name already has data, maybe from a PLC in another workspace, so the tool doesn't reuse
or erase it. Choose another name.

## TIA Portal refuses to download a project with a different CPU
TIA Portal only downloads to a PLC whose CPU type matches the project, or is unspecified. A PLC keeps the CPU
type of the last project downloaded to it. Click **Erase program** on that PLC: it powers the PLC off and
erases its program and hardware configuration (no backup), so its CPU type becomes unspecified. Then power
it on and download your project from TIA Portal.

## Can't set a PLC's IP
Power the PLC on first, then click **Set IP…**. The tool saves the IP and sets it again every time the PLC
is powered on.

## A PLC powers off by itself after a while
What you see: the PLC's diagnostic buffer shows an orderly "Shut down" (RUN to OFF), the PLC's process ended
with exit code 0, and `autostart.log` has no **POWEROFF** line for it.

- **Version 1.0.0 caused this.** PLCSIM powers a PLC off when the program that powered it on lets go of its
  connection to it. Version 1.0.0 opened a new connection on every status refresh and never kept the one it
  powered the PLC on with. When .NET cleaned that connection up, minutes or hours later (sooner while the
  page was open), PLCSIM powered the PLC off. Newer versions keep one connection per PLC while the app runs.
- **This is still expected when the app stops.** Stopping or restarting the `PLCSIM AutoStart` service, or
  ending `PlcsimAutoStart.exe`, powers off the PLCs it powered on. Auto-start powers them on again when the
  app comes back.
- **A different case:** RUN to STOP, and then the PLC restarts by itself within seconds. Check whether TIA
  Portal was downloading to that PLC at that moment.

## The network mode shows Softbus, or "didn't persist"
Your setting in `appconfig.txt` **is** kept across reboots. Two things to know:
- When no PLC is on, PLCSIM switches its live network mode back to Softbus. The tool sets your mode again
  when it starts and before it powers on a PLC, so PLCs come up in the right mode.
- A **fresh install** creates a new `appconfig.txt` with Softbus, because the settings file isn't in the
  ZIP. Choose your network mode on the page again after reinstalling.

## Red "SAFE MODE" banner
In several boots in a row the machine didn't stay responsive after auto-start, so the tool skipped
auto-start this time on purpose. If the machine can't run that many PLCs at once, lower the **auto-start
cap**. Then click **Re-enable auto-start**.

## Port already in use / `HttpListenerException`
Another program uses port 8090. Stop the service, change `http_prefix` in `appconfig.txt`, and start it
again. Or reinstall with `scripts\install.ps1 -Port <number>`.

## `Access is denied` when listening on the network
Listening on `http://+:8090/` needs a URL reservation. The installer adds it. To add it by hand (elevated):
```powershell
netsh http add urlacl url=http://+:8090/ user="DOMAIN\User"
```

## Rebuild fails: `CS0016 ... being used by another process`
The running app locks `PlcsimAutoStart.exe`. Stop it first (elevated):
```powershell
Stop-Service "PLCSIM AutoStart"            # or: Stop-ScheduledTask -TaskName "PLCSIM AutoStart"
Get-Process PlcsimAutoStart | Stop-Process -Force
.\scripts\build.ps1
Start-Service "PLCSIM AutoStart"           # or: Start-ScheduledTask -TaskName "PLCSIM AutoStart"
```
