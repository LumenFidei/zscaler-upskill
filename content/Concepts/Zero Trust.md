# Zero Trust

## Overview

**Zero Trust** is a security model based on the principle:

> Never trust, always verify.

Access decisions are continuously evaluated based on identity, device, context, and policy.

---

# Traditional Security Model

```text
User
 │
 ▼
Network Access
 │
 ▼
Applications
```

Assumption:

> Being inside the network means being trusted.

---

# Zero Trust Model

```text
Identity
 +
Device
 +
Context
 +
Policy
 =
Access Decision
```

---

# Core Principles

## Verify Explicitly

Validate:

- User
    
- Device
    
- Location
    
- Application
    

---

## Least Privilege

Grant only required access.

---

## Assume Breach

Design as if attackers already have access.

---

# Zscaler Zero Trust Architecture

```text
User
 │
 ▼
Identity
 │
 ▼
Device Posture
 │
 ▼
Policy
 │
 ▼
Application
```

---

# ZIA Role

Provides:

- Secure internet access
    
- Threat prevention
    
- Data protection
    

---

# ZPA Role

Provides:

- Application-level access
    
- VPN replacement
    
- Segmentation
    

---

# ZCC Role

Provides:

- Endpoint enforcement
    
- Secure connectivity
    

---

# Key Concepts

## Identity-Based Access

Users receive access based on who they are.

---

## Device Trust

Devices must meet requirements.

Related:

- [[Device Posture]]
    

---

## Continuous Verification

Trust changes over time.

---

# Best Practices

- Use MFA
    
- Validate device posture
    
- Minimize access
    
- Monitor continuously
    

---

# Related Notes

- [[ZIA]]
    
- [[ZPA]]
    
- [[ZCC]]
    
- [[Policy Evaluation]]
    
- [[Device Posture]]