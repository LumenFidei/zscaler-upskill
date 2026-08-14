# Cloud Firewall

## Overview

**Cloud Firewall** is Zscaler's cloud-delivered Layer 3 / Layer 4 firewall service, providing network-level traffic control as part of [[ZIA]].

Unlike traditional on-premises firewalls, Cloud Firewall enforces rules at the Zscaler Service Edge, applying consistent policy regardless of user location.

Related:

- [[ZIA]]
- [[ZIA Policy Design]]
- [[SSL Inspection]]
- [[Traffic Forwarding]]

---

# Purpose

Cloud Firewall controls non-web traffic that URL Filtering and SSL Inspection do not address, such as:

- Non-HTTP/HTTPS protocols
- Peer-to-peer traffic
- Custom application ports
- Network-layer threats

---

# Architecture

```text
User Traffic
     │
     ▼
Zscaler Service Edge
     │
     ▼
Cloud Firewall Engine
     │
     ▼
Rule Match
     │
     ▼
Allow / Block / Log
```

---

# Rule Components

## Source Criteria

- Users
- Groups
- Departments
- Locations
- Source IP ranges

## Destination Criteria

- Destination IP
- Destination country
- Network application (e.g. Skype, BitTorrent)

## Protocol and Port

Examples:

```text
TCP 22
TCP 3389
UDP 53
```

---

# Network Application Control

Cloud Firewall can identify and control specific network applications independent of port, such as:

- File sharing applications
- VoIP applications
- Remote access tools

---

# Actions

|Action|Result|
|---|---|
|Allow|Permit traffic|
|Block|Deny traffic|
|Log|Record activity without blocking|
|Redirect|Send traffic to a different destination|

---

# IPS and Threat Protection

Cloud Firewall integrates intrusion prevention signatures to detect:

- Port scans
- Exploit attempts
- Known attack patterns

---

# Common Use Cases

- Restricting outbound SSH/RDP
- Blocking unsanctioned VPN or proxy tools
- Controlling P2P and file-sharing traffic
- Enforcing DNS-only egress

---

# Troubleshooting

## Traffic Unexpectedly Blocked

Check:

1. Rule order
2. Source/destination match
3. Protocol and port definition
4. Network application classification

---

# Best Practices

- Order rules from most specific to most general
- Avoid overly broad "allow all" rules
- Regularly review network application signatures
- Pair with SSL Inspection and DLP for full coverage

---

# Related Notes

- [[ZIA]]
- [[ZIA Policy Design]]
- [[SSL Inspection]]
- [[DLP]]
