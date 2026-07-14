# IHA-RB-LNX-STATIC-NETWORK-CONFIG

## Purpose

Verifies that a Linux VM's static network configuration matches the
documented design (per `network-addressing-and-firewall-rules.md`), and
corrects it if it doesn't — specifically the default gateway, which
Ubuntu's Subiquity installer can set incorrectly at OS install time. This
procedure also ensures the fix survives a reboot on cloud-init-managed
systems, where a hand-edited netplan file can otherwise be silently
regenerated back to its original (wrong) value.

This exists because libvirt's isolated virtual networks assign the host
bridge the `.1` address on each VLAN subnet (see the `.1` vs. `.254`
callout in `network-addressing-and-firewall-rules.md`). If a VM's gateway
is mistakenly set to `.1` instead of the documented `.254`, the host
bridge will silently answer for it — making the VM *appear* to have a
working gateway even when FW01 (the actual, intended gateway) is powered
off. This can mask the misconfiguration for a long time before it's
noticed.

## Prerequisites

- Shell access to the VM with `sudo` privileges
- The VM's documented, correct gateway address (per
  `network-addressing-and-firewall-rules.md` — `.254` on each VLAN in
  this build)
- The VM uses netplan for network configuration (standard on modern
  Ubuntu Server installs)

## Step-by-step procedure

1. **Check the currently active gateway:**
   ```
   ip route
   ```
   Compare the `default via ...` line against the documented gateway
   address for this VM's VLAN.

2. **If it matches, stop here** — no further action needed.

3. **If it doesn't match, identify the netplan config file:**
   ```
   ls /etc/netplan/
   ```
   Typically a single file, e.g. `50-cloud-init.yaml`.

4. **View the file to confirm the incorrect value and its exact
   location:**
   ```
   sudo cat /etc/netplan/<filename>.yaml
   ```
   Look for the `routes:` block and its `via:` line.

5. **Correct only the gateway value.** Editing a single line is safer
   than retyping the block — netplan/YAML is whitespace-sensitive, and a
   full retype risks breaking indentation. One safe way to do this
   without opening an editor (avoids manual typos):
   ```
   sudo sed -i 's/via: <WRONG_GATEWAY>/via: <CORRECT_GATEWAY>/' /etc/netplan/<filename>.yaml
   ```
   Replace `<WRONG_GATEWAY>` and `<CORRECT_GATEWAY>` with the actual
   addresses. Confirm the change:
   ```
   sudo cat /etc/netplan/<filename>.yaml
   ```

6. **Apply the change with a safe rollback window:**
   ```
   sudo netplan try
   ```
   This applies the new config immediately and reverts automatically if
   not confirmed within the countdown — safer than `netplan apply` for a
   gateway change, since a bad config could otherwise cut off remote
   access.

7. **Confirm the live routing table reflects the fix:**
   ```
   ip route
   ```

8. **Disable cloud-init's network management** so the fix persists
   across reboots (cloud-init will otherwise regenerate the netplan file
   from its original datasource on next boot, silently reverting the
   fix):
   ```
   sudo mkdir -p /etc/cloud/cloud.cfg.d
   echo 'network: {config: disabled}' | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
   ```
   Confirm the file's content is exactly `network: {config: disabled}`:
   ```
   sudo cat /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
   ```

9. **Reboot and re-verify** — this is the actual proof the fix is
   permanent, not just live in the current session:
   ```
   sudo reboot
   ```
   After it comes back up:
   ```
   ip route
   ```

## Verification

- `ip route` shows the correct, documented gateway both immediately
  after the fix and again after a full reboot.
- The `99-disable-network-config.cfg` file exists and contains exactly
  `network: {config: disabled}`.
- **Red flag to watch for:** if a VM can ping its configured "gateway"
  while FW01 is confirmed powered off, do not treat this as a working
  connection — investigate it. On this build, the host bridge itself can
  answer for `.1` addresses independent of FW01, so a response doesn't
  prove the gateway is correctly reaching FW01.

## Related risk register item(s)

IHA-009
