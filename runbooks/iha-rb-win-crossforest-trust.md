# IHA-RB-WIN-CROSSFOREST-TRUST

## Purpose

Establish a one-way forest trust where IRONBRIDGE.LOCAL trusts HAWTHORNE.LOCAL, matching the Phase 2 build roadmap exit criteria ("Trust established and visible in AD") and the Phase 1 identity architecture design decision to build the trust in its default, broad configuration — Forest-wide authentication, no Selective Authentication — with scoping deferred to the planned Phase 4/5 hardening redesign.

## Prerequisites

- DC01-IRONBRIDGE and DC01-HAWTHORNE both powered on and reachable
- Domain Admin credentials on both forests
- No pre-existing conditional forwarder or trust objects left over from a prior attempt (see Troubleshooting below if rebuilding)

## Procedure

### 1. Conditional forwarders

On DC01-IRONBRIDGE:
```powershell
Add-DnsServerConditionalForwarderZone -Name "HAWTHORNE.LOCAL" -MasterServers 10.50.30.10
```

On DC01-HAWTHORNE:
```powershell
Add-DnsServerConditionalForwarderZone -Name "IRONBRIDGE.LOCAL" -MasterServers 10.50.10.10
```

Verify both directions before proceeding:
```powershell
Resolve-DnsName -Name hawthorne.local -Server 10.50.10.10
Resolve-DnsName -Name ironbridge.local -Server 10.50.30.10
```
Each should return an A record with the correct DC IP in the answer section. See evidence:
- `../evidence/2026-07-12/dc01-ironbridge-dns-forwarder-verification.png`
- `../evidence/2026-07-12/dc01-hawthorne-dns-forwarder-verification.png`

### 2. Create the trust

Run the New Trust Wizard separately on each DC (Active Directory Domains and Trusts → right-click domain → Properties → Trusts tab → New Trust), using "This domain only" for Sides of Trust on both sides.

| Run wizard on | Trust name entered | Trust type | Direction | Sides | Authentication level |
|---|---|---|---|---|---|
| DC01-IRONBRIDGE | HAWTHORNE.LOCAL | Forest trust | One-way: outgoing | This domain only | Forest-wide authentication |
| DC01-HAWTHORNE | IRONBRIDGE.LOCAL | Forest trust | One-way: incoming | This domain only | *(not prompted — incoming-only side)* |

**Critical:** the trust password entered on the IRONBRIDGE side must be re-entered identically on the HAWTHORNE side. Since each side is created independently ("This domain only"), the shared password is what links the two trust objects into one working relationship.

A verification failure (error 1787) on the IRONBRIDGE side immediately after finishing the wizard is expected and can be ignored — it's just the wizard trying to confirm a HAWTHORNE-side object that doesn't exist yet. It resolves once the HAWTHORNE-side wizard is completed with the matching password.

### 3. Validate

Preferred method — GUI validate button, which surfaces clearer errors than the console utility:
1. IRONBRIDGE.LOCAL Properties → Trusts tab → select HAWTHORNE.LOCAL → Properties
2. Click **Validate**

Confirm the relationship visually on both sides:
- IRONBRIDGE.LOCAL Properties → Trusts tab: HAWTHORNE.LOCAL listed under *outgoing trusts*
- HAWTHORNE.LOCAL Properties → Trusts tab: IRONBRIDGE.LOCAL listed under *incoming trusts*
- Neither domain should show an entry in the opposite box (confirms genuinely one-way)

Evidence:
- `../evidence/2026-07-12/dc01-ironbridge-trust-properties-outgoing.png`
- `../evidence/2026-07-12/dc01-hawthorne-trust-properties-incoming.png`

### 4. Verify authentication scope and direction via PowerShell

On DC01-IRONBRIDGE:
```powershell
Get-ADTrust -Filter * | Format-List Name, Direction, TrustType, ForestTransitive, SelectiveAuthentication
```
Expected for Phase 2:
- `Direction : Outbound`
- `ForestTransitive : True`
- `SelectiveAuthentication : False`

Evidence: `../evidence/2026-07-12/dc01-ironbridge-adtrust-getadtrust-verification.png`

Console-based verification (secondary, GUI Validate is primary):
```powershell
netdom trust IRONBRIDGE.LOCAL /domain:HAWTHORNE.LOCAL /verify /verbose
```

## Troubleshooting notes (encountered during Day 28 rebuild)

- **Stale conditional forwarder from a prior attempt:** `Add-DnsServerConditionalForwarderZone` fails with `ResourceExists` if a forwarder zone with that name already exists. Check with `Get-DnsServerZone | Where-Object {$_.ZoneName -like "*<domain>*"}` and remove with `Remove-DnsServerZone -Name "<domain>" -Force` before recreating.
- **Removing a trust object requires cross-forest credentials:** `netdom trust <domain> /d:<otherdomain> /remove /force` fails with "user name or password is incorrect" unless you supply credentials for the other domain: add `/Ud:<OTHERDOMAIN>\Administrator /Pd:*`. Supplying the other domain's credentials removes the trust object from both sides in one command — a second removal attempt from the other DC may then fail with "the specified domain either does not exist or could not be contacted," which is expected once nothing remains to remove. Confirm actual state with `Get-ADTrust -Filter *` on both DCs rather than treating that message as a new problem.
- **Leftover trust with mismatched authentication scope:** a trust object surviving from a prior session may show `SelectiveAuthentication : True` when the current build phase calls for Forest-wide authentication (or vice versa for a later hardening phase). Always check `Get-ADTrust` before assuming an existing trust is usable as-is.

## Notes

This trust is intentionally left at Forest-wide authentication for Phase 2, per the Phase 1 design decision to represent a realistic, unscoped post-acquisition trust. Do not add Selective Authentication or "Allowed to Authenticate" ACEs until the Phase 4/5 hardening redesign — that scoping-down step is itself a planned part of the project's vulnerable-state → redesign narrative.
