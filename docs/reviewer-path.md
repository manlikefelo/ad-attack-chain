# Reviewer path — a 3-minute tour

Short on time? Here's the fastest route through this repo, in the order that best
shows how I think.

### 1. The attack-path picture (30 sec)
Open **[Phase 04 — BloodHound](../engagement/04-bloodhound/)** and look at
`evidence/04-attack-paths-from-graph.png`. That one image shows the whole problem:
every misconfiguration as an edge leading to Domain Admin, all funnelling through one
account (`b.otieno`). It's the "why this matters" in a single graph.

### 2. The executive summary (60 sec)
Read **[`executive-summary.md`](../executive-summary.md)**. It's deliberately
non-technical — it's how I'd explain the risk and the fixes to a manager. If you only
read one document, read this one: it shows I can translate technical findings into
business impact.

### 3. One technique, end to end (90 sec)
Open **[Phase 03 — DCSync](../engagement/03-dcsync/)**. It's the sharpest step: a
helpdesk-tier account replicating every password hash in the domain, including
`krbtgt`. Skim:
- **§1 Scenario** — why over-granted replication rights are catastrophic.
- **§2 Attack** — the two `secretsdump` commands.
- **`evidence/02-dcsync-full-domain-dump.png`** — the actual dump.
- **§5 Remediation** — the exact fix and the ACL audit query.

### 4. If you want the defender's side
This whole engagement was **detected**. Head to the companion
**[detection-engineering](../../detection-engineering)** repo to see each of these
attacks caught by a Wazuh rule, verified firing end-to-end. Attack here → detect
there.

---

**What this repo proves:** I can enumerate and compromise an Active Directory domain
using real attacker tradecraft, map every step to MITRE ATT&CK, *and* explain the fix
in both technical and business terms. That last part — the remediation and the
executive framing — is what separates a script-runner from someone you'd hire.
