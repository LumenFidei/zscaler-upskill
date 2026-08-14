# Identity Providers

## Overview

An **Identity Provider (IdP)** authenticates users and provides identity information used by Zscaler services to make access decisions.

Identity is a foundational component of:

- [[ZCC]]
    
- [[ZIA]]
    
- [[ZPA]]
    
- [[Policy Evaluation]]
    

---

# Authentication Flow

```text
User
 │
 ▼
Zscaler
 │
 ▼
Identity Provider
 │
 ▼
Authentication Result
 │
 ▼
Policy Evaluation
```

---

# Common Identity Providers

## Microsoft Entra ID

Common enterprise cloud identity platform.

Used for:

- SSO
    
- MFA
    
- Conditional access
    

---

## Okta

Cloud identity provider.

Used for:

- Federation
    
- User lifecycle
    
- MFA
    

---

## Ping Identity

Enterprise federation platform.

---

## Active Directory Federation Services

Legacy federation solution.

---

# Authentication Methods

## SAML

Most common enterprise method.

Provides:

- Identity assertion
    
- Group information
    
- Federation
    

---

## Certificate Authentication

Uses certificates to verify identity.

---

## MFA

Adds additional verification.

Examples:

- Push approval
    
- Hardware token
    
- Authenticator apps
    

---

# Identity Attributes

Common attributes:

- Username
    
- Email
    
- Group membership
    
- Department
    
- Role
    

---

# Identity in ZPA

Used for:

- Application access
    
- Policy matching
    
- Least privilege enforcement
    

---

# Identity in ZIA

Used for:

- Internet policy
    
- Reporting
    
- User-based controls
    

---

# Troubleshooting

## Authentication Failure

Check:

1. IdP availability
    
2. SAML configuration
    
3. Certificate validity
    
4. User attributes
    
5. Group mapping
    

---

# Best Practices

- Centralize identity
    
- Require MFA
    
- Keep groups clean
    
- Audit access regularly
    

---

# Related Notes

- [[Authentication]]
    
- [[ZPA]]
    
- [[ZIA]]
    
- [[Device Posture]]
    
- [[Policy Evaluation]]