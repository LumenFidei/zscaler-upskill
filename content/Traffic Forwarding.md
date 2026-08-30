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

# Location-Based Policies (Rulesets)

A more granular ZCC 4.8+ (Windows only, requires contacting Zscaler Support to enable) mechanism than a simple App Profile split tunnel: **rulesets** that steer traffic *and* enforce endpoint firewall rules, scoped by network type (On-Trusted vs. Off-Trusted).

## What a Ruleset Can Contain

A ruleset can include either or both:

- **Traffic steering rules** — DNS domain inclusion/exclusion, IP inclusion/exclusion, application bypasses
- **Endpoint firewall rules** — inbound and outbound, including default (catch-all) behavior

Configured at: **Infrastructure → Connectors → Client → Location Based Policies**. Up to **300 rulesets** total.

> [!note] Both network types require an assignment
> You must select a ruleset for **On-Trusted** and a ruleset for **Off-Trusted** separately — if you want different behavior per network type, you need to actually build and assign two rulesets, not one ruleset that "handles both."

## Traffic Steering Fields

- **Domain Inclusion** — DNS for these domains goes to the Public Service Edge (ZIA)
- **Domain Exclusion** — DNS for these domains goes to the device's local DNS server instead
- **Prioritize DNS Exclusions over Z-Tunnel 2.0** — when enabled, exclusions take precedence over inclusions where both could otherwise match
- **Process-Based Application Bypass** — works for both Tunnel 1.0 and 2.0
- **Predefined / Custom IP-Based Application Bypass** — custom IP-based bypass is Tunnel 2.0 only
- **IP Inclusion/Exclusion Lists** — reference a Traffic Steering IP List (below), or use the built-in defaults
- **Source Port-Based Bypass** — port (1–65535) + protocol (TCP/UDP/`*`)

> [!warning] Zscaler recommends against source-port-based bypass
> Zscaler's own guidance is to use **endpoint firewall rules instead of source-port-based bypass**, because firewall rules are **stateful** and source-port bypass is not. If a scenario is choosing between the two for a similar-sounding requirement, stateful firewall rules are the better-practice answer.

## Inbound and Outbound Firewall Rules

Each direction is configured independently, up to **10 rules** per direction per ruleset:

- **Rule Order** — ascending numerical precedence (same first-match-style discipline as everywhere else in Zscaler policy)
- **Firewall IP List** — `Any`, `Local subnet` (auto-detected; ZCC generates both IPv4 multicast and IPv6 link-local/multicast filters from it automatically), or a named Firewall IP List
- **Application** — a specific app, or `Any`
- **Action** — Firewall Allow / Firewall Block
- At least one of **Firewall IP List** or **Application** must be selected

## Default Rules (the Catch-All)

For traffic matching none of the explicit rules in a direction:

|Direction|Options|Default|
|---|---|---|
|Inbound|None (use existing host firewall policy, e.g. Windows Defender) / Firewall Allow / Firewall Block|**None**|
|Outbound|None (use existing host firewall policy) / Firewall Allow|**None**|

> [!warning] Two easy-to-miss constraints
> - **Loopback traffic is always allowed**, regardless of what the default action is set to — the default action never applies to it.
> - **If a ruleset includes any outbound firewall rules at all, the Default Outbound Firewall Rule must be set to Firewall Allow** — leaving it at "None" alongside explicit outbound rules is not a supported combination.
>
> Separately: **Allow ZPA Client to Client Traffic** is an inbound-rule option that permits inbound Private Access client-to-client traffic (e.g. client-based remote assistance in [[PRA]]) even when all other inbound traffic is otherwise blocked by the ruleset.

## Traffic Steering IP Lists

A separate tab on the same Location Based Policies page — reusable IP lists referenced by rulesets (not directly by App Profiles — see gotcha below).

**Built-in defaults** (view-only, but downloadable as CSV and copyable into a custom list):

```text
Default IP Inclusion List (IPv4): 0.0.0.0/0

Default IP Exclusion List:
  IPv4: 224.0.0.0/4 (multicast), 169.254.0.0/16 (link-local),
        255.255.255.255 (broadcast), RFC 1918 ranges
        (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
  IPv6: FF00::/8 (multicast), FE80::/10 (link-local), FC00::/7 (unique local)
```

Custom lists accept IP/subnet/wildcard, `IP:Port`, `IP:Port-range`, and `IP:Port:Protocol` formats for both IPv4 and IPv6. Max 300 custom lists; IPv4 lists cap at ~682 addresses, IPv6 at ~307 (both bounded by a 6,144-character limit — IPv6's longer notation means fewer addresses fit).

> [!warning] Two gotchas worth knowing before you build these
> - **A Traffic Steering IP List can only be used inside a ruleset — it cannot be assigned directly to an App Profile.** If a design calls for referencing one straight from an App Profile, that's not supported; it has to route through a ruleset.
> - **Uploading a CSV replaces the existing list entirely** — it is not additive. Uploading a partial list to "add a few IPs" will silently delete everything not included in that upload.

## Deployment

After saving a ruleset, attach it to an App Profile via the **Location Policies** tab and apply it to one or more network types. See [[App Profiles]].

---

# Split Tunneling and Bypasses

Blueprint objective: *"Given a scenario where a user is in a known location, identify how to bypass their traffic."*

Mechanisms, and when each applies:

|Mechanism|Applies to|
|---|---|
|**Trusted Network Detection**|Detects the user is on a corporate network → alter/disable forwarding|
|**App Profile split tunnel**|Include/exclude specific destinations from the tunnel|
|**Location-Based Policy ruleset**|Granular, network-type-scoped traffic steering *and* endpoint firewall, ZCC 4.8+ Windows|
|**PAC `DIRECT` return**|Browser-level bypass|
|**ZPA app segment scoping**|Controls which private apps are tunneled|

Common exclusions:

- RFC1918 internal ranges
- VoIP and video conferencing (latency-sensitive)
- Certificate-pinned applications
- Management networks

See [[Trusted Network Detection]] and [[App Profiles]].

---

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

## Authentication Failures After Enabling a Location-Based Ruleset

If a device with a custom outbound default firewall rule set to **deny** starts failing to authenticate, check whether the host firewall is actually permitting outbound traffic to the IdP — a deny-by-default outbound posture silently blocks the SAML round-trip along with everything else, and the symptom (auth failure) looks identical to an IdP configuration problem rather than a local firewall one. See [[Identity Providers]] for the IdP side of authentication troubleshooting.

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
- [[PRA]]
- [[Identity Providers]]
