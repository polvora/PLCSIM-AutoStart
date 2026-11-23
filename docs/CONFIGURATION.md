# Settings reference

The settings are in `appconfig.txt`, next to `PlcsimAutoStart.exe`. Each line is `key = value`. Lines that
start with `#` or `;` are comments. An empty value means "use the default".

The tool rewrites this file when you change a setting on the page, so **edit it by hand only while the
service is stopped**. `appconfig.example.txt` is a commented starting point.

## Web server

| Key | Default | Meaning |
|-----|---------|---------|
| `http_prefix` | `http://+:8090/` | Address the page listens on. `http://+:8090/` = reachable from the network. `http://localhost:8090/` = only from this machine. When you install for the network, the installer reserves the address (`netsh http add urlacl`) and opens the port in the firewall. |

## Workspace

| Key | Default | Meaning |
|-----|---------|---------|
| `workspace_root` | `<Documents>\PLCSIM` | Folder that holds your PLCSIM workspaces, in the Documents of the account the app runs as. |
| `workspace` | automatic | The workspace the page shows. Empty = the most recently changed one. You can also pick it on the page. |

## Auto-start at boot

| Key | Default | Meaning |
|-----|---------|---------|
| `autostart_enabled` | `false` | `true` = power on PLCs when the machine boots. |
| `autostart_mode` | `last` | `last` = power on the PLCs that were on before the restart. `fixed` = always power on `autostart_instance`. |
| `autostart_instance` | (empty) | The PLC to power on when `autostart_mode = fixed`. |

## How many PLCs can be on

| Key | Default | Meaning |
|-----|---------|---------|
| `max_powered_on` | `1` | The most PLCs the page lets you power on at once (**Max. powered on at once** on the page). It can be higher than `hard_max_powered_on`. Only raise it if the machine can run that many PLCs at once. |
| `hard_max_powered_on` | `1` | The **auto-start cap**: the most PLCs auto-start powers on at boot. Set it to the number of PLCs the machine can really run. You can change it on the page (it asks you to confirm). |

## Boot protections (loop-breaker)

| Key | Default | Meaning |
|-----|---------|---------|
| `boot_fail_limit` | `2` | How many boots in a row can fail the check below before the tool skips auto-start (SAFE MODE). `1` = skip after one bad boot. |
| `stable_seconds` | `90` | After auto-start, how long PLCSIM and the page must keep answering for the boot to count as good. A good boot sets the failed-boot counter back to 0. |
| `start_stagger_ms` | `4000` | Pause between PLCs during auto-start, so they don't all start at the same moment. |
| `health_probe_interval_ms` | `15000` | How often, during that time, the tool checks that its own page answers. |
| `health_timeout_ms` | `3000` | A check that takes longer than this counts as failed. |
| `last_running` | (managed) | Used by the tool: the PLCs that are on, for `autostart_mode = last`. Leave it alone. |

## Network

| Key | Default | Meaning |
|-----|---------|---------|
| `network_mode` | `Softbus` | `Softbus` = PLCSIM's simulation network, no adapter needed. `TCPIPSingleAdapter` = TCP/IP through the machine's network, no adapter choice. `TCPIPMultipleAdapter` = TCP/IP through the PLCSIM virtual switch, on the adapter you choose. |
| `adapter_index` | `0` | The adapter's `ifIndex`. **Only used in `TCPIPMultipleAdapter` mode.** Pick it on the page; the adapter list only appears in that mode. |
| `adapter_name` | (empty) | That adapter's name, for display only. |

## Advanced

| Key | Default | Meaning |
|-----|---------|---------|
| `storage_layout` | `default` | `default` = use PLCSIM's own saved storage, so each PLC keeps its program after a reboot. **+ New PLC** and **Delete** need this value. |
| `poweron_timeout` | `60000` | How long to wait for a power-on or power-off (ms). |
| `run_timeout` | `60000` | How long to wait for RUN or STOP (ms). |
| `connect_wait_seconds` | `120` | At startup, how long to wait for PLCSIM to answer (s). |
| `api_dll_path` | automatic | Full path of `Siemens.Simatic.Simulation.Runtime.Api.x64.dll`. Empty = find the installed PLCSIM Advanced. You can also set it with the `PLCSIM_API_DLL` environment variable. |
| `log` | `autostart.log` next to the exe | Log file path. |
| `ip.<PLC name>` | (none) | A PLC's saved IP, as `ip,mask,gateway`. **Set IP…** on the page creates it, and the tool sets that IP again every time the PLC is powered on. |
