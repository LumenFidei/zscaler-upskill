# Access Policies

## Overview

**Access Policies** are the ZPA rules determining whether a user may connect to a private application, based on identity, device, and context.

Related:

- [[ZPA]]
- [[Application Segments]]
- [[Device Posture]]

---

> [!important] ZDTE exam weight
> Blueprint objective: *"Identify the steps to create access policies within the ZPA administration console incorporating **posture checks** (e.g., device posture, health score, AV, etc.) and **conditional rules** (e.g., user roles, time-based access)."*
>
> Also feeds the least-privilege objectives under Architecture and Design (22%).

---

# ZPA Policy Types — Know Which Does What

A frequent source of wrong answers. ZPA has several distinct policy types:

|Policy|Decides|
|---|---|
|**Access Policy**|Whether a user may reach an application|
|**Timeout Policy**|Session and re-authentication duration|
|**Client Forwarding Policy**|Whether traffic is forwarded to ZPA, bypassed, or blocked at the client|
|**Isolation Policy**|Whether access is rendered in isolation|
|**App Protection Policy**|Inline inspection of app traffic|
|**Privileged Capabilities**|Controls for PRA sessions (recording, clipboard, file transfer)|

**Exam trap:** if the requirement is "user in a known/trusted location should bypass ZPA," that is a **Client Forwarding Policy**, not an Access Policy. Access Policy governs permission, not path.

---

# Building an Access Policy

```text
1. Define/confirm the Application Segment
        ▼
2. Add it to a Segment Group
        ▼
3. Create Access Policy rule:
      - Criteria: user / group / SAML or SCIM attribute
      - Target:   Segment Group (or app segment)
      - Conditions: posture profile, client type, country,
                    platform, time (via timeout/conditions)
      - Action:  ALLOW or BLOCK
        ▼
4. Place the rule correctly in evaluation order
        ▼
5. Verify with ZPA user activity logs
```

## Criteria Available

- **Identity** — user, group, SAML attribute, SCIM attribute
- **Posture** — device posture profile results
- **Client type** — ZCC, Browser Access, Machine Tunnel, Cloud Connector
- **Location / country**
- **Platform** — Windows, macOS, iOS, Android, Linux
- **Trusted network**

**Exam-relevant:** SAML vs SCIM attributes both appear. SAML attributes come with the assertion at authentication; SCIM attributes come from directory provisioning. Group-based rules generally rely on one of these being correctly mapped — a common root cause when a policy "should" match but doesn't.

---

# Posture Checks in Access Policy

Posture is referenced in Access Policy as a **posture profile**, evaluated at connection time and re-evaluated during the session.

Common checks:

- Managed/domain-joined status
- Disk encryption (BitLocker / FileVault)
- AV present, running, updated
- EDR present and healthy
- Certificate presence
- OS version / patch level
- **ZDX health score** (device experience as a trust signal)

Design pattern the exam favors:

```text
Privileged / sensitive app
        │
        ▼
Requires: managed device + encryption + EDR + MFA

Standard app
        │
        ▼
Requires: managed device only

Contractor app
        │
        ▼
Browser Access, no posture requirement
```

See [[Device Posture]] for the full check catalogue.

---

# Conditional Rules

## Time-Based Access

Restrict access windows — e.g. contractors only during business hours. Implemented via rule conditions and timeout settings.

## Role-Based

Match on group or SAML/SCIM attribute representing role. Least-privilege scenarios almost always resolve to "scope the rule to the specific group, targeting the specific Segment Group."

---

# Rule Order

Top-down, **first match wins**. Consequences:

- A broad allow above a specific deny makes the deny unreachable
- A block rule at the top can shadow everything below it
- Reordering rules changes behavior without changing any rule's content — and shows in audit logs

There is an implicit deny: traffic not matching any allow rule is denied. **Exam relevance:** you do not need an explicit block rule to deny unlisted access — least privilege is the default.

---

# Least Privilege Design

```text
AVOID                          PREFER
─────                          ──────
All Users                      Finance Users
    │                              │
All Applications               Finance Segment Group
```

Least-privilege answers combine three things — a partial answer is wrong:

1. **Narrow application segment** (only needed FQDNs/ports)
2. **Scoped access policy** (specific group, not "any")
3. **Posture and context conditions** appropriate to sensitivity

---

# Troubleshooting

## User Denied Unexpectedly

1. **ZPA user activity log** — which rule matched, what was the decision?
2. Group membership / SAML-SCIM attribute actually present?
3. Rule order — is a broader rule matching first?
4. Posture result — which check failed?
5. Is the app in the Segment Group the rule targets?
6. Client type — is the rule restricted to a client type they aren't using?

## User Allowed Who Shouldn't Be

Look for an over-broad rule above the intended one, or a wildcard application segment capturing more than intended.

---

# Policy Cleanup

Blueprint objective under Operations: *"identify the action for regular policy clean-up (removing unused rules, consolidating objects)."*

- Remove rules with no hits over a defined period
- Consolidate overlapping rules targeting the same population
- Merge redundant Segment Groups
- Remove segments for decommissioned apps
- Review after every migration phase

---

# Related Notes

- [[ZPA]]
- [[ZPA Policy Design]]
- [[Application Segments]]
- [[Device Posture]]
- [[Policy Evaluation]]
