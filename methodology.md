# Methodology

This engagement follows a recognized structure so the work reads like a real
assessment rather than a pile of tool output. I use **PTES** (Penetration Testing
Execution Standard) for the phase flow and **MITRE ATT&CK** to label every
individual action.

## Why two frameworks

- **PTES gives the narrative arc**: the phases an engagement moves through, from
  scoping to reporting. It's what a reader uses to orient: "where in the assessment
  am I?"
- **ATT&CK gives the vocabulary**: a stable, industry-standard ID for each
  attacker behaviour. It's what a blue-teamer uses to map an attack to a detection,
  and it's how my companion [detection-engineering](https://github.com/manlikefelo/detection-engineering) repo
  lines its rules up against this repo's attacks one-for-one.

Using both means the same write-up is legible to a pentest lead *and* to a SOC
analyst, which is exactly the purple-team framing I'm going for.

## PTES phases, mapped to this engagement

| PTES phase | What I did here |
|------------|-----------------|
| **1. Pre-engagement / scope** | Defined the target as my own isolated `lab.local` domain. Authorization is self-owned; scope is DC01 only. Stated in every README. |
| **2. Intelligence gathering** | Nmap service scan + DNS/SMB/LDAP checks to confirm reach to the DC; BloodHound collection (Phase 04) to map the domain graph. |
| **3. Threat modelling** | Read the BloodHound graph to identify the shortest paths to Domain Admin and which principals hold dangerous rights (DCSync, DA membership). |
| **4. Vulnerability analysis** | Identified the four planted weaknesses: a Kerberoastable SPN account, a pre-auth-disabled account, over-granted replication rights, and a stale Domain Admins membership. |
| **5. Exploitation** | Kerberoasting (01), AS-REP roasting (02), DCSync (03): each recovering credentials or hashes. |
| **6. Post-exploitation** | krbtgt hash recovered → Golden Ticket capability (full, persistent domain control). Proof-of-compromise captured, sanitized. |
| **7. Reporting** | This repo: per-phase technical write-ups, an [executive summary](executive-summary.md), a [remediation plan](remediation.md), and the ATT&CK mapping. |

## How each phase is documented

Every folder under [`/engagement`](engagement/) uses the same shape so a reviewer can
compare them quickly:

1. **Scenario**: what the weakness is and why it matters in the real world.
2. **Attack**: the exact commands from KALI01, with expected output.
3. **Telemetry**: the Windows/Sysmon events the attack generates (the bridge to
   detection).
4. **Detection**: how the blue side catches it (pointer to the Wazuh rule).
5. **Remediation**: how a defender fixes the underlying misconfiguration.
6. **Evidence**: real screenshots proving it worked end-to-end.
7. **ATT&CK coverage matrix**: the per-phase technique-by-technique mapping.

Each phase also has a **`RUN-LOG.md`**: the actual execution record (dates,
real command output, the gotchas I hit) as distinct from the polished README.

## A note on honesty

Where a tool didn't behave as the textbook says, I documented it rather than hiding
it, e.g. this GPU-less host couldn't run hashcat's OpenCL backend, so I cracked
with John the Ripper on CPU and noted the canonical hashcat command alongside. The
point of the lab is to show I understand *why* each step works, not to fake a clean
run.
