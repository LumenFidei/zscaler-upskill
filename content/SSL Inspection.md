# SSL Inspection

## Overview

**SSL Inspection** allows Zscaler to decrypt encrypted traffic, inspect content, enforce security policies, and re-encrypt traffic.

SSL inspection is a major security capability within:

- [[ZIA]]
    
- [[Policy Evaluation]]
    

---

# Why SSL Inspection Exists

Modern traffic is encrypted:

```text
HTTPS Traffic
      │
      ▼
Encrypted Content
```

Without inspection:

- Malware can hide
    
- Data leakage can occur
    
- Threats bypass controls
    

---

# SSL Inspection Flow

```text
Client
 │
 ▼
Zscaler
 │
 ▼
Decrypt
 │
 ▼
Inspect
 │
 ▼
Re-encrypt
 │
 ▼
Destination
```

---

# Certificate Requirements

Endpoints must trust the Zscaler certificate authority.

Common deployment:

- Group Policy
    
- MDM
    
- Intune
    
- Jamf
    

---

# Inspection Policies

## Full Inspection

Inspect all traffic.

---

## Selective Inspection

Inspect defined categories.

---

## No Inspection

Bypass inspection.

---

# Common Exceptions

Examples:

- Banking
    
- Healthcare
    
- Certificate-pinned applications
    

---

# Troubleshooting

## Certificate Error

Check:

1. Root certificate installed
    
2. Certificate trust
    
3. Application compatibility
    

---

# Best Practices

- Inspect as much traffic as practical
    
- Document exclusions
    
- Monitor failures
    

---

# Related Notes

- [[ZIA]]
    
- [[Traffic Forwarding]]
    
- [[Cloud Firewall]]