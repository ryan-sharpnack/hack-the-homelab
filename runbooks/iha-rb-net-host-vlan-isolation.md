# Runbook: Host-Level Network Isolation Verification

**Purpose:** Confirm that VLAN10-Corp, VLAN20-Hosp, and VLAN30-Hawth remain isolated at the KVM/QEMU host level — i.e., that FW01 (pfSense) remains the sole inter-VLAN routing path — independent of any single pfSense rule change.

**Related:** Risk Register IHA-006 (host-level cross-VLAN forwarding, Verified – Not Exploitable) recommends periodic re-verification after host OS/libvirt updates. This runbook is that procedure.

**Run this:**
- After any host OS or libvirt package update
- After any change to libvirt network definitions
- Before/after installing or removing Docker on the host (Docker manipulates iptables and can introduce unexpected forwarding rules — see IHA-006 evidence)
- As part of periodic (recommend quarterly) compliance re-verification for GRC-001

---

### Step 1 — Confirm libvirt network scope and state

```
virsh --connect qemu:///system net-list --all
```

**Expected:** `default`, `vlan10-corp`, `vlan20-hosp`, `vlan30-hawth` all show State: active, Autostart: yes, Persistent: yes.

If any network is missing or inactive, stop — investigate before trusting the remaining steps. (Note: omitting `--connect qemu:///system` connects to the per-user session scope instead and will incorrectly show no networks at all — this cost real time during the Day 25 investigation.)

### Step 2 — Confirm kernel-level IP forwarding setting

```
sysctl net.ipv4.ip_forward
```

Record the value. A "1" is expected and is **not** itself a finding — it only matters in combination with Step 3.

### Step 3 — Confirm libvirt firewall rules block inter-VLAN forwarding

```
sudo iptables -L LIBVIRT_FWI -n -v
sudo iptables -L LIBVIRT_FWO -n -v
```

**Expected:** unconditional REJECT rules for virbr1, virbr2, virbr3 in both directions. No ACCEPT or MASQUERADE rules referencing 10.50.10.0/24, 10.50.20.0/24, or 10.50.30.0/24 anywhere in the output.

### Step 4 — Confirm the NAT table has no unexpected masquerade rules

```
sudo iptables -t nat -L -n -v
```

**Expected:** MASQUERADE entries only for 192.168.122.0/24 (libvirt's default NAT network) and/or Docker bridge subnets. No masquerade entries for any 10.50.x.0/24 subnet.

### Step 5 — Confirm no rogue DHCP/DNS services

```
ps aux | grep dnsmasq
```

**Expected:** dnsmasq processes present only for `default.conf`, `vlan10-corp.conf`, `vlan20-hosp.conf`, `vlan30-hawth.conf` (all libvirt-managed). Flag any additional/unexpected dnsmasq instance immediately — don't assume it's benign.

### Step 6 — Empirical cross-VLAN test

From a live host on one VLAN, attempt to reach a host on another VLAN directly, in a scenario where no pfSense allow rule currently exists for that specific traffic (e.g., ICMP between two hosts that only have a narrow TCP rule between them).

**Expected:** fails. A successful connection without a corresponding pfSense rule indicates a host-level bypass and requires immediate investigation — treat as a reopened risk, not a variance to explain away.

---

### Result Logging

Record the outcome and date in Risk Register entry **IHA-006** (Notes field). If any step deviates from expected, do **not** simply re-verify the existing entry — open a new risk record describing the deviation, and reference IHA-006 as the baseline that changed.

### Related Documents
- `docs/network-addressing-and-firewall-rules.md` — the pfSense rule set this isolation model depends on
- Risk Register **IHA-007** — libvirt DHCP conflict, planned for resolution before Phase 3 DC builds
