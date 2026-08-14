# ZIdentity

## Overview

**ZIdentity** is the identity platform underlying Zscaler's [[OneAPI]] authentication model — every OneAPI client is a registered identity within ZIdentity, authenticated via OAuth 2.0.

Related:

- [[OneAPI]]
- [[Identity Providers]]
- [[Authentication]]

---

> [!important] Don't confuse this with Identity Providers.md
> [[Identity Providers]] covers **human** identity — SAML/SSO federation for users logging into ZCC, the admin console, or ZPA-published applications via Entra ID, Okta, etc. **ZIdentity is different**: it's specifically where **API client** identities live for machine-to-machine automation. A human user's SSO login and an API client's OAuth credential are two entirely separate identity concerns, even though both ultimately govern access to the same Zscaler platform. Conflating them is a real, easy mistake — an admin fixing a human SSO issue in the wrong place, or looking for an automation credential in the human IdP config, is a predictable failure mode.

---

# How an API Client Gets Created

The RBAC role a client needs is defined **per product first**, then surfaced into ZIdentity automatically:

```text
1. In the product (e.g. ZIA or ZPA):
   Administration → Role Management → Add API Role
        │
        ▼
2. Return to the ZIdentity Admin UI:
   Integration menu → API Resources
        │
        ▼
3. Click the View icon next to "Zscaler APIs"
        │
        ▼
4. The newly created role should appear under the
   relevant product dropdown
        │
        ▼
5. If it doesn't appear automatically:
   click "Sync Now" in the API Resources menu
   to force an on-demand sync
        │
        ▼
6. Register an API client in ZIdentity, assign it
   the synced role(s)
        │
        ▼
7. Client authenticates using OAuth 2.0
   Client Credentials flow
```

**The "Sync Now" button matters operationally** — a role created product-side doesn't always appear instantly in ZIdentity. If a freshly created role is missing when configuring a client, that's the first thing to check before assuming the role wasn't actually created.

---

# Authentication Flow

OneAPI uses the **OAuth 2.0 Client Credentials** flow — machine-to-machine, with no human user interaction or login prompt involved:

```text
API Client
    │
    ▼  presents client_id + client_secret
ZIdentity (authorization server)
    │
    ▼  issues access token
API Client
    │
    ▼  presents token
api.zsapi.net (OneAPI)
    │
    ▼
Requested resource (ZIA / ZPA / ZDX / ZCC / ZTC)
```

## Required Credential Values

|Value|Purpose|
|---|---|
|`client_id`|Identifies the registered API client|
|`client_secret`|Client's authentication secret|
|`vanity_domain`|Tenant-specific domain identifier|
|`cloud`|Which Zscaler cloud the tenant lives on|

These four values are what every official SDK and Terraform provider expects for OneAPI authentication — see [[Zscaler SDKs and Tooling]] for concrete configuration examples.

---

# RBAC and Least Privilege

The same least-privilege principle that governs human access (see [[Access Policies]] and [[ZPA Policy Design]]) applies identically to API clients:

- Create narrowly-scoped roles per automation purpose rather than one broad "admin" API client used everywhere
- A client automating ZIA URL category updates should not also hold ZPA policy-write permissions it never exercises
- Because API clients are fully managed identities in ZIdentity, they carry complete audit trails and behavioral visibility — same governance expectations as a human admin account, not a lesser standard

---

# Troubleshooting

## Client Authentication Fails
1. Confirm `client_id` / `client_secret` are current — not expired or rotated without updating the consuming automation
2. Confirm `vanity_domain` and `cloud` values match the actual tenant
3. Confirm the client's assigned role actually grants the scope the automation needs

## Role Created But Client Still Denied
1. Check whether the role has actually synced into ZIdentity — use "Sync Now" under API Resources
2. Confirm the role was assigned **to the specific client**, not just created and left unassigned

## Works for ZIA, Fails for ZPA (or vice versa)
RBAC roles are granted **per product** — a client with a ZIA role has no implicit ZPA access. Each product's role must be created and assigned separately, even though authentication itself is unified.

## Government Cloud Tenant
Confirm current OneAPI/ZIdentity support status for `gov`/`govus` clouds before assuming standard OneAPI setup applies — this has been an evolving area. See the caveat in [[OneAPI]].

---

# Related Notes

- [[OneAPI]]
- [[Identity Providers]]
- [[Authentication]]
- [[Access Policies]]
- [[Zscaler SDKs and Tooling]]
