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

---

# Application Discovery

Blueprint objective: *"Identify the steps to configure Application Discovery."*

Purpose: identify private applications actually in use so they can be onboarded, rather than guessing.

General flow:

```text
Define a broad discovery segment
        │
        ▼
Set the segment to discovery/log-only behavior
        │
        ▼
Users generate traffic
        │
        ▼
Review discovered applications in the console
        │
        ▼
Create narrow production segments from findings
        │
        ▼
Remove/narrow the discovery segment
```

Key concept: discovery is a **temporary, deliberately broad** configuration used to build an inventory. Leaving it in place permanently defeats segmentation.

Feeds directly into [[Migration from VPN to ZPA]] Phase 1–2.

---

# Bypass and Health Settings

Segment-level options that appear in scenarios:

- **Bypass** — send this app's traffic direct rather than through ZPA
- **Health reporting** — how connector health for this app is evaluated
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
