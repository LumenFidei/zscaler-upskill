# Troubleshooting Methodology

## Overview

A structured method for isolating Zscaler issues by identifying **where** in the path the failure occurs.

---

> [!important] ZDTE exam weight — 15%
> Blueprint objectives, all three phrased the same way:
> - "Given a scenario where specific locations and/or user types cannot access some internet applications, investigate the **PAC file** for mismatched logic"
> - "Given a scenario in which legitimate access to an app is being prevented by a block policy, and evidence relating to the failure, **identify where the blocking happened** and the steps to resolve it"
> - "Given a scenario in which legitimate access to a **private** app is being prevented by a block policy... identify where the blocking happened and the steps to resolve it"
>
> The exam skill is **localizing the block**, not listing generic steps. Every question hands you evidence and asks: which layer did this?

---

# Localizing the Block — The Core Skill

Match the **symptom and evidence** to the layer:

|Evidence|Blocking layer|
|---|---|
|Zscaler block page shown in browser|**ZIA URL Filtering** — user reached the proxy|
|DNS resolution fails / NXDOMAIN|**DNS Control** — blocked before connection|
|Connection times out, no block page|Firewall, routing, or tunnel — not URL policy|
|Certificate error|**SSL Inspection** — cert trust or pinned app|
|Non-web protocol fails, browsing fine|**Cloud Firewall** or Tunnel 1.0 (non-web not forwarded)|
|App not listed / "not authorized" in ZPA|**ZPA Access Policy** or missing segment|
|ZPA app resolves but won't connect|**Connector** health, Server Group, or routing|
|Works on-network, fails remotely (or vice versa)|**Trusted Network Detection** or location/sublocation policy|
|Only some users or some sites affected|**PAC file logic** — see [[PAC Files]]|

**The single most useful discriminator:** did the user get a **Zscaler block page** or a **timeout/failure**?

- Block page → traffic reached Zscaler and policy denied it. Fix the policy.
- Timeout/no response → traffic never got there, or was dropped below the policy layer. Fix forwarding, DNS, firewall, or connectivity.

Candidates lose points by proposing a policy change for a connectivity failure.

---

# Internet Access Failure (ZIA path)

```text
User cannot reach an internet app
          │
          ▼
Block page shown?
   ├── Yes → ZIA policy. Check web logs for the matching rule.
   │          URL Filtering? Firewall? DLP? Which rule name?
   │
   └── No  → Did DNS resolve?
              ├── No  → DNS Control policy or DNS forwarding
              └── Yes → Is traffic forwarded?
                         ├── No  → ZCC/tunnel/PAC (see below)
                         └── Yes → Firewall rule or destination-side issue
```

## PAC-Specific Investigation

Blueprint calls this out explicitly. When *specific locations or user types* fail:

1. Determine which PAC file that population actually loads — they often differ
2. Walk the logic top-down for the failing host
3. Find the **first** matching line — that's the one that decided
4. Look for a broad rule shadowing a specific exception
5. Check for a missing default return

Full detail in [[PAC Files]].

---

# Private Access Failure (ZPA path)

```text
User cannot reach a private app
          │
          ▼
Is the app in an Application Segment?
   ├── No  → Not published. Create/extend the segment.
   └── Yes → Does an Access Policy permit this user?
              ├── No  → Policy gap, or rule order, or group membership
              └── Yes → Did posture pass?
                         ├── No  → Device Posture failure (see below)
                         └── Yes → Connector layer:
                                    health, Server Group, DNS, routing
```

Work the object hierarchy in order — see [[Application Segments]].

## Posture Failures

If access is denied despite a correct policy, check which posture check failed:

- Unsupported OS version
- AV/EDR not running or disconnected
- Encryption disabled
- Certificate expired or missing
- Device unmanaged

See [[Device Posture]].

---

# The Five Layers

## Layer 1: Endpoint
ZCC status, version, authentication state, local connectivity, posture result

## Layer 2: Identity
Authentication success, group membership, MFA, SAML attributes

## Layer 3: Policy
App Profile, URL/Firewall/DNS/DLP rules, ZPA Access Policy, **rule order**

## Layer 4: Network
DNS, tunnel state, firewall, routing, PAC logic

## Layer 5: Application
Availability, connector health, application configuration

---

# Rule Order Is a Recurring Root Cause

Across ZIA policy, ZPA Access Policy, DNS Control, and PAC files, evaluation is **top-down, first match wins**. A correct rule placed below a broader rule never executes.

When a policy "looks right but doesn't work," check order before checking content. Audit logs will show rule-order changes — see [[Logging and Reporting]].

---

# Evidence Collection

Before diagnosing, collect:

- User identity and group membership
- Device and OS
- Exact timestamp
- Destination (FQDN, IP, port)
- Exact error message or screenshot
- Location / on- vs off-network
- Relevant logs

---

# Root Cause Process

```text
Symptom
  │
  ▼
Evidence  ← localize the layer first
  │
  ▼
Hypothesis
  │
  ▼
Test (change one variable)
  │
  ▼
Root Cause
```

---

# Best Practices

- Localize before you change anything
- Change one variable at a time
- Use logs to confirm which rule matched — don't infer it
- Document fixes

---

# Related Notes

- [[PAC Files]]
- [[Logging and Reporting]]
- [[Application Segments]]
- [[Device Posture]]
- [[Traffic Forwarding]]
- [[Policy Evaluation]]
