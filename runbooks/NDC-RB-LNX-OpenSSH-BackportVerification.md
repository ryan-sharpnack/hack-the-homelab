Task: Investigate and remediate NDC-007 — OpenSSH below 10.3
Asset: RHEL01 (192.168.123.40)
Date: 2026-06-29

Commands run:
  ssh -V
  sudo dnf check-update openssh
  sudo dnf info openssh
  rpm -q --changelog openssh | head -50

Result: Installed version is OpenSSH 9.9p1 build
23.el10_2.rocky.0.1. No newer version available in
Rocky Linux 10 repos. Changelog confirms 23 patch
revisions with multiple CVEs backported and resolved
by Red Hat maintainers through April 2026.

Verification: rpm changelog shows active CVE remediation
including CVE-2026-35385, CVE-2026-3497, CVE-2025-61984
and others — all resolved within the 9.9p1 package.
Screenshot captured to findings/.

Notes: Rocky Linux follows Red Hat's backporting model.
Security fixes are applied to a stable base version without
incrementing the upstream version number. Nessus flagged
this as vulnerable based on version string comparison alone
— this is a known false positive pattern on RHEL-family
systems. When scanning RHEL/Rocky/AlmaLinux hosts, version
number findings require changelog verification before
actioning. This is distinct from a true vulnerability.
