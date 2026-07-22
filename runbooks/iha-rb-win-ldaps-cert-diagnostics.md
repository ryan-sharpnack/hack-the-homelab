# IHA Runbook: Diagnosing a Non-Functional LDAPS Listener on Active Directory

**Component:** DC01-IRONBRIDGE (IRONBRIDGE.LOCAL) — Active Directory LDAPS (TCP 636)
**Date:** July 21, 2026 (IHA Day 16)
**Status:** Root cause confirmed; remediation attempted, unresolved; deprioritized (see Outcome)
**Related risk register entry:** IHA-013

---

## Purpose

Captures the diagnostic method used to trace an LDAPS bind failure from APP01 against DC01-IRONBRIDGE back to a specific, root cause — not just the fix attempted. The symptom in this case (TLS resets immediately after a successful TCP handshake) is a recognizable, reusable signature worth documenting on its own, independent of whether the specific remediation below fully resolved it.

## Symptom

An LDAPS bind from a non-domain-joined client (APP01, authenticating as `svc-app01`) against a domain controller's port 636 fails, but the failure mode is ambiguous at first glance — it is not a connection-refused error, and not a credential/bind-rejected error either.

```bash
ldapsearch -x -H ldaps://10.50.10.10:636 -D "svc-app01@ironbridge.local" -w '<password>' \
  -b "DC=ironbridge,DC=local" -s base
```
```
ldap_sasl_bind(SIMPLE): Can't contact LDAP server (-1)
```

That message alone is not diagnostic — it can mean a firewall block, a routing problem, a dead service, or (as in this case) a TLS-layer failure. Isolating which one requires dropping below `ldapsearch` to the TCP/TLS layer directly.

## Diagnostic Methodology

**1. Confirm this isn't a network/firewall problem before suspecting the certificate.**

```bash
openssl s_client -connect 10.50.10.10:636
```
```
CONNECTED(00000003)
write:errno=104
---
no peer certificate available
---
New, (NONE), Cipher is (NONE)
Protocol: TLSv1.3
```

`CONNECTED` confirms the TCP three-way handshake succeeded — the port is open, something is listening, and no firewall rule or routing issue is blocking the connection. If nothing were listening at all, this would instead show `Connection refused` before ever reaching `CONNECTED`.

The failure happens one layer up: `write:errno=104` (`ECONNRESET`) fires the instant the client sends its TLS ClientHello, and the session ends with `Cipher is (NONE)` — no TLS session ever negotiated. Port open, TCP fine, TLS resets instantly with nothing exchanged: this specific combination is the signature of a listener that is up and accepting connections but has **no certificate to present**, not a network-layer problem.

**2. Confirm the certificate's *presence* isn't the same as its *usability*.**

On the DC:
```powershell
Get-ChildItem Cert:\LocalMachine\My | Format-List Subject, Thumbprint, NotAfter, HasPrivateKey, EnhancedKeyUsageList
```

A certificate can pass every check here — correct `Subject`, `HasPrivateKey: True`, `Server Authentication` present in `EnhancedKeyUsageList` — and still be unusable by the process that actually needs it. Do not treat a clean `Get-ChildItem` result as proof the LDAPS bind will work; it only proves the certificate object exists in the store.

**3. Go to the authoritative source: what AD itself logged when it tried to use the certificate.**

```powershell
Get-WinEvent -LogName "Directory Service" -MaxEvents 100 | `
  Where-Object {$_.Id -eq 1220 -or $_.Id -eq 1221} | `
  Format-List TimeCreated, Id, Message
```

- **Event 1221** — AD found and successfully bound a certificate. If this appears, the certificate side is confirmed fine and the problem lies elsewhere.
- **Event 1220** — AD looked for a certificate and rejected every candidate. Critically, the message body names the specific reason rather than leaving it to guesswork.

In this case, Event 1220 returned:
```
LDAP over Secure Sockets Layer (SSL) will be unavailable at this time because the server was unable to obtain a certificate.
Additional Data
Error value:
8009030e No credentials are available in the security package
```

`8009030e` (`SEC_E_NO_CREDENTIALS`) with a certificate that otherwise looks valid in `Get-ChildItem` is the specific signature of this root cause: a key-storage-provider mismatch.

## Root Cause

`New-SelfSignedCertificate` (the PowerShell cmdlet) defaults to storing the certificate's private key via **CNG** (the modern Key Storage Provider). AD DS's LDAPS binding is handled by `lsass`/Schannel, which — in this environment's configuration — cannot access CNG-stored keys for this purpose. The certificate exists, has a private key, and reports the correct EKU to any tool that inspects it, but `lsass` cannot actually retrieve and use that key at bind time, and fails silently to the fallback state Event 1220 describes.

This is a known-enough issue that Microsoft's own documented procedure for enabling LDAPS avoids `New-SelfSignedCertificate` for exactly this reason, using `certreq` with an INF file that explicitly forces the legacy CSP instead.

## Remediation Attempted

**1. Removed the CNG-backed certificate:**
```powershell
Get-ChildItem Cert:\LocalMachine\My | `
  Where-Object {$_.Subject -eq "CN=DC01-IRONBRIDGE.ironbridge.local"} | Remove-Item
```

**2. Generated a replacement request forcing the legacy CSP**, saved as `ldaps-cert-request.inf`:
```
[Version]
Signature="$Windows NT$"

[NewRequest]
Subject = "CN=DC01-IRONBRIDGE.ironbridge.local"
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
MachineKeySet = TRUE
SMIME = FALSE
PrivateKeyArchive = FALSE
UserProtected = FALSE
UseExistingKeySet = FALSE
ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
ProviderType = 12
RequestType = Cert
KeyUsage = 0xa0

[EnhancedKeyUsageExtension]
OID=1.3.6.1.5.5.7.3.1
```

**3. Generated and installed the certificate directly** (no CA required — `RequestType = Cert` self-signs and installs in one step):
```powershell
certreq -new C:\ldaps-cert-request.inf C:\ldaps-cert.cer
```

**4. Rebooted** so `lsass` would pick up the new certificate fresh, then re-checked the event log and retested from APP01 (steps 1 and 3 above).

**Result:** the identical failure signature persisted — `errno=104`, `Cipher is (NONE)` — even with the legacy-CSP certificate in place. The `certreq`/legacy-CSP approach, while the documented correct fix for this specific error class, did not resolve it in this environment on the first attempt.

## Outcome

Deprioritized rather than pursued further, for a specific reason: **Phase 5's actual dependency on `svc-app01` — Kerberoasting — operates over Kerberos (TCP/UDP 88), which is entirely independent of whether LDAPS (636) functions.** The SPN registered against `svc-app01` via `setspn` already provides everything Phase 5 needs. Continuing to debug LDAPS specifically would have been solving a problem the roadmap doesn't currently require solved.

Logged as **IHA-013**, status Open, likelihood Low / impact Medium. Longer-term remediation path: a full AD Certificate Services (AD CS) deployment on IRONBRIDGE.LOCAL, which would issue and auto-enroll a properly-chained certificate rather than relying on a manually-provisioned self-signed one — and would likely sidestep this specific CSP issue by handling key storage through the standard enrollment path rather than either PowerShell method used here.

## References

- Risk Register: IHA-013
- Microsoft documentation: `certreq` / INF-based certificate request for LDAPS enablement
- Event Viewer: `Directory Service` log, Event IDs 1220 / 1221
