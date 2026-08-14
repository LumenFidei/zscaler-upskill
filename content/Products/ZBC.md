# Zscaler Branch Connector

## Overview

**Branch Connector** is a virtual appliance used by Zscaler to connect branch offices, data centers, cloud environments, and private networks to the Zscaler cloud.

Branch Connector extends Zero Trust access and application connectivity beyond individual endpoints.

---

# Objectives

- Connect sites to ZPA
    
- Enable private application access
    
- Replace legacy VPN architectures
    
- Extend Zero Trust networking
    

---

# Architecture

```text
Branch Office
      │
      ▼
Branch Connector
      │
      ▼
Zscaler Cloud
      │
      ▼
Applications
```

---

# Core Functions

## Site Connectivity

Provides secure connectivity between:

- Branches
    
- Data centers
    
- Cloud environments
    

---

## Application Access

Provides access to:

- Internal applications
    
- Legacy applications
    
- Private services
    

---

## Traffic Segmentation

Enables:

- Application segmentation
    
- Site segmentation
    
- User segmentation
    

---

# Deployment Models

## Branch Office

```text
Users
  │
  ▼
Branch Connector
  │
  ▼
Zscaler Cloud
```

---

## Data Center

```text
Applications
      │
      ▼
Branch Connector
      │
      ▼
ZPA Cloud
```

---

## Cloud

Supported environments:

- AWS
    
- Azure
    
- Google Cloud
    

---

# Connectivity

## Outbound Connections

Branch Connector establishes outbound connectivity to Zscaler.

Benefits:

- No inbound firewall rules
    
- Simplified security
    

---

# High Availability

## Recommended Design

```text
Branch Connector A

Branch Connector B
```

Benefits:

- Failover
    
- Load sharing
    
- Maintenance flexibility
    

---

# Routing

## Common Integrations

- SD-WAN
    
- MPLS
    
- Internet circuits
    

---

# Application Discovery

Used to identify:

- Internal applications
    
- Legacy services
    
- Migration candidates
    

---

# Monitoring

## Health Metrics

- CPU
    
- Memory
    
- Connectivity
    
- Throughput
    

---

# Troubleshooting

## Site Cannot Reach Applications

Check:

1. Connector status
    
2. Routing
    
3. DNS
    
4. Policy assignment
    

---

## Connector Offline

Check:

1. Internet access
    
2. Firewall rules
    
3. Connector registration
    
4. Zscaler cloud connectivity
    

---

# Design Best Practices

## Redundancy

Deploy at least two connectors per critical location.

## Segmentation

Separate:

- Production
    
- Development
    
- Management
    

## Monitoring

Enable:

- Health alerts
    
- Capacity monitoring
    
- Connectivity monitoring
    

---

# Related Notes

- [[ZPA]]
    
- [[App Connector]]
    
- [[Application Segments]]
    
- [[Site Connectivity]]
    
- [[Zero Trust]]