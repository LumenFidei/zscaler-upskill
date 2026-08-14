# Conditional Access

## Overview

**Conditional Access** is the practice of granting or restricting access based on a combination of dynamic conditions — such as identity, device posture, location, and risk — rather than a static allow/deny decision.

Conditional Access principles underpin both [[Policy Evaluation]] in Zscaler and identity-layer controls enforced by [[Identity Providers]].

Related:

- [[Policy Evaluation]]
- [[Device Posture]]
- [[Identity Providers]]
- [[Zero Trust]]

---

# Purpose

Move beyond simple username/password gating to access decisions that account for real-time context.

---

# Common Conditions

## Identity

- User
- Group membership
- Role

## Device

- Managed status
- Compliance/posture state

## Location

- Country
- Corporate network vs. remote

## Risk

- Sign-in risk score
- Impossible travel detection

---

# Conditional Access Flow

```text
Access Request
      │
      ▼
Evaluate Identity
      │
      ▼
Evaluate Device
      │
      ▼
Evaluate Location / Risk
      │
      ▼
Policy Decision
      │
 ┌────┴────┐
 │         │
Allow   Deny / Step-Up
```

---

# Where Conditional Access Applies in Zscaler

## Identity Provider Layer

IdPs such as Microsoft Entra ID or Okta can enforce conditional access before a user ever reaches Zscaler.

## ZIA / ZPA Policy Layer

Zscaler's own [[Policy Evaluation]] model applies similar logic using identity, device posture, and location as inputs.

---

# Common Outcomes

|Condition|Possible Outcome|
|---|---|
|Unmanaged device|Restrict to browser-only access|
|High-risk sign-in|Require step-up MFA|
|Non-compliant posture|Deny or quarantine|
|Trusted network + managed device|Full access|

---

# Best Practices

- Coordinate conditional access rules between the IdP and Zscaler policy layers to avoid conflicts
- Start with monitoring/report-only mode before enforcing
- Apply stricter conditions to higher-risk applications
- Review conditional access policies alongside access policy reviews

---

# Related Notes

- [[Policy Evaluation]]
- [[Device Posture]]
- [[Identity Providers]]
- [[Zero Trust]]
- [[Access Policies]]
