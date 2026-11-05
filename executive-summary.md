# Executive Summary

*Written for a non-technical reader: a manager or business owner, not an engineer.*

## What I tested

With full authorization, I acted as an attacker against a self-owned test copy of a
Windows **Active Directory** network: the system that controls logins, permissions,
and access across a typical corporate environment. The goal was to see how far an
attacker could get and how quickly.

## What I found

**An attacker starting with *no access at all* could take complete control of the
entire network.** Not "read a few files", full control: the ability to log in as
anyone, read any data, and stay in undetected more or less permanently.

The path took four steps, each abusing a common real-world misconfiguration:

1. **Free password, no login required.** One account was set up so the system would
   hand out its scrambled password to anyone who asked: no credentials needed. I
   recovered the real password in under a second because it was weak.
2. **That account was over-powered.** It turned out to be a high-privilege account
   that should never have had that level of access: a "temporary admin" grant that
   was never removed.
3. **Copy every password in the company.** Using that account, I made the network
   duplicate its entire password database to me, including the master key that
   underpins all authentication.
4. **Permanent control.** With that master key, an attacker can forge access passes
   for any user indefinitely: the equivalent of a master key to every door that
   still works even after the locks are changed.

Separately, a **service account** (the kind that runs background software) was also
crackable, and a **mapping tool** confirmed every one of these weaknesses as a clear
route to total takeover.

## Why it matters to the business

These are not exotic, movie-hacker techniques. They are the **most common ways real
Active Directory networks get compromised**, and they rely on ordinary
configuration mistakes, not sophisticated exploits. An attacker who got a single
foothold (a phished employee, say) could realistically repeat this. Full domain
control means ransomware deployment, data theft, and business shutdown are all on the
table.

## The good news

**Every weakness has a straightforward fix, and most cost nothing but time.** Three
of the four problems live on a *single account*, cleaning up that one account
removes most of the risk. My full [remediation plan](remediation.md) lists the exact
fixes in priority order:

- Remove admin rights that were never meant to be permanent.
- Turn a security feature (login pre-authentication) back on where it was switched off.
- Give service accounts strong, machine-managed passwords.
- Rotate the master key so any keys an attacker copied stop working.

## Bottom line

The network was fully compromisable from a standing start, but the fixes are cheap,
well-understood, and can be completed quickly. Prioritising the single over-powered
account closes most of the exposure immediately.
