# Zscaler SDKs and Tooling

## Overview

The practical "how do I actually call this" landscape for [[OneAPI]] — official SDKs, Terraform providers, and a couple of community tools worth knowing apart from the official ones.

Related:

- [[OneAPI]]
- [[ZIdentity]]
- [[API Automation Use Cases]]

---

# Official Python SDK

Zscaler publishes an official Python SDK supporting **both** OneAPI and the legacy per-product APIs through a dual-client design.

## Key Characteristics

- Single OAuth2 HTTP client for OneAPI; separate legacy clients remain available for tenants not yet migrated to ZIdentity
- Requires **Python 3.10+**
- Semantic versioning, with release notes published per version
- Built-in intelligent retry handling for request failures

## Choosing Which Client to Use

```text
Tenant migrated to ZIdentity?
   ├── Yes → use the OneAPI client
   └── No  → use the corresponding Legacy client
```

## A Documented Exception Worth Knowing

Not everything is available through OneAPI yet. As of the most recent SDK documentation, **AI Guard**'s policy detection endpoints specifically require the Legacy AI Guard client regardless of whether the tenant has otherwise migrated to ZIdentity — every other AI Guard resource is OneAPI-only. This is a concrete example of the "API parity, not UI parity — yet" caveat from [[OneAPI]]: migration status isn't always a single on/off switch for every capability.

---

# Official Terraform Providers

Zscaler publishes and maintains dedicated Terraform providers per product:

|Provider|Manages|
|---|---|
|`zscaler/zia`|ZIA configuration|
|`zscaler/zpa`|Application segments, segment groups, server groups, access policies|
|`zscaler/ztc`|Zero Trust Cloud (multi-cloud onboarding, forwarding policy)|

## Authentication

All three authenticate via OneAPI's OAuth 2.0 Client Credentials flow, configured through environment variables:

```bash
export ZSCALER_CLIENT_ID="<client_id>"
export ZSCALER_CLIENT_SECRET="<client_secret>"
export ZSCALER_VANITY_DOMAIN="<vanity_domain>"
export ZSCALER_CLOUD="<cloud>"
```

Or directly in the provider block:

```hcl
provider "zpa" {
  client_id      = "[ZSCALER_CLIENT_ID]"
  client_secret  = "[ZSCALER_CLIENT_SECRET]"
  vanity_domain  = "[ZSCALER_VANITY_DOMAIN]"
  zscaler_cloud  = "[ZSCALER_CLOUD]"
  customer_id    = "[ZPA_CUSTOMER_ID]"
}
```

These four credential values are exactly the ones described in [[ZIdentity]] — every official tool consumes the same underlying identity, just through different configuration surfaces.

## Backward Compatibility

Current provider versions (v4.0.0+) support OneAPI, but retain backward compatibility with the legacy API framework — the recommended path for tenants not yet migrated to ZIdentity. Legacy mode requires setting `use_legacy_client` explicitly; it isn't the silent default once a provider supports OneAPI.

## The Parity Caveat, Concretely

Terraform providers only implement what's actually published via the public API. A feature visible in the admin console UI but not yet exposed through the API cannot be represented in Terraform — this is a reflection of API coverage, not a provider bug. Worth checking current provider docs before assuming a specific console setting is Terraform-manageable.

---

# zscaler-terraformer

A separate, official CLI tool distinct from the Terraform *providers* above — its job is **reverse-generating** Terraform configuration files from an **existing** ZIA/ZPA tenant.

## Why It Matters

For a tenant configured manually over time (the common real-world state, not a greenfield IaC rollout), `zscaler-terraformer` produces a starting `.tf` representation of what's already deployed — the practical on-ramp to bringing an existing environment under Terraform management, rather than starting from a blank file and trying to match production by hand.

Supports the same OneAPI authentication as the providers, with legacy-framework fallback available via the `use_legacy_client` attribute for tenants not yet on ZIdentity.

---

# Postman Collection

Zscaler maintains an official OneAPI documentation/collection on the Postman API Network — useful for interactively exploring endpoints and testing calls before writing automation code against them.

---

# A Community Tool Worth Knowing — And Distinguishing From the Official SDK

**pyZscaler** is a Python SDK for Zscaler's APIs, but it is explicitly **not affiliated with or supported by Zscaler**. It's community-maintained, built on the RESTfly framework, and offers its own conveniences (dot-notation access to JSON responses, automatic CamelCase-to-snake_case conversion).

> [!warning] Don't confuse this with the official SDK
> The naming similarity between "pyZscaler" (community) and Zscaler's official Python SDK is a real, easy point of confusion — the same category of mix-up covered generally in [[Common Gotchas]]. Before building production automation on either one, confirm which you're actually looking at: official support channels and update cadence differ meaningfully between an officially-maintained SDK and a community project.

---

# Tool Selection Summary

```text
Need to explore/test the API interactively?
        → Postman collection

Writing custom automation logic in Python?
        → Official Python SDK (OneAPI client)

Managing configuration declaratively, version-controlled?
        → Official Terraform providers (zia / zpa / ztc)

Bringing an EXISTING tenant under Terraform management?
        → zscaler-terraformer (reverse-generates config)

Evaluating a community Python SDK?
        → Confirm it's what you think it is first (see warning above)
```

---

# Troubleshooting

## Terraform Plan Shows an Unmanageable Setting
Confirm the setting is actually published via the public API yet — see the parity caveat above. Not a provider bug.

## Terraform Provider Rejects Credentials
Confirm all four OneAPI values (`client_id`, `client_secret`, `vanity_domain`, `cloud`) are current and match the tenant. See [[ZIdentity]] troubleshooting for the underlying auth chain.

## SDK Behaves Differently Than Documentation Describes
Confirm which SDK is actually in use — official vs. community — before assuming a documentation mismatch is a bug.

---

# Related Notes

- [[OneAPI]]
- [[ZIdentity]]
- [[API Automation Use Cases]]
- [[Common Gotchas]]
