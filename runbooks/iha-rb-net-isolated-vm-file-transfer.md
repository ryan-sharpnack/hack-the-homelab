# IHA-RB-NET-ISOLATED-VM-FILE-TRANSFER

**Category:** NET
**Applies to:** any guest VM with no outbound internet access by design (least-privilege segmentation)
**Last updated:** 2026-07-15 (Day 10)

## Purpose

Transfer a file (installer, script, etc.) onto a VM that intentionally has no general internet access, without temporarily opening firewall rules that would undermine the segmentation model. Used for Alloy installer delivery to DC01-IRONBRIDGE and DC01-HAWTHORNE on Day 10; applicable to any future isolated host (WS01, APP01, etc.).

## Procedure

1. Download the target file on the host machine (or any VM that does have internet access).
2. Build a small ISO containing it:
   \`\`\`bash
   sudo apt install -y genisoimage   # skip if already installed
   mkdir -p ~/iso-transfer
   cp <file> ~/iso-transfer/
   genisoimage -o ~/transfer.iso -J -r ~/iso-transfer/
   \`\`\`
   (`-J` Joliet extensions for Windows-readable filenames, `-r` Rock Ridge for permissions.)
3. Attach as a virtual CD-ROM — works with the target VM powered off:
   - **virt-manager GUI:** select VM → Show virtual hardware details → Add Hardware → Storage → select the ISO → Device type: CDROM device → Finish.
   - **virsh CLI:**
     \`\`\`bash
     virsh -c qemu:///system attach-disk <VM-name> ~/transfer.iso sdX --type cdrom --mode readonly --config
     \`\`\`
     Always include `-c qemu:///system` explicitly until `LIBVIRT_DEFAULT_URI` is set permanently — this environment has repeatedly fallen back to `qemu:///session` otherwise.
4. Boot (or continue running) the VM. Inside the guest, copy the file off the mounted drive to local disk.
5. Optional cleanup: detach the virtual CD-ROM from the VM's hardware details once the file's copied off. Read-only and inert either way if left attached.

## Why this approach

Avoids opening temporary outbound internet rules on hosts deliberately segmented against it — a one-time file transfer isn't worth a firewall exception that has to be remembered and reverted later.
