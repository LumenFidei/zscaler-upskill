# Service Edge

## Overview

A **Zscaler Service Edge** is a cloud-based enforcement point where Zscaler security services inspect traffic, apply policy, and provide secure connectivity.

Service Edges are the foundation of:

- [[ZIA]]
    
- [[ZPA]]
    
- [[Traffic Forwarding]]
    

---

# Architecture

```text
User
 │
 ▼
ZCC / Tunnel / Branch Connector
 │
 ▼
Zscaler Service Edge
 │
 ▼
Destination
```

---

# Responsibilities

## Traffic Inspection

Analyzes:

- Web traffic
    
- Application traffic
    
- DNS requests
    
- Network flows
    

---

## Policy Enforcement

Applies:

- URL filtering
    
- Firewall rules
    
- DLP policies
    
- Access policies
    

---

## Connectivity Brokerage

For ZPA:

- Connects users
    
- Connects App Connectors
    
- Creates secure sessions
    

---

# Service Edge Selection

Factors include:

- Geographic location
    
- Performance
    
- Availability
    
- Capacity
    

---

# ZIA Service Flow

```text
User
 │
 ▼
Service Edge
 │
 ├── URL Filtering
 ├── SSL Inspection
 ├── DLP
 └── Threat Protection
```

---

# ZPA Service Flow

```text
User
 │
 ▼
Service Edge
 │
 ▼
App Connector
 │
 ▼
Application
```

---

# Monitoring

Monitor:

- Latency
    
- Availability
    
- Throughput
    
- User experience
    

Related:

- [[ZDX]]
    

---

# Troubleshooting

## High Latency

Check:

1. Service Edge selection
    
2. ISP performance
    
3. Endpoint network
    
4. Application performance
    

---

# Best Practices

- Use nearest service edges
    
- Monitor experience metrics
    
- Understand regional behavior
    

---

# Related Notes

- [[ZIA]]
    
- [[ZPA]]
    
- [[ZDX]]
    
- [[Traffic Forwarding]]