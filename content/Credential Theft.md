# Credential Theft

## Overview

**Credential Theft** refers to techniques attackers use to obtain valid usernames, passwords, tokens, or certificates in order to impersonate legitimate users.

Stolen credentials are a common enabler of [[Lateral Movement]] and a primary detection target for [[Deception]].

Related:

- [[Deception]]
- [[Lateral Movement]]
- [[Identity Providers]]
- [[Authentication]]

---

# Why Credential Theft Matters

Valid credentials allow attackers to blend in with normal user activity, making detection harder than malware-based intrusions.

---

# Common Techniques

## Password Harvesting

Extracting stored or cached passwords from browsers, files, or memory.

## Token Theft

Stealing session or authentication tokens to bypass the need for a password.

## Browser Credential Access

Reading saved credentials from browser credential stores.

## Phishing

Tricking users into submitting credentials to attacker-controlled sites.

## Keylogging

Capturing credentials as they are typed.

---

# Detection Approaches

## Deception-Based Detection

Planted lure credentials (browser credentials, RDP files, SSH keys) are never used by legitimate processes, so any use is a strong compromise indicator. See [[Deception]].

## Identity-Based Detection

Unusual authentication patterns, such as impossible travel or logins from unexpected devices, monitored via [[Identity Providers]].

## Endpoint-Based Detection

Monitoring access to credential stores (e.g. LSASS memory access on Windows).

---

# Impact if Successful

```text
Credential Theft
       │
       ▼
Impersonation
       │
       ▼
Lateral Movement / Privilege Escalation
       │
       ▼
Objective (Data Access, Ransomware, etc.)
```

---

# Mitigation

- Enforce MFA to reduce the value of stolen passwords
- Use short-lived tokens where possible
- Apply least-privilege access via [[Zero Trust]] principles
- Monitor for credential store access attempts

---

# Investigation Checklist

- Which credential was involved?
- Was it a lure/breadcrumb or a real credential?
- Which host or account showed the activity?
- Has the credential been used elsewhere?

---

# Related Notes

- [[Deception]]
- [[Lateral Movement]]
- [[Identity Providers]]
- [[Authentication]]
- [[SOC Workflow]]
