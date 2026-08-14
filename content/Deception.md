# Zscaler Deception

## Overview

**Zscaler Deception** is a cyber deception platform designed to detect attacker activity by deploying decoys, lures, and traps throughout an environment.

The platform operates on the principle that legitimate users should never interact with deception assets.

Any interaction is considered highly suspicious and warrants investigation.

---

# Objectives

- Detect lateral movement
    
- Detect credential theft
    
- Detect insider threats
    
- Detect ransomware activity
    
- Reduce dwell time
    

---

# Architecture

```text
Endpoints
    │
    ▼
Lures
    │
    ▼
Decoys
    │
    ▼
Detection Engine
    │
    ▼
Security Team
```

---

# Core Components

## Lures

Artifacts designed to attract attackers.

Examples:

- Browser credentials
    
- SSH keys
    
- RDP files
    
- Network shares
    

---

## Decoys

Simulated assets.

Examples:

- Servers
    
- Databases
    
- Domain Controllers
    
- Applications
    

---

## Breadcrumbs

Fake information planted throughout the environment.

Examples:

- Registry entries
    
- Files
    
- Credentials
    
- URLs
    

---

# Threat Detection

## Credential Theft

Detects:

- Password harvesting
    
- Token theft
    
- Browser credential access
    

---

## Lateral Movement

Detects:

- SMB scanning
    
- RDP attempts
    
- Administrative movement
    

---

## Ransomware Detection

Detects:

- Discovery activity
    
- Enumeration
    
- Privilege escalation
    

---

# Alert Workflow

```text
Attacker Activity
        │
        ▼
Decoy Interaction
        │
        ▼
Alert Generated
        │
        ▼
SOC Investigation
```

---

# Deployment Locations

## Endpoints

Deploy lures directly onto devices.

## Data Centers

Deploy server decoys.

## Cloud Environments

Deploy cloud-hosted decoys.

---

# Investigation

## Key Questions

- Which lure was accessed?
    
- Which host initiated access?
    
- Which credentials were involved?
    
- Was lateral movement observed?
    

---

# Benefits

- High fidelity alerts
    
- Low false positives
    
- Early attacker detection
    
- Visibility into attacker behavior
    

---

# Best Practices

- Deploy across all trust zones
    
- Regularly refresh lures
    
- Monitor privileged systems
    
- Integrate with SIEM platforms
    

---

# Related Notes

- [[Threat Hunting]]
    
- [[Lateral Movement]]
    
- [[Credential Theft]]
    
- [[SOC Workflow]]