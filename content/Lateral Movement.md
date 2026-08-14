# Lateral Movement

## Overview

**Lateral Movement** describes techniques attackers use to move through an environment after gaining an initial foothold, seeking additional systems, credentials, and privileges.

Detecting lateral movement is a primary objective of [[Deception]] and a common focus of [[Threat Hunting]].

Related:

- [[Deception]]
- [[Threat Hunting]]
- [[Credential Theft]]
- [[Zero Trust]]

---

# Why Lateral Movement Matters

Initial access is rarely the attacker's final objective. Attackers typically need to move laterally to reach:

- Domain controllers
- File servers
- Privileged accounts
- High-value applications

---

# Common Techniques

## Network Scanning

Enumerating hosts and services (e.g. SMB scanning) to find additional targets.

## Remote Service Abuse

Using RDP, SSH, WMI, or PsExec-style tools to access other systems.

## Credential Reuse

Leveraging harvested credentials across multiple systems. See [[Credential Theft]].

## Pass-the-Hash / Pass-the-Ticket

Reusing authentication material without needing plaintext credentials.

---

# Detection Approaches

```text
Attacker Foothold
        │
        ▼
Reconnaissance / Scanning
        │
        ▼
Credential Use on New Host
        │
        ▼
Detection Trigger
```

## Deception-Based Detection

Decoy hosts and breadcrumb credentials are designed to be discovered only during lateral movement attempts, making any interaction highly suspicious. See [[Deception]].

## Behavioral Detection

Unusual authentication patterns, such as a user account authenticating to systems it has never accessed before.

---

# Zero Trust as Mitigation

[[Zero Trust]] architectures reduce lateral movement risk by removing flat network access and enforcing application-level segmentation, so a compromised credential does not grant broad network reach.

---

# Investigation Checklist

- Which host initiated the movement?
- Which credentials were used?
- Which systems were accessed or attempted?
- Was privilege escalation involved?

---

# Best Practices

- Segment networks and applications to limit blast radius
- Deploy decoys across trust zones
- Monitor for unusual authentication patterns
- Restrict administrative protocol usage (RDP/SMB) between peer workstations

---

# Related Notes

- [[Deception]]
- [[Threat Hunting]]
- [[Credential Theft]]
- [[Zero Trust]]
- [[SOC Workflow]]
