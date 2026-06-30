Task: Create and enforce account lockout policy via GPO
Asset: DC01 / WS01
Date: 2026-06-29

Commands run:
  gpupdate /force        (on WS01 — forced policy refresh)
  gpresult /r            (on WS01 — confirmed policy application)

Settings configured:
  Account lockout threshold:              5 invalid attempts
  Account lockout duration:               15 minutes
  Reset account lockout counter after:    15 minutes
  Allow Administrator account lockout:    Enabled

GPO name: NDC-Account-Lockout-Policy
Linked to: homelab.local (domain level)

Result: Policy created in GPMC, configured with lockout threshold
and Administrator lockout enabled, linked at domain level,
confirmed applied on WS01 via gpresult /r output.

Verification: NDC-Account-Lockout-Policy visible under Applied
Group Policy Objects in gpresult /r Computer Settings output.
Screenshots captured to findings/.

Notes: Built-in Administrator account (RID 500) now subject to
lockout policy. In a production environment monitor for accidental
Administrator lockout. Console access to VM available for recovery
if needed.
