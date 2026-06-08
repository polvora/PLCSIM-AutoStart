# PLCSIM Auto-Start

**Automatic startup (and remote web control) for Siemens S7-PLCSIM Advanced virtual PLCs.**

![PLCSIM Auto-Start web interface](docs/ui.png)

PLCSIM Auto-Start is a small always-on web app that **extends** S7-PLCSIM Advanced — it does **not**
replace it. You still create and configure your virtual PLCs in the Siemens PLCSIM Advanced GUI as
usual; PLCSIM Auto-Start reads that workspace and adds what the GUI doesn't give you:

- 🔄 **Automatic startup** — your PLCs come back up on their own after a server reboot, completely
  unattended. This is the headline feature. Before turning it on, read
  [enabling auto-start safely](#enabling-auto-start-capacity-and-the-freeze-loop-risk) — there is one
  capacity decision that keeps a busy machine from getting stuck on boot.
- 🌐 **Remote control from a browser** — power on, RUN, STOP and power off your PLCs from any machine
  on the network. Manage a headless simulation host from your own desktop or another VM, with no
  remote-desktop session needed.
- 💾 **Persistent by default** — every instance is registered against PLCSIM's persistent storage, so a
  PLC's downloaded program survives a restart. Combined with auto-start, a PLC comes back on its own
  after a reboot — nothing to re-open, nothing to re-download from TIA.

> Independent, open-source tool — **not** affiliated with Siemens. It uses the Siemens PLCSIM Advanced
> API, which you install separately under your own Siemens license. The proprietary Siemens DLL is
> **not** included here; the tool locates the one already on your machine.

---

## Other features

Beyond remote control and auto-start:

- **Power-on limit** — cap how many PLCs can be powered on at once (default **1**, adjustable in the UI).
- **Per-PLC IP override**, re-applied on every power-on, so a PLC stays reachable on your subnet.
- **Network mode**: Softbus (zero-config) or TCP/IP mapped to a host adapter.
- **Maintenance mode** — a one-click release of the PLCSIM connection so the official control panel /
  TIA Portal can connect (e.g. to add a new instance) without stopping the service. Powered-on PLCs keep
  running; one click resumes.
- **Installs as a Windows Service** you Start/Stop from `services.msc` / Task Manager. (Because PLCSIM
  needs an interactive session, the service is a launcher that runs the app in the logged-in session;
  `-AsTask` uses a Scheduled Task instead. See [docs/INSTALL.md](docs/INSTALL.md#how-the-service-works-the-session-0-catch).)

---

## Requirements

- **Windows 10 / Windows Server 2016 or newer** (x64).
- **Siemens S7-PLCSIM Advanced** installed (tested with **V20**). Provides the runtime and the API DLL.
- **.NET Framework 4.x** — built into modern Windows; no Visual Studio or .NET SDK required.
- A **logged-in Windows session** — i.e. a user signed in to the desktop, not the hidden SYSTEM
  context. PLCSIM needs this for networking, so the Windows Service runs as a *launcher* that starts the
  app inside the logged-in session. For a server to recover on its own after a reboot, enable auto-logon
  (the installer offers it; see [docs/INSTALL.md](docs/INSTALL.md)).

---

## Quick start (no programming needed)

1. **Download** the project — green **Code** button → **Download ZIP**. You get
   **`PLCSIM-AutoStart-main.zip`** (the prebuilt `PlcsimAutoStart.exe` is inside). Extract it anywhere;
   it produces a `PLCSIM-AutoStart-main` folder.
2. **Double-click `Install.cmd`** and accept the admin prompt (UAC). It sets everything up: detects
   your PLCSIM Advanced install, makes the UI reachable from the LAN (no authentication; it opens the
   firewall for the port), creates `appconfig.txt`, installs an always-on **Windows Service** (Start/Stop
   it from `services.msc` / Task Manager), and offers to enable **auto-logon** for fully unattended boot.
   *(Command-line alternative: run `scripts\install.ps1` from an elevated PowerShell; add `-LocalOnly`
   to bind to localhost.)*
3. Open the UI:
   - on this machine: **http://localhost:8090**
   - from another machine: **http://&lt;this-machine-ip&gt;:8090**

That's it. To remove it later, run `.\scripts\uninstall.ps1` as administrator.

---

## For developers

The prebuilt `PlcsimAutoStart.exe` ships in the repo. Only if you change the backend
(`src\PlcsimAutoStart.cs`) or UI (`wwwroot\index.html`), rebuild with `.\scripts\build.ps1` — no IDE
needed, just the in-box .NET Framework compiler.

---

## Enabling auto-start: capacity and the freeze-loop risk

Auto-start powers your PLCs on at **every** boot, with no one watching. That convenience comes with one
thing to get right, because a virtual S7-PLCSIM PLC is **heavy** — each running instance takes real CPU
and RAM on the host.

**The failure mode (why the cap exists).** If you let auto-start bring up more PLCs than the machine can
actually run at the same time, the host can bog down — or, in the worst case, lock up — while those
instances all start at boot. And because auto-start runs again on *every* reboot, a freeze that forces a
restart just hits the same overload again: **freeze → reboot → freeze**, a loop that can leave the
machine unreachable until someone intervenes. Many PLCSIM users never run into this; but on a headless,
auto-logon server it is exactly the situation that can turn a small misjudgment about capacity into a
stuck box. The point isn't that it *will* freeze — it's that nobody is there to catch it if it does.

**The decision you make.** The *auto-start cap* (`hard_max_powered_on`) is simply **the most PLCs you are
confident this machine can run unattended at once**. Auto-start never brings up more than that, so the
boot path stays inside a number you trust. The default is **1** — the safe baseline. It governs the
**boot path only**: your manual power-on limit (`max_powered_on`) can go higher, so you can still test
real capacity by hand without changing what happens on the next reboot. Both are editable in the UI (the
cap behind a confirmation, since raising it is the risky direction).

**The backstop, if a setup still misbehaves.** A loop-breaker watches for the freeze loop above: a
counter is bumped before each auto-start and cleared only after the service passes repeated `/health`
probes for a while — so a *soft freeze* (processes alive but the machine unresponsive) won't fool it
into thinking the boot was clean. After `boot_fail_limit` boots that never stabilize, the service enters
**SAFE MODE**: it skips auto-start, shows a red banner in the UI, and waits for you to fix the load and
click *Re-enable*. This is the seatbelt — set the cap correctly and you should never need it.

Every value is tunable in [docs/CONFIGURATION.md](docs/CONFIGURATION.md).

---

## Documentation

- [docs/INSTALL.md](docs/INSTALL.md) — install, auto-logon, LAN access.
- [docs/CONFIGURATION.md](docs/CONFIGURATION.md) — every `appconfig.txt` key.
- [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) — common problems and fixes.

## License

[MIT](LICENSE) © 2026 Marcelo Tapia. Not affiliated with Siemens.
