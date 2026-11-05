# Remediation

For each misconfiguration I exploited, here's the concrete fix and the hardening
control behind it. Fixes are ordered by impact,
do the top ones first.

## Priority summary

| # | Finding | Exploited in | Severity | Fix effort |
|---|---------|--------------|----------|------------|
| 1 | `b.otieno` in Domain Admins (stale) | Phase 04 | Critical | Low |
| 2 | `b.otieno` holds DCSync replication rights | Phase 03 | Critical | Low |
| 3 | `b.otieno` has Kerberos pre-auth disabled | Phase 02 | High | Low |
| 4 | `svc_sql` Kerberoastable (SPN + weak/RC4) | Phase 01 | High | Medium |

The first three all sit on **one account** (`b.otieno`), which is why the chain is so
severe, and why fixing that single account collapses most of the attack path.

---

## 1. Remove stale Domain Admins membership  (T1087.002, the DA edge)

**Problem.** `b.otieno`, a Tier-2 helpdesk account, is a member of **Domain Admins**:
a "temporary" grant that was never revoked. BloodHound shows it as a direct edge to
domain takeover.

**Fix.**
```powershell
Remove-ADGroupMember -Identity "Domain Admins" -Members b.otieno -Confirm:$false
```
**Hardening control.** Adopt **tiered administration** (Microsoft's Tier 0/1/2 model):
helpdesk accounts live in Tier 2 and can *never* hold Tier 0 (domain) rights. Review
privileged group membership on a schedule and require just-in-time elevation (e.g.
time-bound group membership) instead of standing admin rights.

## 2. Strip replication (DCSync) rights  (T1003.006)

**Problem.** `b.otieno` holds `DS-Replication-Get-Changes` and
`DS-Replication-Get-Changes-All` on the domain root: the two rights DCSync needs.
Only Domain Controllers (and sanctioned sync services like Entra Connect) should have
these.

**Fix: audit who holds them, then remove:**
```powershell
$domainDN = (Get-ADDomain).DistinguishedName
(Get-Acl "AD:$domainDN").Access |
  Where-Object { $_.ObjectType -in `
    '1131f6aa-9c07-11d1-f79f-00c04fc2dcd2', `   # Get-Changes
    '1131f6ad-9c07-11d1-f79f-00c04fc2dcd2' } |  # Get-Changes-All
  Select-Object IdentityReference, ObjectType, AccessControlType
# Then remove the ACE for any principal that is not a DC / sanctioned service.
```
**Hardening control.** Baseline the domain-root ACL and alert on changes to it.
Treat replication rights as Tier 0. If `krbtgt` was ever exposed (it was, here),
**rotate the krbtgt password twice** (with a replication interval between the two
rotations) to invalidate any forged Golden Tickets.

## 3. Re-enable Kerberos pre-authentication  (T1558.004)

**Problem.** `b.otieno` has `DoesNotRequirePreAuth = True`, so the DC hands out its
AS-REP to anyone: no credentials needed to harvest a crackable hash.

**Fix.**
```powershell
Set-ADAccountControl -Identity b.otieno -DoesNotRequirePreAuth $false
```
**Hardening control.** Pre-auth is on by default; nothing should turn it off. Audit
for it domain-wide on a schedule:
```powershell
Get-ADUser -Filter 'DoesNotRequirePreAuth -eq $true' -Properties DoesNotRequirePreAuth
```
Combine with a strong password policy so that even a leaked AS-REP can't be cracked
offline in reasonable time.

## 4. Fix the Kerberoastable service account  (T1558.003)

**Problem.** `svc_sql` is a regular user account carrying an SPN
(`MSSQLSvc/dc01.lab.local:1433`) with a weak password and RC4 enabled, so any
domain user can request an RC4-encrypted service ticket and crack it offline.

**Fix (best to acceptable):**
- **Best:** convert to a **Group Managed Service Account (gMSA)** so the password is
  128 chars, machine-managed, and rotated automatically, un-crackable in practice.
- **Otherwise:** set a **25+ character random password** and force AES, removing RC4:
  ```powershell
  Set-ADUser svc_sql -KerberosEncryptionType AES128,AES256   # drop RC4
  ```
- **Always:** remove SPNs that aren't actually needed.

**Hardening control.** Inventory all SPN-bearing accounts, enforce gMSA/long-random
passwords for service accounts, and disable RC4 domain-wide where compatibility
allows. Detection-side, alert on RC4 (0x17) TGS requests, which is exactly what my
[detection-engineering](https://github.com/manlikefelo/detection-engineering) rule **100010** does.

---

## Verifying the fixes

After applying all four, the BloodHound graph should show **no path** from an
ordinary user to Domain Admin, and re-running Phases 01–03 should fail to recover any
credentials. In the lab I reset to DC01's clean snapshot between runs so the
misconfigs can be re-demonstrated; in production you'd confirm each fix and re-scan.
