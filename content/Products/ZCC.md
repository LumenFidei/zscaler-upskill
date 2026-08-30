# Zscaler Client Connector (ZCC)

## Overview

**ZCC** is the endpoint agent that authenticates users, forwards traffic, collects posture, and enforces policy across [[ZIA]], [[ZPA]], and [[ZDX]].

---

> [!important] ZDTE exam weight
> Blueprint objectives landing here:
> - "Given a scenario about **ZCC profile and update management** with required results, identify the options that need to be configured"
> - "Given a scenario about **ZCC and ZEN connectivity monitoring**, identify the action that should be taken based on the monitoring information"
>
> Note the blueprint uses **ZEN** (Zscaler Enforcement Node) — the older term for what is now called Service Edge. Treat them as the same thing on the exam.

---

# Configuration Object Model

Know which object controls what — questions hinge on picking the right one:

|Object|Controls|
|---|---|
|**App Profile**|Which settings apply to which users/devices|
|**Forwarding Profile**|**How** traffic is sent (tunnel type, on/off-network behavior)|
|**Trusted Network**|How "on corporate network" is detected|
|**Posture Profile**|Device trust checks|

**Exam trap:** the Forwarding Profile — not the App Profile — determines tunnel mode and on-network vs off-network behavior. The App Profile *assigns* a forwarding profile to a population.

---

# Forwarding Profile

Defines behavior per network state:

```text
On Trusted Network   → e.g. Tunnel / None / ZPA only
Off Trusted Network  → e.g. Tunnel 2.0
VPN Trusted Network  → behavior when a third-party VPN is active
```

## Tunnel Modes

|Mode|Scope|
|---|---|
|**Tunnel 1.0**|Web ports only (80/443)|
|**Tunnel 2.0**|**All ports and protocols**|
|Tunnel w/ Local Proxy|Tunnel plus local proxy listener|
|Local Proxy only|Proxy-based, no tunnel|
|None|No ZIA forwarding|

**Exam-critical:** non-web protocols reaching the Cloud Firewall requires **Tunnel 2.0**. If a scenario needs SSH/RDP inspected by ZIA and the profile is Tunnel 1.0, that's the root cause.

---

# App Profile

Assigns configuration to populations, evaluated by rule order — first match wins.

Contains:
- Forwarding profile assignment
- Authentication settings (automatic vs interactive login, reauth interval, MFA)
- UI controls (hide UI, read-only, allow logout, notifications, troubleshooting access)
- Logging level
- Update policy
- Platform scoping (Windows, macOS, iOS, Android, Linux)

**Exam trap:** if the wrong settings apply to a user, check App Profile **rule order** and group membership before suspecting the settings themselves.

---

# Update Management

A named blueprint objective. Options that appear in scenarios:

|Setting|Effect|
|---|---|
|**Auto-update**|Client upgrades per policy|
|**Pinned version**|Hold a population on a specific build|
|**Staged rollout**|Phase upgrades across groups|
|**Restart behavior**|Whether upgrade forces/defers restart|
|**User-initiated update**|Allow the user to trigger|

Design pattern the exam favors:

```text
Pilot group    → latest version, auto-update
Production     → pinned to a validated version
                 then promoted after pilot validation
```

If a scenario requires "validate before wide deployment," the answer is a staged rollout with a pilot App Profile — not disabling updates globally.

---

# ZCC and ZEN Connectivity Monitoring

Blueprint objective: interpret monitoring data and **identify the action**.

## What to Read

- Tunnel state (up/down, which version)
- Which ZEN / Service Edge the client is connected to
- Authentication state
- Latency to the ZEN
- Policy download / last sync time
- Posture result

## Interpretation → Action

|Observation|Likely cause|Action|
|---|---|---|
|Tunnel down, auth failed|IdP or SAML issue|Check [[Identity Providers]], cert validity|
|Tunnel up, high latency to ZEN|Suboptimal ZEN selection or ISP|Investigate path, see [[ZDX]] / [[Path Analysis]]|
|Connected to a distant ZEN|Geo/DNS resolution issue|Verify egress IP and location mapping|
|Policy stale / not downloading|Connectivity or auth|Re-authenticate, check cloud reachability|
|On-network but tunneling anyway|Trusted Network Detection failing|Check TND criteria — see [[Trusted Network Detection]]|
|Off-network but not tunneling|TND falsely reporting trusted|Check for overlapping gateway/DNS criteria|

**Diagnostic principle:** distinguish *authentication* problems from *connectivity* problems from *policy* problems. The client status view tells you which.

---

# Authentication

Methods: SAML, Entra ID, Okta, Ping, ADFS, certificate-based, Kerberos.

Settings that matter in scenarios:
- Automatic vs interactive login
- Reauthentication interval
- MFA enforcement
- Machine tunnel (pre-logon connectivity for domain operations)

**Machine Tunnel** is worth knowing: it establishes connectivity before user logon, used for domain-joined boot-time operations and patching. It authenticates the *device*, not a user.

## Automatic Username Population and Device Registration (App Supportability)

Configured at **Infrastructure → Connectors → Client → App Supportability**:

- **Automatically Populate Username for IdP Authentication** — autofills the IdP login form via JavaScript, or via the `login_hint` SAML attribute (sent to both ZIA and ZPA). If both are enabled, `login_hint` wins.
- **Username Format** (login_hint only, ZCC 4.9+ Windows) — SAM Account Name vs. User Principal Name (UPN); UPN falls back to SAM Account Name on non-domain-joined devices.
- **Register device with ZPA IdP Username** — registers the device using the ZPA-authentication username rather than the enrollment username.

Full reasoning for *why* these matter — including the Entra ID guest-user scenario they directly address — lives in [[Identity Providers]].

---

# Split Tunnel

Exclusions configured in the forwarding profile:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Typical: local subnets, VoIP, video conferencing, management networks, certificate-pinned apps.

Fewer exclusions = more inspection coverage. Every bypass is a visibility gap — expect this framing in "which design is better" questions.

---

# Diagnostics

Available from the client:
- Tunnel, authentication, policy, DNS, connectivity logs
- Gateway reachability tests
- DNS resolution tests
- Log export for support

---

# Programmatic Device Enrollment Queries (Legacy Mobile Admin Portal API)

Beyond client-side diagnostics, enrolled device state can be queried programmatically via the **ZCC Mobile Admin Portal's public API** — useful for auditing, compliance reporting, or feeding device state into other automation without logging into the console directly.

> [!note] Source and framework distinction
> Based on a Zscaler community-contributed guide (community.zscaler.com), not official product documentation — treat as field-tested rather than vendor-guaranteed. More importantly: this API uses **`apiKey`/`secretKey`** authentication, which is the **legacy** framework, not [[OneAPI]]'s OAuth2/ZIdentity model. Don't assume the OneAPI credential pattern (`client_id`/`client_secret`/`vanity_domain`/`cloud`) applies here — this is a separate, ZCC Mobile Admin Portal–specific API surface.

## Authentication Flow

```text
POST request with:
{
  "apiKey": "string",
  "secretKey": "string"
}
        │
        ▼
JWT (JSON Web Token) returned
        │
        ▼
JWT used to authorize subsequent device-list requests
```

## Fetching the Device List

The device list endpoint supports filtering by **registration type/state** via a `registrationTypes` parameter:

|Value|State|
|---|---|
|0|All states except Removed|
|1|Registered|
|3|Removal pending|
|4|Unregistered|
|5|Removed|
|6|Quarantined|

For a report of "everything currently relevant" (i.e. excluding devices already fully removed), `registrationTypes: 0` is the value to use — this is the pattern the source guide itself demonstrates.

**Common use case:** a scheduled Postman or scripted job pulling device state periodically for a compliance/audit report, without a human manually exporting from the console each time. See [[API Automation Use Cases]] for the same trigger → automated call → result pattern applied elsewhere, though note this specific example runs on the legacy API, not OneAPI.

---

# Troubleshooting

## Client Not Forwarding
1. Authentication state
2. Forwarding profile and tunnel mode
3. App Profile assignment (rule order!)
4. Trusted Network Detection result
5. Split tunnel exclusions matching unintentionally

## Non-Web Traffic Not Inspected
Tunnel 1.0 in use — needs Tunnel 2.0.

## Wrong Policy Applied
App Profile ordering or group membership.

## Device Enrollment API Returns Auth Error
Confirm you're using the legacy `apiKey`/`secretKey` pair for the Mobile Admin Portal specifically — this is a different credential from any OneAPI/ZIdentity client registered for ZIA/ZPA automation. The two are not interchangeable.

---

# Related Notes

- [[Traffic Forwarding]]
- [[Tunnel 2.0]]
- [[App Profiles]]
- [[Trusted Network Detection]]
- [[Device Posture]]
- [[Authentication]]
- [[OneAPI]]
- [[API Automation Use Cases]]
