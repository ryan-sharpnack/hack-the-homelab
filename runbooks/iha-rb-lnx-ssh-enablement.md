# IHA-RB-LNX-SSH-ENABLEMENT

## Purpose

Enables SSH access on a Linux lab VM so it can be administered directly
(log tailing, config edits, service restarts) without needing the VM
console window for routine work, and documents the corresponding pfSense
firewall rule. This becomes more important as WS01/APP01 come online in
Phase 3 and multiple hosts need to be managed in parallel.

## Prerequisites

- Console access to the target VM (required for the initial install,
  since SSH isn't available yet)
- `sudo` privileges on the VM
- Access to the pfSense web UI with rule-editing permissions
- The VM's documented static IP (per
  `network-addressing-and-firewall-rules.md`)

## Step-by-step procedure

1. **From the VM's console**, install and enable the OpenSSH server:
   ```
   sudo apt update && sudo apt install openssh-server -y
   sudo systemctl enable ssh --now
   ```

2. **Verify the service is active and listening**, still from the
   console:
   ```
   sudo systemctl status ssh
   ss -tlnp | grep :22
   ```
   Expect `active (running)` and a listener on port 22 (both `0.0.0.0:22`
   and `[::]:22`).

3. **Add a pfSense firewall rule** (Firewall → Rules → the interface the
   target VM sits on):
   - **Action:** Pass
   - **Protocol:** TCP
   - **Source:** the specific client that will be connecting — see the
     note below before setting this to "Any"
   - **Destination:** the VM's static IP (Address or Alias)
   - **Destination Port Range:** SSH (22)
   - **Description:** short, specific (e.g. `SSH admin access to
     SIEM01`)
   - **Save**, then **Apply Changes**

4. **Test the connection** from the intended client:
   ```
   ssh <username>@<VM IP>
   ```

## Important note — hypervisor host vs. external clients

If the machine you're connecting *from* is the KVM/QEMU hypervisor host
itself (as opposed to a separate physical or virtual client), be aware
that traffic between the host and a guest VM on the same virtual bridge
is switched at Layer 2 and **never traverses FW01**, regardless of any
pfSense rule. Confirm this before assuming a pfSense rule is the actual
access control in effect:
```
ip route get <VM IP>
```
If the output shows a `virbrX` device rather than a route through FW01,
the pfSense rule you configured is not what's permitting this specific
connection — the host's implicit administrative reach over its own
guests is. The pfSense rule still has value for genuinely external
clients (a separate laptop, a future dedicated attack-simulation VM in
Phase 5), but don't rely on it as the control for host-to-guest access.

## Verification

- `systemctl status ssh` shows `active (running)` on the target VM
- `ss -tlnp | grep :22` shows an active listener
- SSH connection from the intended client succeeds and authenticates
- If the client is the hypervisor host, confirm via `ip route get` that
  the connection path is understood, rather than assumed to be
  firewall-mediated

## Related risk register item(s)

None directly. Authentication is currently password-based rather than
key-based; consider a future `IHA-RB-SEC-SSH-KEY-HARDENING.md` runbook
if key-based auth is adopted, at which point a risk register entry
tracking password-based SSH as an interim state may be warranted.
