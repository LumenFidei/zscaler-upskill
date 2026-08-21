# ZCC Mobile Admin Portal API

## Overview

The **ZCC Mobile Admin Portal** exposes a public API that allows programmatic retrieval of enrolled device data, without logging into the portal directly. This supports documentation, auditing, and automation use cases where a manual export isn't practical.

This note is part of the Automation cluster alongside [[OneAPI]], [[ZIdentity]], and [[API Automation Use Cases]].

> [!important] ZDTE tie-in
> Automation and Optimization is the smallest blueprint domain by weight, but API-driven device inventory retrieval is a concrete, testable use case — know the authentication flow (API key/secret → JWT) and that device state filtering is done via `registrationTypes`.

Related:

- [[ZCC]]
- [[API Automation Use Cases]]
- [[Zscaler SDKs and Tooling]]

---

# Use Case

Retrieve a list of currently enrolled ZCC devices programmatically, for:

- Documentation
- Auditing
- Automation pipelines

Tooling: API calls made via Postman (or any HTTP client).

---

# Authentication Flow

```text
API Key + Secret Key
        │
        ▼
Authenticate to Zscaler API
        │
        ▼
JSON Web Token (JWT)
        │
        ▼
Authorized API Requests
```

Authentication request body shape:

```json
{
  "apiKey": "string",
  "secretKey": "string"
}
```

The returned JWT is then used to authorize subsequent calls, including the device list request.

---

# Fetching the Device List

Device results can be filtered by registration state using the `registrationTypes` parameter.

## Registration Type Values

|Value|Meaning|
|---|---|
|0|All states except Removed|
|1|Registered|
|3|Removal pending|
|4|Unregistered|
|5|Removed|
|6|Quarantined|

A common pattern is `registrationTypes = 0` to retrieve a complete inventory of active-relevant devices while excluding fully removed ones.

---

# Workflow Summary

```text
1. POST credentials (apiKey + secretKey)
        │
        ▼
2. Receive JWT
        │
        ▼
3. GET device list with Authorization header
        │
        ▼
4. Filter results by registrationTypes as needed
```

---

# Troubleshooting

## Authentication Fails

Check:

1. API key / secret key validity
2. Correct request body structure
3. Whether the JWT has expired before making the device list call

---

## Device List Missing Expected Devices

Check:

1. `registrationTypes` filter value — a narrow filter (e.g. `1` only) will exclude quarantined or pending-removal devices
2. Whether the device was ever fully registered vs. still pending

---

# Best Practices

- Treat the API key/secret like any other credential — store securely, never hardcode in shared scripts
- Use `registrationTypes = 0` for general auditing so devices in transitional states aren't silently excluded
- Re-authenticate for a fresh JWT on a schedule rather than assuming long-lived token validity
- Pair this API with scheduled automation (e.g. cron, CI pipeline) for recurring device inventory reports

---

# Related Notes

- [[ZCC]]
- [[API Automation Use Cases]]
- [[OneAPI]]
- [[ZIdentity]]
- [[Zscaler SDKs and Tooling]]
