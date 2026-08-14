# Endpoint Security

## Overview

**Endpoint Security** refers to the collective set of protections applied to a device to reduce risk of compromise, including antivirus, EDR, encryption, and patch management.

Endpoint Security posture is a foundational input to [[Device Posture]] evaluations across [[ZCC]], [[ZIA]], and [[ZPA]].

Related:

- [[Device Posture]]
- [[EDR]]
- [[Zero Trust]]

---

# Purpose

Reduce the likelihood and impact of endpoint compromise by combining multiple layers of protection.

---

# Core Components

## Antivirus / Anti-Malware

Detects and blocks known malicious software.

## EDR

Detects and responds to advanced threats and suspicious behavior. See [[EDR]].

## Disk Encryption

Protects data at rest if a device is lost or stolen (BitLocker, FileVault, LUKS).

## Patch Management

Keeps operating systems and applications current against known vulnerabilities.

## Device Management (MDM/UEM)

Ensures devices are enrolled, configured, and compliant with organizational policy.

---

# Endpoint Security Stack

```text
Application Layer
      │
      ▼
EDR / Antivirus
      │
      ▼
Operating System
      │
      ▼
Disk Encryption
      │
      ▼
Hardware / Firmware
```

---

# Relationship to Device Posture

[[Device Posture]] evaluates the current state of these endpoint security controls before making access decisions, rather than assuming they are always active.

```text
Endpoint Security Controls
           │
           ▼
Device Posture Assessment
           │
           ▼
Access Decision
```

---

# Common Gaps

- Antivirus installed but disabled
- EDR agent disconnected
- Encryption not enabled on all volumes
- Devices missing critical patches

---

# Best Practices

- Layer multiple controls rather than relying on one
- Continuously monitor control health, not just at enrollment
- Automate remediation for common failures
- Align endpoint security requirements with data sensitivity

---

# Related Notes

- [[Device Posture]]
- [[EDR]]
- [[Zero Trust]]
- [[ZCC]]
