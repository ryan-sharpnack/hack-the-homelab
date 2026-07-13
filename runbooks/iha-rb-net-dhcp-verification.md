# IHA-RB-NET-DHCP-VERIFICATION

## Purpose

Confirms whether libvirt-managed DNS/DHCP services (dnsmasq) on the lab's
virtual networks are actively serving DHCP leases, and if so, on which
VLANs. This check exists because libvirt provisions a dnsmasq instance per
virtual network by default, and if any of those instances have DHCP
enabled, it will conflict with the DHCP server role planned for
DC01-IRONBRIDGE / DC01-HAWTHORNE in Phase 3. Running this procedure before
configuring AD DHCP prevents a duplicate-DHCP-server condition on the
network.

## Prerequisites

- Shell access to the KVM/QEMU host with permissions to run `virsh` and
  `ps`
- `libvirt` default connection URI set to `qemu:///system`
  (`export LIBVIRT_DEFAULT_URI=qemu:///system`, or pass `-c qemu:///system`
  explicitly on each command)
- Names of the virtual networks to check (this build: `vlan10-corp`,
  `vlan20-hosp`, `vlan30-hawth`)

## Step-by-step procedure

1. **List running dnsmasq processes** to see which virtual networks have
   an active dnsmasq instance:
   ```
   ps aux | grep dnsmasq
   ```
   Note the config file referenced by each process (e.g.
   `vlan10-corp.conf`, `vlan20-hosp.conf`, `vlan30-hawth.conf`). A running
   dnsmasq process on its own does **not** confirm DHCP is active —
   libvirt also uses dnsmasq for DNS-only virtual networks. This step
   only tells you which networks have *a* dnsmasq instance; it does not
   distinguish DNS-only from DHCP-serving. Treat it as a starting point,
   not a conclusion.

2. **Dump the live XML definition for each virtual network** to check its
   actual configured services:
   ```
   virsh -c qemu:///system net-dumpxml vlan10-corp
   virsh -c qemu:///system net-dumpxml vlan20-hosp
   virsh -c qemu:///system net-dumpxml vlan30-hawth
   ```

3. **Inspect each output for a `<dhcp>` block** inside the `<ip>` element.
   - If `<dhcp>` is present (with a `<range>` defined), that network is
     actively serving DHCP leases.
   - If no `<dhcp>` block is present, the network is DNS-only (or has no
     services configured) — dnsmasq may still be running for DNS, but it
     is not handing out leases.

## Verification

- Confirm you have checked **every** virtual network in scope (all three
  VLANs in this build) — a partial check is not a valid pass.
- Record, per network, whether a `<dhcp>` block was found.
- If any network shows an active `<dhcp>` block, treat this as a
  **blocking finding**: disable the DHCP range on that libvirt network (or
  reconfigure it as non-DHCP) before enabling the AD DHCP server role on
  the corresponding domain controller.
- If no `<dhcp>` block is found on any network, it is safe to proceed with
  configuring the AD DHCP server role in Phase 3.
- Re-run this procedure any time a new virtual network is added, or
  before enabling a DHCP-serving role on a VM, since libvirt's default
  network definitions can change across host updates.

## Related risk register item(s)

IHA-007
