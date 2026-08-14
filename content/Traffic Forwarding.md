# Traffic Forwarding

## Overview

**Traffic Forwarding** describes how user traffic is redirected from endpoints, branch locations, and networks to Zscaler cloud services for inspection, policy enforcement, and secure access.

Related:

- [[ZCC]]
- [[ZIA]]
- [[PAC Files]]
- [[Service Edge]]

---

> [!important] ZDTE exam weight
> This is the single highest-yield topic in the vault. Blueprint objectives that land here:
> - "Given a set of user groups and geographical locations, identify and explain appropriate ZIA connectivity methods (GRE, IPSec, PAC, ZCC) for each"
> - "Given a network topology and user scenario, identify the most appropriate ZIA traffic forwarding strategy"
> - "Given a scenario about an implementation without a client connector (e.g. workload), identify how users are able to send traffic to Zscaler"
> - "Given a scenario where a user is in a known location, identify how to bypass their traffic"
>
> Expect method-selection scenarios. Know the decision criteria cold.

---

# Method Selection: The Core Exam Skill

|Method|Encrypted|Best for|Key constraint|
|---|---|---|---|
|**ZCC (Tunnel 2.0)**|Yes|Roaming/remote users|Requires client install|
|**GRE**|**No**|Branch, high bandwidth|Requires **static public IP**|
|**IPSec**|Yes|Branch where encryption required or IP is dynamic|Lower throughput per tunnel than GRE|
|**PAC**|N/A|Browser-only, legacy, selective routing|Web traffic only; no non-web protocols|

## Decision Logic

```text
Is it a roaming user or laptop?
   → ZCC

Is it a fixed site?
   ├── Static public IP + high bandwidth + encryption not required → GRE
   └── Dynamic IP, or encryption required → IPSec

Is it a server/workload with no client possible?
   → Explicit proxy, PAC, or tunnel from the network

Is it browser-only / legacy proxy environment?
   → PAC
```

**Exam trap:** GRE has *no encryption*. If a scenario mentions a compliance requirement for encrypted site-to-cloud transport, the answer is IPSec, not GRE.

**Exam trap:** PAC only handles what the browser sends. Non-web protocols (SSH, RDP, arbitrary TCP/UDP) will not be forwarded by a PAC file — those need a tunnel or ZCC.

---

# Endpoint Forwarding (ZCC)

```text
User Device
      │
      ▼
ZCC
      │
      ▼
Zscaler Cloud
      │
      ▼
Internet / Private Application
```

## Tunnel 1.0 vs Tunnel 2.0

|                | Tunnel 1.0 | Tunnel 2.0 |
|---|---|---|
|Scope|Web ports (80/443)|**All ports and protocols**|
|Model|Route-based / proxy-style|Packet-filter based|
|Non-web traffic|Not forwarded|Forwarded|

**Exam relevance:** if a scenario needs non-web protocols inspected by the ZIA firewall, Tunnel 2.0 is required. Tunnel 1.0 alone will not deliver that traffic.

---

# Branch Forwarding

```text
Branch Network
       │
       ▼
GRE / IPSec Tunnel
       │
       ▼
Zscaler Cloud
```

## GRE

- Lightweight encapsulation, no encryption overhead
- Requires a **static public IP** on the customer edge for tunnel registration
- Common with SD-WAN
- Typically supports higher per-tunnel throughput than IPSec
- Deploy **redundant tunnels** to primary and secondary data centers

## IPSec

- Encrypted transport (IKE negotiation)
- Works with **dynamic IP** — critical differentiator from GRE
- Lower per-tunnel throughput than GRE; scale by adding tunnels
- Choose when encryption is a stated requirement

> [!note] Verify current throughput figures
> Per-tunnel bandwidth ceilings change between releases and licence tiers. Confirm current numbers in Zscaler's help documentation rather than memorizing a figure — the exam's sizing questions reference the Help Doc chart directly.

---

# Locations and Sublocations

Blueprint objective: *"Given a set of requirements, identify locations and sublocations designed to meet those requirements."*

## Location

Represents a traffic source, identified by its **public egress IP** (or by tunnel/VPN credential). Policy and bandwidth control attach here.

## Sublocation

A subdivision of a location defined by **internal IP ranges** behind the same public egress IP.

```text
Location: Chicago Branch  (public IP 203.0.113.10)
   ├── Sublocation: Corporate    10.10.0.0/16
   ├── Sublocation: Guest Wi-Fi  10.20.0.0/16
   └── Sublocation: IoT          10.30.0.0/16
```

**Why this matters on the exam:** sublocations are the mechanism for applying *different policy, authentication, or bandwidth rules to different internal networks that share one public IP*. If a scenario says "guest network needs looser policy but comes from the same office egress," the answer is a sublocation.

Sublocation-level settings commonly include:

- Enforce authentication (on/off)
- SSL inspection
- Bandwidth control
- Firewall enablement

---

# Split Tunneling and Bypasses

Blueprint objective: *"Given a scenario where a user is in a known location, identify how to bypass their traffic."*

Mechanisms, and when each applies:

|Mechanism|Applies to|
|---|---|
|**Trusted Network Detection**|Detects the user is on a corporate network → alter/disable forwarding|
|**App Profile split tunnel**|Include/exclude specific destinations from the tunnel|
|**PAC `DIRECT` return**|Browser-level bypass|
|**ZPA app segment scoping**|Controls which private apps are tunneled|

Common exclusions:

- RFC1918 internal ranges
- VoIP and video conferencing (latency-sensitive)
- Certificate-pinned applications
- Management networks

See [[Trusted Network Detection]] and [[App Profiles]].

---

# Workloads Without a Client

For servers, IoT, and cloud workloads where ZCC cannot be installed:

- Explicit proxy configuration to the Zscaler node
- PAC file where the application honors proxy settings
- GRE/IPSec tunnel from the hosting network
- Cloud Connector / Branch Connector for cloud and site egress

See [[ZBC]] and [[Site Connectivity]].

---

# Troubleshooting

## Traffic Not Forwarding

1. ZCC status and authentication state
2. Tunnel status (and which version — 1.0 vs 2.0)
3. App Profile assignment — is the user in the profile they should be?
4. Split tunnel / bypass rules unintentionally matching
5. Trusted Network Detection falsely reporting "trusted"
6. PAC logic (see [[PAC Files]])

## Wrong Policy Applied at a Branch

Check location vs **sublocation** match — a device in an unmapped internal range falls back to the parent location's policy.

---

# Best Practices

- Standardize forwarding profiles per population
- Minimize bypass rules; document every exception
- Deploy redundant tunnels per site
- Monitor tunnel health continuously

---

# Related Notes

- [[ZCC]]
- [[ZIA]]
- [[PAC Files]]
- [[Tunnel 2.0]]
- [[Trusted Network Detection]]
- [[App Profiles]]
