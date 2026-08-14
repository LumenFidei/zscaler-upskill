# ZPA Policy Design

## Overview

Design principles for who accesses private applications and under what conditions.

Related:

- [[ZPA]]
- [[Access Policies]]
- [[Application Segments]]

---

> [!important] ZDTE exam weight
> Blueprint objectives:
> - "Given a least privilege scenario, identify how to ensure policies are **implemented** appropriately based on specific user identities, device posture, and application attributes"
> - "Given a least privilege scenario, identify how to ensure policies are **enforced** appropriately..."
> - "Given a scenario including a specific application segment policy, identify the action for regular policy clean-up"
> - "Given a scenario including a security posture and a policy review, identify aspects of **effectiveness, redundancy, and alignment**"
>
> Note the exam separates *implemented* from *enforced* — design vs runtime verification.

---

# Implemented vs Enforced

A distinction the blueprint makes explicitly:

|**Implemented**|**Enforced**|
|---|---|
|The policy is configured correctly|The policy actually takes effect at runtime|
|Right groups, segments, posture profiles|Rule order correct, posture actually evaluating, identity resolving|
|Verified in the console|Verified in **logs**|

A policy can be perfectly implemented and never enforced — because a broader rule sits above it, group membership isn't syncing, or the posture profile isn't attached. **When a question asks about enforcement, the answer involves checking logs or rule order, not editing the rule.**

---

# The Decision Model

```text
User Identity
 +
Device Posture
 +
Context (location, client type, time)
 +
Application
 =
Access Decision
```

All four must be considered for a complete least-privilege answer.

---

# Least Privilege — The Three-Part Answer

Partial answers are the wrong answers. Complete least privilege requires:

## 1. Application attributes — narrow the segment

```text
AVOID:  *.company.com, ports 1-65535
PREFER: finance.company.com, port 443
```

## 2. User identity — scope the rule

```text
AVOID:  Criteria = Any user
PREFER: Criteria = Finance-Users group
```

## 3. Device posture — condition the access

```text
Sensitive app → managed + encrypted + EDR healthy
```

Missing any one leaves an over-permissive path.

---

# Segmentation Design

Group applications by:

- **Business function** — HR, Finance, Engineering
- **Risk level** — standard vs privileged
- **Ownership** — who approves access

```text
Segment Group: Finance Apps
   ├── finance-portal.company.com:443
   ├── reporting.company.com:443
   └── ap-system.company.com:8443
```

Then one Access Policy rule targets the Segment Group rather than each app — this is the consolidation the exam asks about.

---

# Privileged Access

Higher-sensitivity design pattern:

- MFA required
- Managed device mandatory
- Strong posture (EDR + encryption + certificate)
- Narrow segments, narrow ports
- Session recording where PRA is in use
- Shorter timeout policy

---

# Policy Review — Effectiveness, Redundancy, Alignment

Blueprint objective. What to look for in a review scenario:

## Effectiveness
- Do rules actually match traffic? (hit counts)
- Do posture conditions reflect current security standards?
- Are sensitive apps genuinely gated?

## Redundancy
- Multiple rules granting the same group the same access
- Overlapping application segments
- Segment Groups with identical membership
- Rules made unreachable by rules above them

## Alignment
- Does policy reflect the stated security posture?
- Any "any user / any app" rules surviving from pilot?
- Contractors on the same rules as employees?
- Decommissioned apps still published?

---

# Policy Lifecycle

```text
Discover  → Application Discovery, VPN log analysis
   │
Define    → segments, groups, policies
   │
Test      → pilot group, monitor mode
   │
Deploy    → phased rollout
   │
Review    → cleanup, consolidation, recertification
```

See [[Migration from VPN to ZPA]].

---

# Cleanup Actions

- Delete rules with zero hits over the review window
- Consolidate rules with identical criteria and targets
- Merge overlapping segments
- Remove segments for retired applications
- Recertify group membership
- Tighten wildcard segments discovered during review

---

# Troubleshooting Design Problems

## Users have more access than intended
Look for wildcard segments, "any user" rules, or a broad allow above narrower rules.

## Policy exists but has no effect
Check rule order and whether the segment is in the targeted Segment Group.

## Posture requirement not applying
Confirm the posture profile is attached to the rule and the checks are actually evaluating on the client.

---

# Related Notes

- [[ZPA]]
- [[Access Policies]]
- [[Application Segments]]
- [[App Connectors]]
- [[Device Posture]]
- [[Migration from VPN to ZPA]]
