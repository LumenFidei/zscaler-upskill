# Site Connectivity

## Overview

**Site Connectivity** refers to the general set of methods used to connect physical locations — branch offices, data centers, and cloud environments — to the Zscaler cloud, as opposed to connecting individual endpoints.

Related:

- [[ZBC]]
- [[Traffic Forwarding]]
- [[Branch Connector]]

---

# Purpose

Endpoint-based forwarding via [[ZCC]] is not always practical for servers, IoT devices, or entire office networks. Site Connectivity methods forward traffic at the network level instead.

---

# Common Methods

## Branch Connector

Virtual appliance providing outbound connectivity for a site. See [[ZBC]] and [[Branch Connector]].

## GRE Tunnels

Lightweight tunnel commonly used for branch-to-cloud connectivity, often paired with SD-WAN.

## IPsec Tunnels

Encrypted site-to-cloud connectivity, used where GRE's lack of encryption is unacceptable.

---

# Comparison

|Method|Encryption|Typical Use Case|
|---|---|---|
|Branch Connector|Yes (outbound TLS)|Full ZPA + ZIA site connectivity|
|GRE Tunnel|No|Branch ZIA connectivity, SD-WAN|
|IPsec Tunnel|Yes|Branch/DC connectivity requiring encryption|

---

# Site Connectivity Flow

```text
Site (Branch / Data Center)
         │
         ▼
Site Connectivity Method
         │
         ▼
Zscaler Cloud
         │
         ▼
Policy Enforcement
```

---

# Design Considerations

- Sites with private application needs typically require [[ZBC|Branch Connector]] rather than GRE/IPsec alone
- Redundant connectivity paths should be used for critical sites
- Bandwidth and latency requirements should drive method selection

---

# Troubleshooting

## Site Cannot Reach Zscaler Cloud

Check:

1. Tunnel/connector status
2. Underlying internet circuit health
3. Routing configuration
4. Firewall rules permitting outbound connectivity

---

# Best Practices

- Standardize connectivity methods by site type
- Document which sites use which method and why
- Monitor tunnel and connector health continuously
- Plan for failover at every critical site

---

# Related Notes

- [[ZBC]]
- [[Branch Connector]]
- [[Traffic Forwarding]]
- [[App Connectors]]
