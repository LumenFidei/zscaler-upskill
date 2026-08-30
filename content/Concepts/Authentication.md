# Authentication

## Overview

**Authentication** verifies the identity of a user, device, or service before granting access to Zscaler-protected resources.

Related:

- [[Identity Providers]]
- [[ZCC]]
- [[ZPA]]
- [[PRA]]
- [[Policy Evaluation]]

---

# Authentication vs Authorization

## Authentication
Answers *"who are you?"* — username, certificate, MFA verification.

## Authorization
Answers *"what are you allowed to access?"* — applications, internet categories, security policies. This is what [[Policy Evaluation]] governs, and it only happens **after** authentication succeeds.

**This ordering matters for troubleshooting:** a failure before the user reaches any policy decision is an authentication problem, not an access-policy problem — different layer, different fix. See [[Troubleshooting Methodology]].

---

# Authentication Flow

```text
User
 │
 ▼
Zscaler Client Connector
 │
 ▼
Identity Provider
 │
 ▼
Authentication Response
 │
 ▼
Policy Evaluation
 │
 ▼
Access Granted
```

---

# Authentication Components

## Identity Provider
Provides user identity. See [[Identity Providers]] for the full list and configuration detail.

## Federation
Allows Zscaler to trust an external identity system — commonly via SAML.

## Multi-Factor Authentication
Additional verification layer. See [[MFA]].

---

# How the Correct IdP Gets Selected

A mechanic that's easy to take for granted until it breaks: Zscaler determines which IdP to authenticate a user against based on the **domain portion of their username**. This works invisibly for the overwhelming majority of users, whose email domain matches their organization's registered authentication domain.

It becomes a real, documented problem for identities that don't cleanly match a registered domain — most commonly **guest/external users** in Entra ID, whose actual UserPrincipalName lives in a `#EXT#@tenant.onmicrosoft.com` format distinct from their real email. Full mechanics and the fix are in [[Identity Providers]] — worth knowing this exists even if you never touch guest-user configuration directly, since the resulting symptom (a generic authentication failure with no obvious cause) doesn't announce itself as a "guest user problem."

---

# ZCC Authentication

```text
ZCC Starts
    │
    ▼
User Authentication
    │
    ▼
Policy Download
    │
    ▼
Tunnel Established
```

---

# ZPA Authentication

```text
User Requests Application
          │
          ▼
Authentication
          │
          ▼
Posture Check
          │
          ▼
Access Policy
          │
          ▼
Application Connection
```

Note authentication happens **before** posture and access policy — a user who fails authentication never reaches a posture or policy decision at all. This same ordering applies to clientless access patterns like [[PRA]] and Browser Access.

---

# Certificate Authentication

Validates device or user identity using certificates rather than (or alongside) a password. See [[Certificate Authentication]] for machine vs. user certificate distinctions.

---

# Troubleshooting

## Authentication Failure — General Checklist
1. Identity provider status
2. SAML configuration
3. Certificate validity
4. User attributes
5. MFA configuration

## Generic Failure, No Obvious Cause, Affecting an External or Guest User
Don't burn time re-checking the general checklist from scratch — jump straight to the domain-based IdP selection mismatch described above and detailed fully in [[Identity Providers]]. This specific symptom pattern (works fine for regular users, fails for guests) is the signature to recognize.

## Auth Succeeds but Access Still Denied
That's not an authentication problem — authentication already passed. Move to [[Access Policies]] or [[Device Posture]] troubleshooting instead.

---

# Best Practices

- Require MFA
- Use centralized identity
- Remove stale accounts
- Audit authentication events
- Document non-obvious identity edge cases (like guest-user federation) where your team will actually find them again

---

# Related Notes

- [[Identity Providers]]
- [[Device Posture]]
- [[Policy Evaluation]]
- [[ZCC]]
- [[ZPA]]
- [[PRA]]
