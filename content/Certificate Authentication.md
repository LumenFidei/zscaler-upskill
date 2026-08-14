# Certificate Authentication

## Overview

**Certificate Authentication** uses digital certificates to verify the identity of a device or user before granting access to Zscaler-protected resources.

Certificate checks are commonly used as a [[Device Posture]] signal and as a strong-trust component within [[Authentication]].

Related:

- [[Device Posture]]
- [[Authentication]]
- [[Identity Providers]]

---

# Purpose

Certificates provide strong, hard-to-spoof identity assurance compared to username/password alone.

---

# Certificate Types

## Machine Certificates

Identify the device itself, independent of who is logged in.

## User Certificates

Identify the specific user, often issued via a corporate PKI or MDM platform.

---

# How Certificate Authentication Works

```text
Device / User
      │
      ▼
Present Certificate
      │
      ▼
Validate Against Trusted CA
      │
      ▼
Check Revocation Status
      │
      ▼
Identity Confirmed
```

---

# Common Deployment Methods

- Microsoft Intune certificate profiles
- Jamf certificate payloads
- Group Policy autoenrollment
- Manual PKI issuance

---

# Use Cases

- Validating managed device identity before ZPA access
- Strengthening privileged access requirements
- Reducing reliance on password-only authentication
- Supporting device-bound trust in BYOD-restricted environments

---

# Integration with Device Posture

Certificate presence and validity is commonly one of several checks in a posture profile, alongside encryption and EDR status. See [[Device Posture]].

---

# Troubleshooting

## Certificate Not Presented

Check:

1. Certificate deployment status (MDM/GPO)
2. Certificate store location
3. Application/client certificate selection behavior

## Certificate Rejected

Check:

1. Certificate expiration
2. Trust chain to a recognized CA
3. Revocation status
4. Certificate template/attribute mismatch

---

# Best Practices

- Automate certificate deployment and renewal via MDM
- Monitor certificate expiration proactively
- Use machine certificates for device trust and user certificates for identity where both are needed
- Maintain a clear revocation process for lost or decommissioned devices

---

# Related Notes

- [[Device Posture]]
- [[Authentication]]
- [[Identity Providers]]
- [[Zero Trust]]
