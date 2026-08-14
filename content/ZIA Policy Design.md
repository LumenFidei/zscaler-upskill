# ZIA Policy Design

## Overview

How internet traffic is inspected, classified, and controlled — and critically, **in what order**.

Related:

- [[ZIA]]
- [[Policy Evaluation]]

---

> [!important] ZDTE exam weight
> Blueprint objectives landing here:
> - "Given a scenario, identify if **URL Policy, CloudApp Policy, or some combination** is the most appropriate filtering"
> - "Given a scenario about **ZIA Firewall configuration**, identify how to enable non-web protocols to be allowed access based on user/department/group/blocked for others"
> - "Given a scenario about setting up **ZIA DLP policies**, identify how to set up and enable the policies"
> - "Given a scenario, identify how to build a policy to provide access to a specific destination for a subset of users"
>
> The URL vs CloudApp distinction is a near-certain question.

---

# Policy Evaluation Order

Traffic passes through engines in a fixed sequence. **Knowing the order tells you which engine blocked something.**

```text
Request
   │
   ▼
Authentication  (who is this?)
   │
   ▼
Location / Sublocation
   │
   ▼
DNS Control
   │
   ▼
Cloud Firewall  (L3/L4 — non-web included)
   │
   ▼
SSL Inspection  (decrypt for the engines below)
   │
   ▼
Cloud App Control  ← evaluated BEFORE URL Filtering
   │
   ▼
URL Filtering       ← SKIPPED if Cloud App Control already
   │                  issued an explicit ALLOW for this app
   ▼
Threat / Sandbox / DLP
   │
   ▼
Final Action
```

**Exam consequence:** if the Cloud Firewall blocks a port, URL Filtering never runs — adjusting URL policy won't fix it. Likewise, without SSL Inspection the engines below it see only the domain, not the full URL or payload.

Within each engine: **top-down, first match wins.** Exceptions must sit above broader rules.

> [!warning] Cloud App Control evaluates before URL Filtering — and can override it
> Per Zscaler's own documentation, when a Cloud App Control rule **explicitly allows** a cloud application, the service applies **only** the Cloud App Control policy for that request — URL Filtering is not also evaluated. A URL Filtering rule blocking the same destination does not take effect if Cloud App Control already allowed the app.
>
> **This is a distinct trap from the general "first match wins" rule** — it isn't about rule order within one engine, it's about which *engine* gets skipped entirely. If a scenario describes a URL Filtering block that mysteriously isn't enforcing on a specific cloud app, the answer is very likely a Cloud App Control allow rule sitting upstream of it, not a problem with the URL Filtering rule itself.

---

# URL Filtering vs Cloud App Control

The distinction the exam tests:

| |**URL Filtering**|**Cloud App Control**|
|---|---|---|
|Controls|Access to a site|**Actions within** an application|
|Granularity|Allow/block the destination|Upload, download, post, share, create|
|Example|Block `dropbox.com` entirely|Allow Dropbox, block **uploads** to it|
|Question it answers|"Can they go there?"|"What can they do once there?"|

## Choosing

```text
Requirement mentions blocking/allowing a SITE or CATEGORY
        → URL Filtering

Requirement mentions an ACTION inside an app
(upload, download, share, post)
        → Cloud App Control

Requirement is "allow the app but restrict what they do"
        → BOTH  (URL allows, CloudApp restricts the action)
```

**Exam trap:** "allow users to view corporate Box but prevent uploading personal files" is *not* a URL Filtering answer. URL Filtering only decides whether Box is reachable. The upload restriction is Cloud App Control.

**Tenant restriction** — allowing a corporate SaaS tenant while blocking personal tenants of the same app — is also Cloud App Control territory, not URL Filtering.

**When combining both ("allow the app but restrict what they do")**, the practical build order matters: Cloud App Control's allow is what lets the request through the engine that runs first, and its action-level rules (block upload, allow view) do the restricting. URL Filtering in this combination is mostly redundant once Cloud App Control has already allowed the app — don't rely on a URL Filtering block as a safety net for the same destination, since it may never be evaluated.

## URL Filtering Actions

|Action|Result|
|---|---|
|Allow|Access granted|
|Block|Access denied|
|Caution|Warning page, user proceeds|
|Continue|User override permitted|
|Isolate|Rendered in [[Browser Isolation]]|

Note that Caution/Continue/Isolate have no DNS-layer equivalent — if a scenario requires user override or isolation, DNS Control cannot deliver it.

---

# Cloud Firewall — Non-Web Protocols by Group

A blueprint objective verbatim. Pattern for "allow SSH/RDP for one group, block for everyone else":

```text
Rule 1 (higher)
  Criteria: Source = [authorized group]
            Destination = [target IPs/FQDNs]
            Protocol/Port = TCP 22 (or 3389)
  Action:   ALLOW

Rule 2 (lower)
  Criteria: Source = Any
            Protocol/Port = TCP 22
  Action:   BLOCK
```

**Order is the answer.** The specific allow must sit *above* the general block. Reversed, nobody gets access.

Prerequisites that appear as distractors:

- Traffic must actually reach ZIA — **Tunnel 2.0 required** for non-web from ZCC. Tunnel 1.0 forwards web ports only.
- Firewall must be enabled for that location/sublocation.
- User identity must be available for group-based matching.

Match criteria available: users, groups, departments, locations, source/destination IP, ports, protocols, network applications.

---

# DLP Policy Setup

Blueprint objective. The dependency chain matters more than the rule syntax:

```text
1. SSL Inspection enabled for the relevant traffic
        │   (without it, DLP cannot see payload in HTTPS)
        ▼
2. Define the classifier
        - Predefined dictionary, OR
        - Custom dictionary/pattern, OR
        - EDM (exact data match), OR
        - IDM (indexed document match)
        ▼
3. Create DLP engine (combines dictionaries with logic)
        ▼
4. Create DLP rule
        - Criteria: users/groups/locations/destinations
        - Engine
        - Action: allow / block / quarantine
        - Severity + notification
        ▼
5. Assign auditor / notification settings
        ▼
6. Deploy in MONITOR first, then enforce
```

**Exam trap #1:** DLP without SSL Inspection misses virtually all modern traffic. If a scenario says DLP "isn't catching anything," check SSL Inspection coverage first.

**Exam trap #2:** EDM vs IDM vs dictionary —
- **EDM** = exact records from a structured source (specific customer rows)
- **IDM** = fingerprinted whole documents
- **Dictionary/regex** = pattern types (SSN format, card number format)

Pick EDM when the requirement is "our actual customer data," not "anything shaped like an SSN."

---

# Access for a Subset of Users

General pattern across every ZIA engine:

```text
Specific ALLOW rule for the group   ← must be higher
General BLOCK rule                   ← lower
```

Scope by user, group, department, location, or sublocation. Use **sublocations** when the differentiator is an internal network behind a shared public IP — see [[Traffic Forwarding]].

---

# Layered Design

```text
DNS Control      → protocol-agnostic, pre-connection
Cloud Firewall   → L3/L4, non-web
SSL Inspection   → visibility enabler
URL Filtering    → site/category
Cloud App Control→ in-app actions
DLP / Sandbox    → content
```

---

# Troubleshooting

## User Unexpectedly Blocked

1. Check **web logs** — which rule name matched? Don't guess.
2. Identify **which engine** (firewall vs URL vs DLP) — determines the fix
3. Check rule order
4. Verify group membership and identity resolution
5. Check location/sublocation match

## Policy Looks Right but Doesn't Work

Rule order, nearly always. Check audit logs for order changes — see [[Logging and Reporting]].

---

# Best Practices

- Naming standards for rules
- Exceptions above broad rules, always
- Monitor mode before enforcement (especially DLP)
- Document every exception
- Periodic cleanup of unused rules

---

# Related Notes

- [[ZIA]]
- [[SSL Inspection]]
- [[DLP]]
- [[Cloud Firewall]]
- [[DNS Security]]
- [[Policy Evaluation]]
