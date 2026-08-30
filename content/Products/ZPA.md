# Zscaler Private Access (ZPA)

## Overview

**ZPA** provides Zero Trust Network Access to private applications without network-level access or inbound exposure.

---

> [!important] ZDTE exam weight
> Blueprint areas landing here:
> - **"Deploy Zscaler Private Service Edge"** — named certification outcome
> - App segments, Segment Groups, Server Groups, connector sizing
> - Access policies with posture and conditional rules
> - ZPA dashboard interpretation (Operations, 16%)
> - Private-app block troubleshooting (Troubleshooting, 15%)

---

# Architecture

```text
User
  │
  ▼  ZCC (or Browser Access)
ZPA Public / Private Service Edge
  │
  ▼  outbound TLS from both sides
App Connector
  │
  ▼
Private Application
```

**The defining property:** both the client and the connector make **outbound** connections that meet at the Service Edge. Nothing listens inbound. No inbound firewall rules, no DMZ, no public IP for applications.

---

# Public vs Private Service Edge

A named certification outcome — know the distinction.

| |**Public Service Edge**|**Private Service Edge**|
|---|---|---|
|Hosted by|Zscaler|**Customer**, on-premises or in their cloud|
|Location|Zscaler's global cloud|Inside the customer environment|
|Use case|Default, roaming users|Latency-sensitive local access, data residency, on-prem brokering|
|Managed via|Zscaler|Zscaler console, customer infrastructure|

## When a Private Service Edge Is the Answer

- Users and applications are **co-located** and routing via the public cloud adds unacceptable latency ("hairpinning" to the internet and back)
- Regulatory/data residency requires brokering to stay in-country or on-premises
- Local access must survive a WAN outage to the Zscaler cloud
- High-volume local traffic where local brokering is more efficient

## Deployment Considerations

- Deployed as a VM, enrolled with a provisioning key (same pattern as App Connectors)
- Requires outbound connectivity to Zscaler for control plane
- Deploy **redundantly** — same N+1 logic as connectors
- Assigned to Service Edge Groups, referenced in policy/config

---

# Object Hierarchy

The most-tested ZPA structure:

```text
Application Segment   ─── what the app is (FQDN/IP + ports)
        │
Segment Group         ─── logical grouping (policy target)
        │
Access Policy         ─── who may reach it
        │
Server Group          ─── which connectors serve it
        │
App Connector Group   ─── the connectors
```

Full detail in [[Application Segments]].

---

# Access Path

```text
User Request
     │
     ▼
Authentication (IdP)
     │
     ▼
Posture Evaluation
     │
     ▼
Access Policy Match
     │
     ▼
Server Group → App Connector
     │
     ▼
Application
```

Each stage is a possible failure point — troubleshoot in this order.

---

# Client Types

|Type|Use case|
|---|---|
|**ZCC**|Standard managed users|
|**Browser Access**|Clientless — contractors, third parties, unmanaged devices|
|**Machine Tunnel**|Pre-logon device connectivity|
|**Cloud/Branch Connector**|Workloads and sites without a client|

Access Policy can match on client type — useful for "contractors get browser-only access" designs.

## Browser Access
Requires certificate configuration for the published application. Limitations: application compatibility, web apps only. The answer when onboarding third parties without deploying a client.

**Onboarding a guest/external user (e.g. an Entra ID guest) to Browser Access or PRA has a known, non-obvious configuration requirement** — ZPA's IdP settings need "Use with Arbitrary Domains" enabled, plus a matching pair of attribute-mapping changes on the Entra ID side. Full mechanics in [[Identity Providers]].

---

# Privileged Remote Access (PRA)

Clientless, browser-based, audited access to administrative infrastructure (RDP/SSH/VNC), with session recording, clipboard/file transfer controls, and credential injection, controlled via **Privileged Capabilities** policy.

Full detail — including the specific guest-user Entra ID configuration case — now lives in its own note: [[PRA]].

---

# Dashboards and Operations

Blueprint: interpret a ZPA dashboard for service health, usage trends, security alerts.

|Signal|Interpretation|Action|
|---|---|---|
|Connector CPU/memory high|Undersized or overloaded|Add connector capacity — see [[App Connectors]]|
|Connector offline|Host, network, or outbound blocked|Check host and outbound path|
|Spike in denied access|Policy change or posture failure|Check audit log and posture results|
|One app with high latency|Connector placement or app health|Check which connector is serving it|
|Unused application segments|Stale configuration|Policy cleanup|
|Users on one connector only|No HA / group misconfiguration|Verify connector group membership|

---

# Troubleshooting Private App Access

Blueprint objective (15% domain). Work the hierarchy — but note the hierarchy below assumes the user already **authenticated successfully**. A guest/external user hitting a generic `401: Authentication Failed` never reaches this hierarchy at all — that's a domain-based IdP selection problem, not a segment/policy/connector problem. See [[Identity Providers]] before working the flow below if the symptom is an auth failure rather than a denial or timeout.

```text
Is the app in an Application Segment?     → No: publish it
        ▼ Yes
Is that segment in the targeted Segment Group?
        ▼ Yes
Does an Access Policy allow this user?    → check rule order, group membership
        ▼ Yes
Did posture pass?                          → identify failed check
        ▼ Yes
Is a Server Group assigned & healthy?
        ▼ Yes
Can the connector resolve DNS + route to the app?
```

**Discriminator:** app not appearing at all → segment/policy problem. App appears but won't connect → connector/DNS/routing problem.

Check **ZPA user activity logs** to see the actual decision and matching rule rather than inferring.

---

# Security Benefits

- Applications never internet-facing
- No network-level access — eliminates lateral movement paths
- Application-specific least privilege
- Identity-follows-user policy
- Continuous posture verification

---

# Related Notes

- [[Application Segments]]
- [[App Connectors]]
- [[Access Policies]]
- [[ZPA Policy Design]]
- [[Device Posture]]
- [[Migration from VPN to ZPA]]
- [[Logging and Reporting]]
- [[PRA]]
- [[Identity Providers]]
