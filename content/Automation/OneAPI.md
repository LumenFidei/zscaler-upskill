# OneAPI

## Overview

**OneAPI** is Zscaler's unified programming interface for the entire platform — a single API framework replacing the older model of separate, product-specific APIs for ZIA, ZPA, ZDX, and others.

Related:

- [[ZIdentity]]
- [[API Automation Use Cases]]
- [[Zscaler SDKs and Tooling]]

---

> [!important] ZDTE relevance — read this note before the others
> Unlike other notes in this vault, I don't have a confirmed, verbatim ZDTE blueprint line naming "OneAPI" specifically — it's a newer capability, and the blueprint objectives I've quoted elsewhere in this vault were extracted from the published exam guide directly. What's confirmed is domain-level: this entire cluster of files (OneAPI, ZIdentity, Automation Use Cases, SDKs and Tooling) sits squarely under **Automation and Optimization (11%)** — the domain with the thinnest existing coverage in this vault before these files were added. Treat the content here as strong domain preparation rather than memorized answers to specific quoted questions.

---

# The Problem It Replaces

Before OneAPI, each Zscaler product had its own API, its own base URL, and often its own authentication model:

```text
Legacy model:
ZIA API   → zsapi.zscalertwo.net/api/v1/   (own auth)
ZPA API   → separate endpoint              (own auth)
ZDX API   → separate endpoint              (own auth)
```

Automating across products meant juggling multiple credential types, multiple client libraries, and inconsistent request/response conventions. OneAPI collapses this into one model.

> [!note] Legacy APIs are not (yet) decommissioned
> As of the most recent public confirmation, Zscaler has not set a decommissioning date for the legacy per-product APIs. Migrating to OneAPI is recommended, not currently mandatory. Existing legacy-API integrations continue to function.

---

# Three Architectural Pillars

## 1. A Common Endpoint

All Zscaler resources are reachable through a single base URL:

```text
api.zsapi.net
```

One endpoint to configure, rather than a different base URL per product.

## 2. Unified Authentication via ZIdentity

Every OneAPI client authenticates the same way, regardless of which product it's calling — via an API client registered in **ZIdentity**, using OAuth 2.0. See [[ZIdentity]] for the full mechanics.

## 3. Global Distribution

Requests are automatically routed to the nearest regional endpoint, regardless of which Zscaler cloud the tenant lives on — the client doesn't need to know or hardcode regional routing logic.

---

# Product Coverage

OneAPI currently spans:

- **ZIA** — Zscaler Internet Access
- **ZPA** — Zscaler Private Access
- **ZDX** — Zscaler Digital Experience
- **ZCC** — Zscaler Client Connector (management functions)
- **ZTC** — Zscaler Zero Trust Cloud (cloud workload / multi-cloud onboarding)

New functions are added on an ongoing basis as product features roll out, with the stated goal of eventually reaching functional parity with what's configurable manually through the admin console.

> [!note] API parity, not UI parity — yet
> At any given time, some features configurable in the admin console UI may not yet be exposed via the public API. This isn't a bug in a specific SDK or Terraform provider — it reflects what Zscaler has actually published. See [[Zscaler SDKs and Tooling]] for how this affects Terraform specifically.

---

# Legacy API vs OneAPI

| |**Legacy**|**OneAPI**|
|---|---|---|
|Endpoint|Per-product, per-cloud|Single: `api.zsapi.net`|
|Auth|Per-product credentials|Unified, via ZIdentity|
|Auth flow|Varies by product|OAuth 2.0 Client Credentials|
|Routing|Manual/hardcoded per cloud|Automatic, nearest region|
|Status|Not decommissioned, not the recommended path forward|Recommended for new integrations|

---

# Government Cloud Caveat

OneAPI and ZIdentity historically did **not** support the `zscalergov` / `zscalerten` (GOVUS) clouds — those environments required the legacy API framework. This has since been addressed via unified `gov` / `govus` cloud values in current provider versions, but it's a detail that has changed over time. If working with a government tenant, verify current support status rather than assuming either way — this is exactly the kind of platform detail that shifts between releases.

---

# Why It Matters Operationally

- **Least privilege extends to automation.** API clients get scoped RBAC roles just like human admins — a client automating URL category updates shouldn't hold ZPA policy-write access it never uses.
- **One client library, one auth flow, all products.** Reduces the credential-management sprawl of the legacy model.
- **Cross-product orchestration becomes practical.** A single authenticated client can act across ZIA and ZPA in the same script — this is what makes the orchestration use cases in [[API Automation Use Cases]] realistic rather than theoretical.

---

# Troubleshooting

## API Client Can't Authenticate
Check [[ZIdentity]] first — nearly all OneAPI auth failures trace back to the client registration or RBAC role assignment there, not to the product-side API itself.

## A Feature Isn't Available via the API
Confirm it's actually published in OneAPI yet — new capabilities roll out incrementally, and a UI-only feature is a real, current limitation, not necessarily a client bug.

## Requests Going to the Wrong Region
Confirm the client isn't hardcoding a regional endpoint from legacy-API-era code — OneAPI's global routing only works correctly when the client is pointed at `api.zsapi.net`, not a specific regional URL.

---

# Related Notes

- [[ZIdentity]]
- [[API Automation Use Cases]]
- [[Zscaler SDKs and Tooling]]
- [[Identity Providers]]
- [[ZIA]]
- [[ZPA]]
