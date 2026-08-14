# EDR (Endpoint Detection and Response)

## Overview

**EDR (Endpoint Detection and Response)** platforms monitor endpoint activity for signs of compromise and provide response capabilities such as isolation and remediation.

EDR presence and health status is a common signal evaluated by [[Device Posture]] before granting access.

Related:

- [[Device Posture]]
- [[Zero Trust]]
- [[Endpoint Security]]

---

# Purpose

- Detect malicious behavior on endpoints
- Provide investigation and response tooling
- Reduce dwell time for compromised devices

---

# Common Platforms

- CrowdStrike Falcon
- Microsoft Defender for Endpoint
- SentinelOne
- VMware Carbon Black

---

# Typical Posture Checks

```text
EDR Check
    │
    ▼
Installed?
    │
    ▼
Running?
    │
    ▼
Connected?
    │
    ▼
Healthy?
```

Each stage must pass for a device to be considered EDR-compliant.

---

# Role in Zero Trust

EDR health is one of several signals combined with identity, encryption, and management status to form an overall device trust decision. See [[Device Posture]] and [[Zero Trust]].

---

# Integration with ZPA and ZIA

- ZPA may deny or restrict access to sensitive application segments if EDR is not healthy
- ZIA may apply stricter internet policy for devices with degraded EDR status

---

# Common Failure Causes

- EDR agent not installed
- Agent installed but not running
- Agent running but disconnected from management console
- Outdated agent version

---

# Troubleshooting

## EDR Check Failing

Check:

1. Agent installation status
2. Agent process/service status
3. Connectivity to EDR management console
4. Agent version against minimum requirement

---

# Best Practices

- Require EDR for privileged and developer populations first
- Monitor EDR health continuously, not just at login
- Automate remediation workflows for common failures
- Align EDR requirements with organizational risk tolerance

---

# Related Notes

- [[Device Posture]]
- [[Zero Trust]]
- [[Endpoint Security]]
- [[ZPA]]
- [[ZIA]]
