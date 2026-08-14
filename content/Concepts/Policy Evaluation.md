# Policy Evaluation

## Overview

**Policy Evaluation** is the process Zscaler uses to determine whether a request should be allowed, blocked, inspected, isolated, or restricted.

Policy decisions are based on:

- User identity
    
- Device posture
    
- Location
    
- Application
    
- Risk
    
- Security policy
    

---

# Zero Trust Decision Model

```text
Identity
   +
Device Posture
   +
Context
   +
Application
   +
Policy
   =
Access Decision
```

---

# Evaluation Flow

```text
Request
   │
   ▼
Identify User
   │
   ▼
Identify Device
   │
   ▼
Determine Context
   │
   ▼
Match Policy
   │
   ▼
Apply Action
```

---

# Policy Inputs

## User Identity

Examples:

- User account
    
- Group membership
    
- Department
    

Related:

- [[Identity Providers]]
    

---

## Device Posture

Examples:

- Managed device
    
- Encryption status
    
- EDR status
    

Related:

- [[Device Posture]]
    

---

## Location

Examples:

- Corporate office
    
- Remote user
    
- Country
    

---

## Application

Examples:

- SaaS application
    
- Private application
    
- Website
    

---

# ZIA Policy Evaluation

Common policy areas:

- URL Filtering
    
- Cloud Firewall
    
- DLP
    
- Sandbox
    
- Browser Isolation
    

---

# ZPA Policy Evaluation

Common decision factors:

- User
    
- Device
    
- Application
    
- Posture
    

---

# Actions

Possible outcomes:

|Action|Result|
|---|---|
|Allow|Permit access|
|Block|Deny access|
|Inspect|Perform security inspection|
|Isolate|Remote browser session|
|Restrict|Limited access|

---

# Policy Troubleshooting

## User Unexpectedly Blocked

Check:

1. Identity
    
2. Group membership
    
3. Policy order
    
4. Device posture
    
5. Application match
    

---

# Best Practices

- Use least privilege
    
- Keep policies simple
    
- Document exceptions
    
- Review regularly
    

---

# Related Notes

- [[ZIA]]
    
- [[ZPA]]
    
- [[Device Posture]]
    
- [[Identity Providers]]
    
- [[Access Policies]]