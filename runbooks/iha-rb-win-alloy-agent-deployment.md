# IHA-RB-WIN-ALLOY-AGENT-DEPLOYMENT

**Category:** Windows
**Related:** IHA-011, IHA-012, Day 17 (Phase 4, Task 1)
**Applies to:** Non-DC Windows hosts on isolated (no outbound internet) VLANs — first used on WS01

## Purpose

Deploys Grafana Alloy on a Windows host with no outbound internet access, using a GUI-based virtual media transfer rather than the CLI-based ISO mount used on the domain controllers (see `iha-rb-siem-loki-grafana-deployment.md` for the original DC method). Confirms ingestion in Grafana Explore and adds the host to the IHA Phase 2 dashboard.

## Prerequisites

- Target VM confirmed to have no outbound internet (verify — don't assume — e.g. a failed DNS lookup in a browser)
- Alloy Windows installer available, either fresh or reused from a prior transfer (`alloy-transfer.iso` on the Kali host in this build)
- Administrator access on the target VM
- SIEM01 powered on and its Loki push endpoint reachable from the target VM's VLAN (confirm firewall rules if the host is on a different segment than SIEM01)

## Procedure

1. **Check the target VM's existing block devices before attaching new media**, so you don't collide with an in-use slot:
   ```bash
   virsh domblklist <VMNAME>
   ```

2. **Attach the installer ISO via virt-manager (GUI method):**
   - Open `virt-manager`, double-click the target VM, click the hardware-details icon
   - **Add Hardware** → **Storage** → **Select or create custom storage** → Browse to the ISO
   - **Device type:** CDROM device · **Bus type:** IDE or SATA
   - **Finish** — this attaches live, no VM restart required in most cases (a shutdown/restart may be needed depending on existing hardware config)

3. **On the target VM:** open File Explorer, locate the new drive letter, copy the installer to a local staging folder (e.g. `C:\Temp`), and run it as Administrator. Accept the default install path (`C:\Program Files\GrafanaLabs\Alloy`) — this registers Alloy as a Windows service with a placeholder config.

4. **Replace the placeholder config.** Location: `C:\Program Files\GrafanaLabs\Alloy\config.alloy`. Open via an elevated Notepad — launch Notepad *from* an elevated PowerShell/Command Prompt session rather than right-clicking Notepad's own shortcut, since File → Open from an already-elevated process is more reliable than elevating after the fact. Replace contents with:
   ```
   loki.write "endpoint" {
     endpoint {
       url = "http://10.50.10.20:3100/loki/api/v1/push"
     }
   }

   loki.source.windowsevent "security" {
     eventlog_name          = "Security"
     use_incoming_timestamp = true
     labels = {
       job      = "windows_eventlog",
       instance = constants.hostname,
     }
     forward_to = [loki.write.endpoint.receiver]
   }

   loki.source.windowsevent "system" {
     eventlog_name          = "System"
     use_incoming_timestamp = true
     labels = {
       job      = "windows_eventlog",
       instance = constants.hostname,
     }
     forward_to = [loki.write.endpoint.receiver]
   }

   loki.source.windowsevent "application" {
     eventlog_name          = "Application"
     use_incoming_timestamp = true
     labels = {
       job      = "windows_eventlog",
       instance = constants.hostname,
     }
     forward_to = [loki.write.endpoint.receiver]
   }
   ```
   This must match the DCs' config exactly — same `job` label spelling (`windows_eventlog`, underscore), same `instance = constants.hostname` pattern — so cross-host LogQL queries work without relabeling.

5. **Restart the service** via `services.msc` → Alloy → Restart (or `Restart-Service Alloy` from an elevated prompt).

6. **Detach the ISO** once install is confirmed complete, via virt-manager (remove the CD-ROM hardware) or `virsh change-media <VMNAME> <device> --eject`.

## Verification

In Grafana Explore, Loki data source:
```
{instance="<HOSTNAME>"}
```
Confirm log lines landing within a minute or two. Then add a dedicated panel to the relevant dashboard (Add → Panel → Logs visualization → same query, scoped) rather than folding the new host into an existing multi-host panel — keeps naming accurate and stays modular as more hosts are added.

## Troubleshooting / Gotchas

- **"Access denied" saving config.alloy:** Notepad wasn't elevated at launch. Opening the file first and elevating after doesn't work — the whole process needs to start elevated.
- **Label mismatch silently splits data:** a hyphen vs underscore in the `job` label (or any label) creates two distinct label values in Loki, which will silently break any cross-host query expecting one consistent value. Always diff against a known-good host's config rather than retyping from memory.
- **No outbound internet on the target VM is not a general Alloy limitation** — it only affects how you get the installer *onto* the box. Once installed, Alloy communicates outbound to SIEM01 over the internal network, which doesn't require internet access at all.

## Related Documentation

- `iha-rb-siem-loki-grafana-deployment.md` (original DC-side Alloy deployment, CLI transfer method)
- `iha-rb-net-isolated-vm-file-transfer.md` (general isolated-VM file transfer pattern)
- Risk register: IHA-011, IHA-012
