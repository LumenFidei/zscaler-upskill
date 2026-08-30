# Application Segments

## Overview

**Application Segments** define private applications published through [[ZPA]]. They replace network-based access rules with application-specific access control.

Related:

- [[ZPA]]
- [[App Connectors]]
- [[Access Policies]]

---

> [!important] ZDTE exam weight
> Blueprint objectives landing here:
> - "Given a scenario about locking down an application for specific authorized users, identify how to build that AppSegment"
> - "Given a scenario about creating ZPA application segments, identify how to create the application segments and map them to private applications"
> - "Identify the steps of consolidating application segments into local Segment Groups"
> - "Identify the steps to configure Application Discovery"
>
> Know the **object hierarchy** and which object does what. This is the most commonly confused area in ZPA.

---

# The ZPA Object Hierarchy

This is the highest-value thing to memorize for ZPA questions:

```text
Application Segment   ─── what the app IS (FQDN/IP + ports)
        │
        ▼
Segment Group         ─── logical grouping of app segments
        │
        ▼
Access Policy         ─── WHO may reach it
        │
        ▼
Server Group          ─── which connectors serve it
        │
        ▼
App Connector Group   ─── the connectors themselves
```

**Exam-critical distinctions:**

| Object | Answers | Common confusion |
|---|---|---|
|Application Segment|"What is the app?"|Not where policy lives|
|Segment Group|"Which apps belong together?"|Used as the policy target — simplifies rules|
|Server Group|"Which connectors reach it?"|Not a grouping of *applications*|
|App Connector Group|"Which connectors, where?"|Bound to Server Group|

If a question asks how to make one access rule cover several related apps → **Segment Group**.
If a question asks how to control which connectors serve an app → **Server Group**.

---

# Building an Application Segment

## Required Components

**Name** — descriptive, e.g. `Finance Portal`

**Domain / IP**
```text
finance.company.com
10.20.30.0/24
```
Supports FQDN, wildcard FQDN, individual IP, or CIDR range.

**TCP/UDP Ports**
```text
443
3389
22
```

**Segment Group** assignment

**Server Group** assignment

---

# Locking Down an App for Specific Users

A blueprint objective verbatim. The mechanism is a two-part answer, and partial answers are the wrong answers:

1. **Define the segment narrowly** — only the FQDN(s) and only the ports the app actually needs. A broad segment (wildcard domain, wide port range) is the failure mode.
2. **Write an Access Policy** scoping that Segment Group to the specific user group, with posture and context conditions.

The segment does **not** grant access on its own — an Access Policy must permit it. Conversely, a policy can't grant access to something not defined in a segment.

---

# Wildcards and Overlap

Wildcard segments (`*.company.com`) are convenient but dangerous:

- Overlapping segments create ambiguous matching
- Broad wildcards undermine least privilege
- Troubleshooting becomes harder — you can't tell which segment matched

**Best practice, and the likely exam-correct answer:** specific segments per application, avoid overlap, reserve wildcards for genuinely dynamic app families.

> [!warning] The most important, easily-missed rule: ZPA always matches the *most specific* segment
> This is worth internalizing precisely, because it silently breaks policy coverage rather than throwing an error:
>
> 1. You have a wildcard segment `*.exapp.company.com`, added to an Access Policy.
> 2. You later create a new, more specific segment for `file.exapp.company.com`.
> 3. **`file.exapp.company.com` is now no longer covered by the original policy** — even though the wildcard would still technically match it, ZPA always evaluates policy against the *most specific* segment that matches, not the broadest one.
>
> The fix isn't a bug workaround — it's designing for it: any time you carve a specific segment out of a wildcard's territory, you need a **new** Access Policy rule for that specific segment, because the old one no longer applies to it.
>
> **A related, separate gotcha:** ports do **not** inherit from a broad segment (IP subnet or wildcard) down to a more specific one (a specific IP or FQDN) that also matches. If the specific segment needs particular ports, they must be **explicitly specified on it** — they are not inherited from the wildcard segment that would otherwise have matched.

---

# Application Discovery

Blueprint objective: *"Identify the steps to configure Application Discovery."*

Purpose: identify private applications actually in use so they can be onboarded, rather than guessing — without configuring an explicit segment for every application up front.

## How It Actually Works

```text
Wildcard/domain segment configured for discovery
        │
        ▼
User requests an app matching that domain
        │
        ▼
ZCC redirects the request to the App Connectors
configured for that segment
        │
        ▼
App Connector performs a DNS lookup to process the request
        │
        ▼
If successful: app appears in "Applications Discovered
in Past 14 Days" widget (Applications dashboard)
```

**Configuration:** enter the domain in wildcard format within an application segment — either `*.<domain>` or `.<domain>` (e.g. `*.safemarch.com` or `.safemarch.com`; both forms work). **Wildcard segments can be used for discovery** — this is explicitly supported, not a workaround.

## Conditions for Discovery to Work Efficiently

- The TCP/UDP **port range** specified for the domain should be **wide**
- The application policy should allow access to a **broad set of users**

Narrow ports or narrow user scope on the discovery segment will undercount what's actually in use — the mechanism depends on enough real traffic actually being permitted to flow through it to observe.

> [!note] Match your DNS search domains
> If enabling discovery for bare domain suffixes, align it with your organization's DNS search domains. Otherwise, requests made as short hostnames (not FQDNs) may never get the domain suffix appended, meaning no qualifying FQDN is ever sent through for discovery — the app just never shows up, with no error to explain why.

## Health Reporting Limitation

Discovered (not yet explicitly defined) applications only support **On Access** health reporting — the App Connector reports health starting when a user accesses the app, continuing for up to **30 minutes** after the last access. After that window, status reverts to **Unknown** until the next access. **Continuous health reporting requires explicitly defining the application** as its own segment — it's not available for pure-discovery-mode apps.

## Excluding Specific Apps From an Otherwise-Discovered Domain

The **Bypass** field lets you exclude specific applications from Private Access even within a domain that's otherwise configured for discovery — useful when most of a domain's apps should be onboarded via discovery, but a few specific ones should never be reachable through ZPA.

## The 14-Day Window

Discovered applications are listed most-to-least-recent. **If an app was discovered more than 14 days ago, its original User Activity diagnostics are no longer available** — worth knowing before assuming discovery data will always be there to review retroactively.

Feeds directly into [[Migration from VPN to ZPA]] Phase 1–2.

---

# Defining a Dynamically Discovered Application

Converting a discovered (wildcard-matched) app into its own explicit segment — required if you want a dedicated access policy for it, or need settings discovery mode doesn't support (like Continuous health reporting or a custom Bypass configuration).

## Procedure

```text
Analytics (top nav)
        │
        ▼
Enable "Switch to Existing Reports"
        │
        ▼
Private Applications → Applications
        │
        ▼
"Applications Discovered in Past 14 Days" widget
   (add from the widgets dropdown if not already shown —
    max 8 widgets total can be selected/displayed)
        │
        ▼
Click "Add Application Segment"
        │
        ▼
Select the specific discovered apps → "Define Selected Applications"
        │
        ▼
Add Application Segment window opens — configure normally
```

**This is exactly the action that triggers the most-specific-segment gotcha above** — defining a discovered app this way carves it out of whatever wildcard segment was previously matching it, and any existing policy on that wildcard no longer covers the newly-defined specific segment. Plan for a new Access Policy rule as part of this same change, not as an afterthought discovered later when access mysteriously breaks.

---

# Private Link Domains

A specialized segment-matching mechanism for applications hosted behind a cloud provider's private endpoint (e.g. **Azure Private Link**) — lets ZPA route only the traffic that's actually destined for the private endpoint, without needing an overly broad application segment that would otherwise capture unrelated traffic to the same public domain.

## The Problem This Solves

Configuring a broad public domain directly in an application segment (e.g. `*.database.windows.net`) works, but causes **all** traffic for that domain — including resources that have nothing to do with the private link — to route through ZPA. Unnecessary and potentially incorrect routing.

## How It Works

```text
1. User Access
   User requests a resource by its standard public domain
   (e.g. companyserver.database.windows.net)
        │
        ▼
2. DNS Resolution
   ZPA evaluates the DNS records returned for the request
        │
        ▼
3. Private Link Resolution
   DNS returns a CNAME pointing to a private link alias
   (e.g. companyserver.privatelink.database.windows.net)
   — this alias resolves to a private IP inside the cloud VNet
        │
        ▼
4. Application Segment Matching
   ZPA evaluates up to 5 CNAME records (left to right,
   ascending order) in the public DNS response
```

**Both conditions must be true** for private link routing to actually occur:

1. The resolved CNAME matches a **configured private link domain**
2. The private link alias itself (e.g. `*.privatelink.database.windows.net`) is defined in an **application segment**, including its required ports and protocols

If either is missing, the request doesn't route through the private link path — it's not enough to configure just one side.

## Configuration

```text
Policies → Access Control → Private Applications → App Segments
   → Defined Application Segments
        │
        ▼
Column Menu icon (⋮) → Private Link Domains
        │
        ▼
Enter the domain in "Frontend Wildcard Domains for
DNS Request and CNAME Match" (e.g. *.database.windows.net)
        │
        ▼
Confirm a matching application segment exists covering
the private link alias itself, with ports/protocols specified
        │
        ▼
Save
```

Multiple private link domains can be added via "Add More" in the same configuration window.

---

# Bypass and Health Settings

Segment-level options that appear in scenarios:

- **Bypass** — send this app's traffic direct rather than through ZPA (also usable to exclude specific apps from an otherwise-discovered domain, above)
- **Health reporting** — On Access (discovery default, 30-minute decay to Unknown) vs. Continuous (requires an explicitly defined segment)
- **Double encryption** — additional encryption layer for sensitive apps
- **Application discovery / ICMP behavior**

---

# Access Flow

```text
User Request
      │
      ▼
Application Segment Match
      │
      ▼
Access Policy Evaluation  (user + posture + context)
      │
      ▼
Server Group → App Connector
      │
      ▼
Application
```

---

# Troubleshooting

## Application Not Available

Work the hierarchy in order — this is the exam-friendly method:

1. **Segment definition** — is the FQDN/IP and port actually included?
2. **Segment Group** — is the segment in the group the policy targets?
3. **Access Policy** — does a rule permit this user? Check rule order.
4. **Server Group** — is one assigned, and is it enabled?
5. **Connector health** — see [[App Connectors]]
6. **DNS** — can the connector resolve the app's name?

## Wrong App Matched

Look for overlapping segments or an over-broad wildcard capturing the hostname.

---

# Best Practices

- Descriptive, consistent naming
- One application per segment where practical
- Minimum necessary ports
- Group by business function into Segment Groups
- Review and remove unused segments regularly

---

# Related Notes

- [[ZPA]]
- [[App Connectors]]
- [[Access Policies]]
- [[ZPA Policy Design]]
- [[Migration from VPN to ZPA]]
- [[Common Gotchas]]
