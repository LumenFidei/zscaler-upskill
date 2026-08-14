# Zscaler Internet Access (ZIA)

## Overview

**ZIA** is Zscaler's cloud-delivered Secure Service Edge providing secure access to the internet and SaaS applications.

---

> [!important] ZDTE exam weight
> ZIA spans most of the blueprint. Highest-value areas:
> - Locations and **sublocations** (Implementation)
> - URL vs CloudApp policy selection
> - Firewall for non-web protocols by group
> - DLP setup
> - **Dashboard interpretation** and Baseline insights (Operations, 16%)
>
> See [[ZIA Policy Design]] for policy mechanics; this note covers architecture and operations.

---

# Architecture

```text
User / Branch / Workload
     │
     ▼  ZCC · GRE · IPSec · PAC
Zscaler Service Edge (ZEN)
     │
     ├── DNS Control
     ├── Cloud Firewall
     ├── SSL Inspection
     ├── URL Filtering
     ├── Cloud App Control
     └── DLP · Sandbox · Isolation
     │
     ▼
Internet / SaaS
```

## Components

**Client layer** — ZCC, PAC, GRE, IPSec
**Cloud layer** — Service Edges, Central Authority, Policy Engine, Nanolog
**Security services** — SWG, Cloud Firewall, Sandbox, DLP, DNS Security, SSL Inspection, Browser Isolation

**Central Authority (CA)** — distributes policy and configuration to Service Edges. Worth knowing as the control plane.

**Nanolog** — the logging infrastructure; source for NSS feeds. See [[Logging and Reporting]].

---

# Locations and Sublocations

A named blueprint objective — *"identify locations and sublocations designed to meet requirements."*

## Location
Identified by **public egress IP**, GRE/IPSec tunnel, or VPN credential. Policy, bandwidth control, and authentication settings attach here.

## Sublocation
Subdivides a location by **internal IP range** behind the same public IP.

```text
Location: Chicago (203.0.113.10)
   ├── Corporate    10.10.0.0/16  → full policy, auth enforced
   ├── Guest        10.20.0.0/16  → no auth, restricted policy
   └── IoT          10.30.0.0/16  → firewall only, no SSL inspection
```

**When the answer is a sublocation:** different internal networks share one public egress IP but need different policy, authentication, bandwidth, or SSL inspection settings.

Per-sublocation settings commonly include: enforce authentication, SSL inspection, firewall enable, bandwidth control, caution/DNS settings.

**Exam trap:** an unmapped internal range falls back to the **parent location's** policy — a frequent cause of "wrong policy applied at a branch."

---

# Bandwidth Control

Applied at location/sublocation level, by class of traffic. Used to protect business-critical apps from recreational traffic. Requires the location to be defined with known bandwidth limits.

---

# Dashboards and Operations (16% domain)

Blueprint: *"Given a scenario and graphic of a ZIA dashboard, interpret the information for service health, usage trends, and security alerts"* and *"Given Baseline ZIA insights including an issue, identify the action(s) to resolve."*

## What to Read

|Signal|Interpretation|Typical action|
|---|---|---|
|Spike in blocked transactions|Policy change or campaign|Check audit log for recent rule change|
|Spike in a risky category|Behavioral shift or new threat|Review category policy, investigate users|
|High uncategorized traffic|New/unknown destinations|Consider isolation; submit for categorization|
|SSL inspection failures rising|Cert trust or pinned app|Check cert deployment, add exclusion|
|Bandwidth saturation at a location|Recreational or backup traffic|Apply bandwidth class rules|
|DLP incident spike|Real exfiltration or false positive|Review incident detail, tune engine|
|Threat detections at one location|Localized compromise|Investigate endpoints at that location|

**Method for dashboard questions:** identify the anomaly → determine whether it is a *policy* problem, a *configuration* problem, or a *security event* → the correct action follows from that classification. Answers that jump straight to a config change without diagnosis are usually wrong.

## Correlating with Audit Logs

If a metric changes sharply at a point in time, check the **audit log** near that timestamp for an administrative change — including rule reordering. See [[Logging and Reporting]].

---

# Security Services Summary

## URL Filtering
Site/category access. Actions: allow, block, caution, continue, isolate.

## Cloud App Control
Actions *within* apps — upload, download, share, post. Tenant restriction.

## Cloud Firewall
L3/L4 and network application control for non-web. Requires Tunnel 2.0 from ZCC.

## DNS Control
Protocol-agnostic, pre-connection blocking. See [[DNS Security]].

## SSL Inspection
Visibility enabler for URL, DLP, sandbox, threat engines. See [[SSL Inspection]].

## DLP
Content protection. Requires SSL Inspection. See [[DLP]].

## Sandbox
Unknown file detonation. See [[Cloud Sandbox]].

## Browser Isolation
Remote rendering for risky/uncategorized destinations. See [[Browser Isolation]].

---

# User Identification

- **Client Connector** — preferred
- **SAML** — federated
- **Kerberos** — integrated
- **Certificate** / **Surrogate IP** — where clients can't authenticate

**Surrogate IP** maps a user identity to an IP for a period, allowing user-based policy for traffic that can't carry identity (non-browser apps at a location). Worth knowing for "how do we get user policy without a client" scenarios.

---

# Troubleshooting

## Cannot Access a Website
1. Web logs — which rule matched, which engine?
2. Block page vs timeout? (policy vs connectivity)
3. Category classification
4. SSL inspection issue?
5. Rule order

## SSL Inspection Failure
Root cert deployed and trusted? Pinned application needing exclusion?

## Wrong Policy at a Site
Location/sublocation mapping.

---

# Related Notes

- [[ZIA Policy Design]]
- [[Traffic Forwarding]]
- [[SSL Inspection]]
- [[DLP]]
- [[Cloud Firewall]]
- [[DNS Security]]
- [[Logging and Reporting]]
