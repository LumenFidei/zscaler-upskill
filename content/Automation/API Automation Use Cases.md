# API Automation Use Cases

## Overview

Concrete patterns for what [[OneAPI]] is actually used for in practice — moving from "what OneAPI is" to "what problems it solves."

Related:

- [[OneAPI]]
- [[ZIdentity]]
- [[Zscaler SDKs and Tooling]]

---

> [!important] ZDTE tie-in — Automation and Optimization (11%)
> This domain rewards recognizing **when a task should be automated rather than done by hand**, and **which mechanism fits the task** — API call, Terraform, scheduled job, or event-driven trigger. The scenarios below are the shape of thing this domain tests: given an operational need, identify the automation approach that fits, not just "yes, automation is possible."

---

# Use Case 1: Automated URL Filtering from Threat Intelligence

## The Manual Version

A new malicious domain appears in a threat intel feed. Someone opens the admin console, finds the right URL Filtering rule, and manually adds the domain to a category — with the ever-present risk of a typo blocking (or failing to block) the wrong destination.

## The Automated Version

```text
Threat Intel Feed
        │
        ▼
Ticket / Alert Generated
        │
        ▼
Automation calls OneAPI
        │
        ▼
URL category updated directly
        │
        ▼
Change reflected without manual console entry
```

**Why this matters:** removing the manual copy-paste step removes the typo risk entirely, and closes the gap between "threat identified" and "blocked" from however long a human takes to act, down to however long the automation pipeline takes.

See [[DNS Security]] and [[ZIA Policy Design]] for what's actually being modified on the ZIA side.

---

# Use Case 2: ZPA-as-Code in CI/CD

## The Manual Version

A new internal application is deployed. Someone separately, manually creates the Application Segment, adds it to a Segment Group, and writes an Access Policy rule — a process that can lag behind the actual deployment, or get forgotten entirely.

## The Automated Version

```text
Application Deployment Pipeline
        │
        ▼
Same pipeline provisions, via OneAPI:
   - Application Segment
   - Segment Group membership
   - Access Policy rule
        │
        ▼
Security access exists at the moment
the application goes live — not after
```

**Why this matters:** security configuration ships as part of the same pipeline as the application itself, rather than as a manual follow-up task that can slip. This is the practical realization of the least-privilege object hierarchy covered in [[Application Segments]] and [[Access Policies]] — codified instead of clicked through.

This is also the natural use case for the official Terraform providers — see [[Zscaler SDKs and Tooling]].

---

# Use Case 3: Cross-Product Orchestration

## The Scenario

ZIA detects behavior indicating a compromised user account.

## The Automated Response

```text
ZIA flags user as compromised
        │
        ▼
Automation calls OneAPI
        │
        ▼
ZPA Access Policy updated to isolate the user
        │
        ▼
Access to sensitive internal applications
cut off automatically
```

**Why this matters — and why OneAPI specifically enables it:** this requires acting across *two different products* (ZIA detection, ZPA enforcement) in immediate response to a single event. Under the legacy API model this meant two separate credential sets and two separate client configurations coordinated by custom glue code. A single OneAPI-authenticated client with appropriately scoped roles across both products makes this a much smaller, more maintainable piece of automation.

See [[Zero Trust]] for the underlying principle — continuous verification, not one-time trust — that this kind of automated response actually implements in practice rather than just describes.

---

# Use Case 4: Infrastructure-as-Code for Zscaler Configuration

Beyond one-off scripts, Zscaler configuration can be managed the same way infrastructure generally is — declared in code, versioned, and applied through a standard IaC pipeline, using the official Terraform providers for ZIA, ZPA, and ZTC.

```text
Configuration defined in .tf files
        │
        ▼
Version controlled (git)
        │
        ▼
terraform plan  → review changes before applying
        │
        ▼
terraform apply → OneAPI provisions the change
```

**Why this matters:** configuration drift — the gap between "what the console shows" and "what was actually intended" — is a common source of the "policy looks right but doesn't behave right" symptom described throughout [[Troubleshooting Methodology]]. Managing policy as code makes drift visible in a diff rather than discovered during an incident.

**Caveat carried over from [[OneAPI]]:** Terraform providers only manage what's actually exposed via the public API. A console feature not yet published through OneAPI can't be managed this way yet, regardless of how mature the provider otherwise is.

---

# Use Case 5: AI Agent Orchestration (Emerging)

A newer pattern worth knowing about even if not yet mainstream: OneAPI can be exposed to AI agents through an MCP (Model Context Protocol) server, allowing an agent to orchestrate security policy, manage users, respond to threats, and generate compliance reports across ZIA, ZPA, and ZDX without needing separate portal access for each product.

This is the same underlying OneAPI authentication and RBAC model described in [[ZIdentity]] — the agent is simply another authenticated API client, scoped like any other automation, not a special-cased integration.

---

# Design Principle Across All Five

Every pattern above shares the same shape:

```text
Trigger (event, deployment, schedule, or agent decision)
        │
        ▼
Scoped, authenticated OneAPI client acts
        │
        ▼
Change applied without manual console interaction
```

The automation is only as safe as the RBAC scope behind it — see the least-privilege guidance in [[ZIdentity]] before building any of these patterns. A broadly-scoped automation credential is a single point of failure with more blast radius than the manual process it replaced.

---

# Related Notes

- [[OneAPI]]
- [[ZIdentity]]
- [[Zscaler SDKs and Tooling]]
- [[Application Segments]]
- [[Access Policies]]
- [[Zero Trust]]
- [[Troubleshooting Methodology]]
