# Zscaler Client Connector macOS Internals

## Overview

ZCC on macOS integrates with Apple's security and networking frameworks to provide:

- Traffic forwarding
    
- Zero Trust access
    
- Device posture validation
    
- Secure connectivity
    

Related:

- [[ZCC]]
    
- [[ZPA]]
    
- [[ZIA]]
    

---

# macOS Architecture

```text
macOS Applications
        |
        ▼
ZCC Application
        |
        ▼
Network Extensions
        |
        ▼
Zscaler Cloud
```

---

# macOS Components

## Network Extensions

Modern macOS versions rely on:

- Network Extension Framework
    
- System extensions
    
- Content filtering components
    

---

# Permissions

Deployment requires approval for:

- Network extensions
    
- System extensions
    
- Certificates
    
- Login items
    

---

# Deployment Methods

Common tools:

- Jamf Pro
    
- Microsoft Intune
    
- Kandji
    
- Workspace ONE
    

---

# Configuration

Configuration commonly uses:

- Configuration profiles
    
- Plist settings
    
- MDM payloads
    

---

# Plist Configuration

Typical categories:

- Authentication
    
- Logging
    
- Tunnel settings
    
- UI behavior
    

---

# Certificate Deployment

Required for:

- SSL inspection
    
- Authentication
    
- Trust relationships
    

Deployment methods:

- MDM profiles
    
- Certificate payloads
    

---

# Tunnel Behavior

```text
User Login
    |
    ▼
ZCC Launch
    |
    ▼
Authentication
    |
    ▼
Network Extension Activation
    |
    ▼
Traffic Forwarding
```

---

# Troubleshooting

## Extension Not Loading

Check:

- System extension approval
    
- MDM profile
    
- macOS permissions
    

---

## SSL Errors

Check:

- Certificate trust
    
- Inspection policy
    
- Application compatibility
    

---

# Best Practices

- Deploy with MDM
    
- Pre-stage certificates
    
- Validate OS compatibility
    
- Maintain configuration profiles
    

---

# Related Notes

- [[ZCC]]
    
- [[SSL Inspection]]
    
- [[Traffic Forwarding]]
    
- [[Device Posture]]