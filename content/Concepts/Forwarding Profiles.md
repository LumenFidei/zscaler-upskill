# Forwarding Profiles

## Overview

A **Forwarding Profile** tells Zscaler Client Connector (ZCC) how to handle the traffic it intercepts on an endpoint: what traffic to capture, when to send it to Zscaler, what to bypass, and how that behavior should differ by device platform.

Forwarding Profiles are the client-side foundation underneath [[Traffic Forwarding]] — get them wrong, and problems that look like policy issues are often actually forwarding issues.

> [!important] ZDTE tie-in
> Forwarding Profiles sit squarely in Architecture and Design and Implementation and Deployment. A classic exam trap: symptoms of a login failure or app breakage are frequently misdiagnosed as a policy problem when the real root cause is a misconfigured Forwarding Profile.

Related:

- [[ZCC]]
- [[Traffic Forwarding]]
- [[Tunnel 2.0]]

---

# What a Forwarding Profile Defines

```text
Forwarding Profile
        │
        ├── What traffic to capture
        ├── When to send it to Zscaler
        ├── What to bypass
        └── How behavior differs by platform
```

---

# Platform Scope

Forwarding behavior defines client-side endpoint handling for:

- Windows
- macOS
- iOS
- Android
- Linux (support varies by environment/version)

---

# Universal Forwarding Capabilities

## Traffic Capture & Bypass

```text
What to Capture
      │
      ▼
HTTP/HTTPS Application & Web Traffic

What to Bypass
      │
      ▼
Trusted Local Applications
```

---

## Connection Decision

```text
When to Send Traffic to Zscaler
        │
        ▼
   On-Network?  ──── No ──── Off-Network → Connect to ZIA/ZPA
        │
       Yes
        │
        ▼
Do Not Intercept Trusted Sites (e.g. Microsoft, Local DNS)
```

---

## Route Optimization

```text
What to Bypass
        │
        ▼
Internal Applications, Local DNS
        │
        ▼
Optimized Connection Path
        │
        ▼
Per-Platform Device Routing
(Windows / macOS / iOS / Android / Linux)
```

---

# Why This Matters

|Design Quality|Result|
|---|---|
|Well designed|Users get consistent Zscaler protection and reliable access to internet and private apps|
|Poorly designed|Traffic may bypass inspection, apps may break, or users see login/access issues that look like policy problems|

---

# Troubleshooting

## Traffic Not Being Inspected

Check:

1. Whether the destination is in a bypass rule
2. Whether the endpoint is being classified as on-network incorrectly
3. Forwarding Profile assignment for that platform

---

## App Breaks After Profile Change

Check:

1. Whether the app's traffic was moved from bypass to capture (or vice versa)
2. Platform-specific overrides that may differ from the default profile
3. Per-app tunneling / complex rule exceptions

---

## Symptom Looks Like a Policy Block But Isn't

Check:

1. Forwarding Profile capture/bypass rules before assuming a URL/policy block
2. Whether traffic ever reached Zscaler at all (forwarding issue vs. policy issue)

---

# Best Practices

- Standardize Forwarding Profiles per platform rather than improvising per user
- Keep bypass lists minimal and documented
- Treat "looks like a policy issue" symptoms as a forwarding-profile check first
- Review [[Tunnel 2.0]] behavior alongside Forwarding Profile design, since Tunnel 2.0 is the primary enforcement mechanism for profiles that forward to Zscaler

---

# Related Notes

- [[ZCC]]
- [[Traffic Forwarding]]
- [[Tunnel 2.0]]
- [[Troubleshooting Methodology]]
