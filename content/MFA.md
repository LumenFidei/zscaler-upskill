# MFA (Multi-Factor Authentication)

## Overview

**Multi-Factor Authentication (MFA)** requires users to verify their identity using more than one factor, reducing the risk that a stolen password alone grants access.

MFA is a core component of [[Authentication]] and a common requirement within [[Device Posture]] and [[ZPA Policy Design]] for sensitive access.

Related:

- [[Authentication]]
- [[Identity Providers]]
- [[Device Posture]]
- [[Zero Trust]]

---

# Purpose

Reduce account takeover risk by requiring proof of identity beyond a single credential.

---

# Common Factors

## Something You Know

- Password
- PIN

## Something You Have

- Hardware token
- Authenticator app
- Push notification device

## Something You Are

- Biometric verification (fingerprint, face)

---

# MFA in the Zscaler Authentication Flow

```text
User
 │
 ▼
Primary Authentication (Password/SSO)
 │
 ▼
MFA Challenge
 │
 ▼
MFA Verified
 │
 ▼
Policy Evaluation
 │
 ▼
Access Granted
```

MFA is typically enforced by the [[Identity Providers|Identity Provider]] before Zscaler evaluates access policy.

---

# Common MFA Methods

- Push approval
- Hardware token (e.g. FIDO2 key)
- Authenticator applications (TOTP)
- SMS/voice (legacy, lower assurance)

---

# Where MFA Is Commonly Required

- Initial ZCC authentication
- Privileged application access in [[ZPA]]
- Administrative access to the Zscaler admin portal
- Step-up authentication for high-risk actions

---

# Troubleshooting

## MFA Not Prompting

Check:

1. IdP MFA policy configuration
2. App Profile authentication settings
3. Conditional access rule scope

## MFA Failing

Check:

1. Time synchronization (for TOTP)
2. Device enrollment for push/token
3. IdP service status

---

# Best Practices

- Require MFA for all users, not just privileged accounts
- Prefer phishing-resistant methods (FIDO2) over SMS
- Enforce step-up MFA for sensitive application segments
- Monitor MFA failure patterns for signs of attack

---

# Related Notes

- [[Authentication]]
- [[Identity Providers]]
- [[Device Posture]]
- [[Zero Trust]]
- [[Access Policies]]
