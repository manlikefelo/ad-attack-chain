# Active Directory Attack Chain

An **authorized** red-team engagement against my own `lab.local` Active Directory,
run entirely inside an isolated home lab. Starting from a single low-privileged
domain user (in one case, from *no credentials at all*), I chain four techniques
into full domain compromise, and every action is mapped to a MITRE ATT&CK
technique ID and written up so a reader can follow the reasoning, not just the
commands.

> ### Scope & authorization
> Everything here targets **DC01 (`lab.local`, 10.10.10.10)** on an isolated,
> host-only lab network that I built and own, see my
> [`security-homelab`](https://github.com/manlikefelo/security-homelab) repo for how it was stood up. No system
> outside that lab was touched. All credentials, hashes, and tickets below are
> lab-only throwaways; nothing ties to a real organization. This is the legal,
> ethical way to practise offensive AD tradecraft: **attack only what you own.**

---

## The story in one line

Four intentional misconfigurations, one over-permissioned account (`b.otieno`), and
a straight line from *"I have nothing"* to *"I own the domain"*:

```
AS-REP roast (no creds)  ─►  b.otieno's password
        │
        ▼
   DCSync as b.otieno     ─►  every hash in the domain, incl. krbtgt
        │
        ▼
   Golden Ticket key      ─►  effectively permanent domain control

  (+ Kerberoasting svc_sql, and BloodHound mapping every path to Domain Admin)
```

## Engagement phases

Each phase is a self-contained write-up under [`/engagement`](engagement/) with the
exact commands, real screenshots from the attacker box, and the telemetry it
generated. Read them in order. They build on each other.

| # | Phase | Technique (ATT&CK) | What it achieves | Write-up |
|---|-------|--------------------|------------------|----------|
| 01 | Kerberoasting | **T1558.003** | Crack `svc_sql`'s password from a normal user | [Open](engagement/01-kerberoasting/) |
| 02 | AS-REP Roasting | **T1558.004** | Crack `b.otieno`'s password with **no credentials** | [Open](engagement/02-asrep-roasting/) |
| 03 | DCSync | **T1003.006** | Replicate every domain hash (incl. `krbtgt`) as `b.otieno` | [Open](engagement/03-dcsync/) |
| 04 | BloodHound mapping | **T1087.002 / T1069.002** | Graph every attack path to Domain Admin | [Open](engagement/04-bloodhound/) |

## ATT&CK technique table

| Tactic | Technique | ID | Where |
|--------|-----------|----|----|
| Credential Access | Kerberoasting | T1558.003 | Phase 01 |
| Credential Access | AS-REP Roasting | T1558.004 | Phase 02 |
| Credential Access | Brute Force: Password Cracking | T1110.002 | Phases 01, 02 |
| Credential Access | OS Credential Dumping: DCSync | T1003.006 | Phase 03 |
| Credential Access | Steal or Forge Kerberos Tickets: Golden Ticket | T1558.001 | Phase 03 (enabled) |
| Discovery | Account Discovery: Domain Accounts | T1087.002 | Phase 04 |
| Discovery | Permission Groups Discovery: Domain Groups | T1069.002 | Phase 04 |

## Tools used

All free, all Kali defaults: `impacket` (GetUserSPNs, GetNPUsers, secretsdump),
`bloodhound-python` + BloodHound CE, `john` (CPU cracker. This host has no GPU, so
John substitutes for the canonical hashcat modes; each write-up notes the hashcat
command too), `netexec`, `nmap`.

## Detections generated
Every attack here was **detected** on the blue-team side. The offensive telemetry
each phase produces (Event IDs 4769, 4768, 4662, 4661) is caught by custom Wazuh
rules in my companion repo:

> **[detection-engineering](https://github.com/manlikefelo/detection-engineering)**: the same four techniques,
> written as portable Sigma rules and deployed Wazuh rules, each verified firing
> end-to-end. Attack here → detect there. Each attack has a matching detection.

## Read next

- **[`executive-summary.md`](executive-summary.md)**: one page, non-technical: what an
  attacker could achieve and the top fixes.
- **[`methodology.md`](methodology.md)**: how the engagement is structured (PTES + ATT&CK).
- **[`remediation.md`](remediation.md)**: the concrete fix and hardening control for
  every misconfiguration I exploited.
- **[`docs/reviewer-path.md`](docs/reviewer-path.md)**: a short guide to this repo.

## Skills demonstrated

AD enumeration · Kerberos attack tradecraft (Kerberoasting, AS-REP roasting) ·
credential dumping via replication (DCSync) · offline hash cracking · BloodHound
attack-path analysis · MITRE ATT&CK mapping · defensive remediation framing ·
professional engagement reporting.
