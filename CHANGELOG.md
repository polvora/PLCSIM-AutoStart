# Changelog

All notable changes to this project are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- **New PLC and Delete** on the web page. **+ New PLC** registers a blank PLC (no program, CPU type
  unspecified) in PLCSIM and adds it to the workspace's `Instance.JSON`. It refuses a name that already
  exists, or that already has saved data in PLCSIM's storage. **Delete** works on a powered-off PLC: it
  deletes the PLC's saved data (no backup), unregisters it from PLCSIM and removes it from the workspace.
  Both need `storage_layout = default`.
- **Maintenance mode**: a button that disconnects this tool from PLCSIM without stopping the service, so the
  official PLCSIM control panel can connect. Entering it powers off the PLCs this tool powered on. Click
  **Resume** when you are done.
- **Erase program**: a button on each PLC that erases its saved program and hardware configuration, so
  TIA Portal can download a project with a *different* CPU. It powers the PLC off, erases its storage (no
  backup) and unregisters it. The PLC's CPU type is now shown in the list.

### Changed
- Clearer texts on the web page, in its messages, in the logs and in the documentation. The page now says
  "PLC" everywhere: **+ New CPU** is now **+ New PLC**, and **Wipe CPU** is now **Erase program**.
- The documentation no longer says that the page limit (`max_powered_on`) can't exceed the auto-start cap.
  It can, and on a 2-vCPU VM a second PLC froze the whole machine. The README now has a Capacity section
  with that example.
- The page no longer shows "Unconfigured PLC · not configured" under each PLC. Those values come from the
  workspace file and don't say whether the PLC has a program.

### Fixed
- **PLCs shutting down by themselves at random.** The symptom was an orderly "Shut down" (RUN to OFF) in
  the diagnostic buffer and an exit code 0, minutes or hours after power-on, typically while the web UI
  was open. The cause: the app created a new PLCSIM API interface (`CreateInterface` / `RegisterInstance`)
  on every status request and every operation, and never kept or released it. Each of those objects has
  a finalizer. When the .NET GC finalized the interface an instance had been powered on with, the Runtime
  Manager shut that instance down; frequent status polling made the GC run sooner. Interfaces are now
  created once per registered instance (one for control, one for monitoring), reused, and kept alive for
  the life of the process.

## [1.0.0] - 2026-06-03

First public release.

### Added
- **Remote web control** of the PLCs in a Siemens S7-PLCSIM Advanced workspace (power on, RUN, STOP,
  power off) from a browser, on this machine or any other machine on the network.
- **Instance auto-start** at boot, with two modes: `last` (restore whatever was running before
  the last shutdown) and `fixed` (always start one chosen instance).
- **Persistent by default**: instances are registered against PLCSIM's native persistent storage, so a
  PLC's downloaded program survives restarts.
- Configurable **power-on limit** plus a separate **auto-start safety cap** (editable in the UI,
  with a warning) that bounds how many PLCs auto-start brings up at boot.
- **Boot protections** against a freeze/restart loop:
  - boot-attempt counter that enters SAFE MODE after N non-stabilizing boots,
  - a "clean boot" check gated on repeated self `/health` probes (detects a soft freeze where
    processes stay alive but the web layer becomes unresponsive),
  - staggered start (one instance at a time, abort on failure).
- **Per-PLC IP override** that is re-applied on every power-on.
- Networking via **Softbus** (default, zero-config) or **TCP/IP** mapped to a host adapter.
- Self-contained: builds with the in-box .NET Framework `csc.exe` (no Visual Studio / .NET SDK) and
  auto-detects the Siemens PLCSIM Advanced API DLL.
- Guided installer, uninstaller, and an optional auto-logon helper for unattended boot.
- Installs by default as a **Windows Service** (manageable from `services.msc` / Task Manager). Because
  PLCSIM can't run from session 0, the service is a launcher that starts the web app in the logged-in
  interactive session with the user's elevated token. `-AsTask` installs a Scheduled Task instead.
- The executable is named **`PlcsimAutoStart.exe`** (matching the project name).
