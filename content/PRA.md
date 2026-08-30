# Privileged Remote Access (PRA)

## Overview

**Privileged Remote Access (PRA)** is a ZPA capability providing clientless, browser-based access to administrative infrastructure — RDP, SSH, and VNC sessions rendered through the browser, with no client software and no exposed network-level access.

Related:

- [[ZPA]]
- [[Access Policies]]
- [[Identity Providers]]

---

# Purpose

PRA replaces the traditional model of VPN-plus-RDP-client access to servers and network devices with a Zero Trust equivalent: the administrator never gets network reach to the target, only an authorized, audited session to a specific resource.

```text
Traditional            PRA
─────────────          ───────────────────
Admin                  Admin
  │                      │
  ▼                      ▼
VPN                    Browser
  │                      │
  ▼                      ▼
Network Access         Authorized Session
  │                      │
  ▼                      ▼
Target Server          Target Server
(and everything        (only this resource,
 else on that           through ZPA policy)
 network segment)
```

---

# Core Capabilities

## Clientless Protocol Access
RDP, SSH, and VNC sessions run through the browser — no local RDP/SSH client, no locally cached credentials.

## Session Recording
Full session capture for audit and compliance review — a common requirement for privileged access to production infrastructure.

## Clipboard and File Transfer Controls
Granular control over whether clipboard content or files can move in or out of the session, independent of whether the session itself is otherwise permitted.

## Credential Injection
The administrator authenticates to ZPA and is authorized for the session; the actual target credential is injected server-side. The admin never sees or handles the underlying password — reduces credential sprawl and the risk of a shared privileged password leaking to an individual user.

---

# Policy Control — Privileged Capabilities

PRA-specific behavior (recording on/off, clipboard, file transfer, specific protocol permissions) is governed by a distinct **Privileged Capabilities** policy layered on top of the standard [[Access Policies|Access Policy]] that determines *whether* the user reaches the resource at all.

```text
Access Policy         → can this user reach this resource at all?
        │
        ▼
Privileged Capabilities → what can they do once connected?
                          (recording, clipboard, file transfer)
```

If a scenario requires session auditing or recording specifically, the answer is configuring Privileged Capabilities — not the base Access Policy, which only governs the yes/no access decision.

## Client-to-Client Remote Assistance and the Endpoint Firewall

If ZCC's [[Traffic Forwarding|Location-Based Policy ruleset]] feature is in use, its endpoint firewall rules can otherwise block *all* inbound traffic by default — including legitimate inbound ZPA client-to-client traffic used for client-based remote assistance. A specific ruleset option, **Allow ZPA Client to Client Traffic**, exists precisely to permit this traffic through even when the rest of the inbound firewall posture is locked down. If remote-assistance-style client-to-client PRA scenarios stop working after a ruleset is deployed, this toggle is the first thing to check.

---

# Design Pattern

```text
Privileged Access Segment (narrow — specific servers/ports only)
        │
Access Policy: scoped to admin group,
               strong posture requirement (MFA + managed + EDR)
        │
Privileged Capabilities: recording ON,
                         clipboard/file transfer restricted per policy
```

Same least-privilege discipline as any other [[Application Segments|Application Segment]] — narrow the segment to exactly the servers/ports needed, don't rely on a broad segment plus policy alone to contain scope.

---

# Granting Guest / Third-Party Users Access via Entra ID

A common real-world requirement: providing PRA (or other clientless ZPA access — Browser Access, User Portal) to external users who are not part of the organization directly, but exist as **guest users** in Entra ID. This is a documented, non-obvious configuration case.

> [!note] Source
> This section is based on a Zscaler community-contributed configuration guide (community.zscaler.com), not official Zscaler product documentation. Treat it as a field-tested pattern rather than a vendor-guaranteed procedure, and verify against current official guidance before relying on it in production.

## The Underlying Problem

ZPA uses the **domain portion of the username** to determine which IdP to authenticate against. Guest users in Entra ID are assigned a "local" UserPrincipalName (UPN) in a specific format:

```text
Real email:        awin.raj@company.com
Guest UPN in IdP:   awin_raj#EXT#@company.onmicrosoft.com
```

If the guest user is provisioned to ZPA without adjusting this, ZPA sees the username as `awin.raj@company.com`, but Entra ID's actual UPN for that account remains the `#EXT#@company.onmicrosoft.com` form. The mismatch causes an **authentication failure to the IdP** — commonly surfacing as a generic `401: Authentication Failed`, not an obviously guest-user-specific error.

This is a distinct failure mode from a policy block: the user never gets far enough for [[Access Policies|Access Policy]] or posture to matter — authentication itself fails first. See [[Authentication]] for where this sits in the general auth flow, and [[Troubleshooting Methodology]] for why distinguishing "auth failure" from "policy block" changes where you look.

## The Fix — Two Parts

**Part 1 — Enable Arbitrary Domains on the ZPA side.**

In the ZPA admin portal: **Authentication → User Authentication → IDP Settings**, edit the IdP configuration and enable **Use with Arbitrary Domains**. Without this, ZPA restricts authentication to only the explicitly defined authentication domain(s) for that IdP — which typically does not include the tenant's `.onmicrosoft.com` domain that guest UPNs use.

**Part 2 — Correct the UPN mapping on the Entra ID side.**

This is the more involved half, and it has two dependent pieces — full mechanics documented in [[Identity Providers]]:

1. **SCIM provisioning mapping** (Entra ID → Provisioning → Mappings): change the Zscaler `userName` attribute to map from Entra ID's `originalUserPrincipalName` rather than the default `userPrincipalName`. This makes Entra ID provision the user to Zscaler using the UPN that existed at invitation time, not the guest's real external email.
2. **SAML Attributes & Claims** (on the ZPA Enterprise Application's Single Sign-On config): change the **Unique User Identifier** claim from `user.userprincipalname` to `user.localuserprincipalname`. This makes Entra ID *assert* the same `#EXT#@company.onmicrosoft.com` form during SAML authentication that was used for SCIM provisioning.

## Why Both Parts Are Necessary

Provisioning (SCIM) and authentication (SAML) are two separate data paths between Entra ID and Zscaler. Fixing only the provisioning mapping means ZPA has the *right* username on file, but Entra ID still *asserts* a different UPN at actual login time — same mismatch, just moved from one side to the other. Both mappings have to agree for the username ZPA has on record to match what Entra ID sends during authentication.

## End State

```text
Username on ZPA  =  UserPrincipalName on Entra ID
```

Once aligned, the guest user authenticates normally through Entra ID SAML to reach PRA, Browser Access, or User Portal resources like any other federated user.

---

# Troubleshooting

## Guest User Gets 401 Authentication Failed
This is the signature symptom of the UPN mismatch above, not a policy or posture problem. Check:
1. Is "Use with Arbitrary Domains" enabled for the IdP?
2. Does the SCIM provisioning mapping use `originalUserPrincipalName`?
3. Does the SAML claim use `user.localuserprincipalname` for Unique User Identifier?

## Session Not Recording Despite Policy
Check Privileged Capabilities specifically — recording is not controlled by the base Access Policy.

## User Reaches PRA but Can't Transfer Files
Clipboard/file transfer are independent toggles under Privileged Capabilities — a user can be fully authorized for the session itself while still restricted on data movement within it.

---

# Related Notes

- [[ZPA]]
- [[Access Policies]]
- [[Application Segments]]
- [[Identity Providers]]
- [[Authentication]]
- [[Troubleshooting Methodology]]
