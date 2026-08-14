# Device Posture

## Overview

**Device Posture** evaluates the security state of an endpoint before and during access, adding device trust to identity-based decisions.

Related:

- [[Access Policies]]
- [[ZCC]]
- [[Zero Trust]]

---

> [!important] ZDTE exam weight
> Posture appears throughout the blueprint:
> - "Identify the steps to create access policies... incorporating **posture checks** (e.g., device posture, health score, AV, etc.)"
> - Least-privilege objectives explicitly name "device posture" as a policy input
> - Troubleshooting objectives on private-app blocks frequently resolve to posture failures
>
> Know the **check catalogue** and how posture attaches to policy.

---

# How Posture Attaches to Policy

Posture is not enforced on its own. The chain:

```text
1. ZCC collects posture signals from the endpoint
        ▼
2. Posture Profile defines required checks
        ▼
3. Access Policy rule references the posture profile
        ▼
4. Decision at connection + continuous re-evaluation
```

**Exam trap:** a posture profile that exists but isn't referenced by any Access Policy rule enforces nothing. If a scenario says "posture is configured but non-compliant devices still get in," check the rule binding — not the profile contents.

---

# Posture Check Catalogue

## Operating System
- OS type and version
- Build number
- Patch level

## Endpoint Protection
- AV installed / running / updated
- Specific vendors: Defender, CrowdStrike, SentinelOne, Carbon Black

## EDR
- Installed, running, **connected**, healthy

## Disk Encryption
- Windows: **BitLocker**
- macOS: **FileVault**
- Linux: **LUKS**

## Device Management
- Intune, Jamf, Workspace ONE enrollment
- Compliant / enrolled status
- Domain joined

## Certificates
- **Machine certificate** — identifies the device
- **User certificate** — identifies the user

## ZDX Health Score
Device experience score usable as a trust signal — a distinctive Zscaler capability worth knowing.

## Mobile
- Jailbreak / root detection
- MDM enrollment
- OS version

## Process / File / Registry
- Specific process running
- File or registry key present (useful for custom agent checks)

---

# Access Outcomes

```text
Fully compliant     → full access
Partially compliant → limited access, subset of apps, step-up auth
Non-compliant       → deny, quarantine, remediation page
```

Design pattern: rather than a binary allow/deny, tier access by posture level. A scenario asking how to "allow limited access to non-compliant devices" is answered with a separate lower-privilege rule matching a weaker posture profile.

---

# Posture Profiles by Population

## Standard Workforce
Managed device, encryption enabled, AV active

## Developers
Managed device, EDR active, approved OS

## Privileged Administrators
Managed + EDR + encryption + certificate + MFA

## Contractors
Supported OS, AV installed — or Browser Access with no posture requirement

---

# Continuous Verification

Trust is not permanent. Posture re-evaluates during the session:

```text
Session active
      │
      ▼
EDR agent stops / encryption disabled
      │
      ▼
Posture failure
      │
      ▼
Access revoked or restricted mid-session
```

This is a differentiator versus VPN, where trust is established once at connect time. Expect it as a conceptual question.

---

# Troubleshooting Posture Failures

## Device Fails Assessment

Identify **which specific check** failed — the console reports per-check results. Common causes:

|Symptom|Likely check|
|---|---|
|Recently reimaged device denied|Certificate missing, not yet MDM-enrolled|
|Intermittent denial|EDR disconnected from its console|
|One OS version affected|OS version/build threshold|
|Denied after security tool update|AV/EDR process name or version changed|
|Works on some devices only|Encryption not enabled on all volumes|

## Workflow

```text
Posture failure
      │
      ▼
Identify failed check (console/logs)
      │
      ▼
Validate actual endpoint state
      │
      ▼
Remediate
      │
      ▼
Retest — confirm re-evaluation
```

## Denied Despite Passing Posture
Then it isn't posture — check Access Policy rule order, group membership, and segment assignment. See [[Access Policies]].

---

# Design Best Practices

- **Test in monitoring mode before enforcing** — the single most-cited best practice
- Start simple: managed + encrypted + AV
- Add EDR, certificates, browser posture gradually
- Keep requirements achievable — unrealistic checks cause mass lockouts
- Separate populations into distinct profiles
- Plan a remediation path, not just a denial

---

# Key Takeaways

1. Identity alone is insufficient for trust.
2. Posture profiles enforce nothing until referenced by an Access Policy rule.
3. Common checks: OS, AV, EDR, encryption, management, certificates, ZDX score.
4. Tier access by posture level rather than binary allow/deny.
5. Continuous re-evaluation revokes access when posture degrades mid-session.
6. Always pilot in monitor mode first.

---

# Related Notes

- [[Access Policies]]
- [[ZPA]]
- [[ZCC]]
- [[EDR]]
- [[Certificate Authentication]]
- [[Zero Trust]]
