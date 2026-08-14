# Authentication

## Overview

**Authentication** is the process of verifying the identity of a user, device, or service before granting access to Zscaler-protected resources.

Authentication is a foundational component of:

- [[ZCC]]
    
- [[ZIA]]
    
- [[ZPA]]
    
- [[Identity Providers]]
    
- [[Policy Evaluation]]
    

---

# Authentication vs Authorization

## Authentication

Answers:

> Who are you?

Examples:

- Username
    
- Certificate
    
- MFA verification
    

---

## Authorization

Answers:

> What are you allowed to access?

Examples:

- Applications
    
- Internet categories
    
- Security policies
    

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

Provides user identity.

Examples:

- Microsoft Entra ID
    
- Okta
    
- Ping Identity
    
- ADFS
    

Related:

- [[Identity Providers]]
    

---

## Federation

Allows Zscaler to trust an external identity system.

Common protocol:

- SAML
    

---

## Multi-Factor Authentication

Adds additional verification.

Examples:

- Push notification
    
- Hardware token
    
- Authenticator application
    

---

# ZCC Authentication

## Purpose

Authenticate users before enabling secure forwarding.

Process:

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

Used before private application access.

Flow:

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

---

# Certificate Authentication

## Purpose

Validate device identity.

Common uses:

- Managed devices
    
- Privileged access
    
- Strong device trust
    

---

# Troubleshooting

## Authentication Failure

Check:

1. Identity provider status
    
2. SAML configuration
    
3. Certificate validity
    
4. User attributes
    
5. MFA configuration
    

---

# Best Practices

- Require MFA
    
- Use centralized identity
    
- Remove stale accounts
    
- Audit authentication events
    

---

# Related Notes

- [[Identity Providers]]
    
- [[Device Posture]]
    
- [[Policy Evaluation]]
    
- [[ZCC]]
    
- [[ZPA]]