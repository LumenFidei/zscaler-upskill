# Device Posture

## Overview

**Device Posture** evaluates the security state of an endpoint before and during access, adding device trust to identity-based decisions.

Related:

- [[Access Policies]]
- [[ZCC]]
- [[Zero Trust]]

---

> [!important] ZDTE exam weight
> Posture appears throughout the blueprint:
> - "Identify the steps to create access policies... incorporating **posture checks** (e.g., device posture, health score, AV, etc.)"
> - Least-privilege objectives explicitly name "device posture" as a policy input
> - Troubleshooting objectives on private-app blocks frequently resolve to posture failures
>
> The detailed check catalogue below is sourced from official Zscaler Client Connector product documentation (Configuring Device Posture Profiles) — this is the actual admin console mechanics behind the checks named generically elsewhere in the blueprint.

---

# Where Posture Profiles Are Configured

```text
Policies → Common Configuration → Resources → Device Posture → Add Device Posture
```

Every profile requires a **Name** and one or more **Platforms** (Windows, macOS, Linux, Android, iOS). Platform selection determines which posture types are even available — several checks are single-platform or explicitly excluded from multi-platform profiles (see the File Path gotcha below).

---

# How Posture Attaches to Policy

```text
1. ZCC collects posture signals from the endpoint
        ▼
2. Posture Profile defines required checks
        ▼
3. Access Policy rule references the posture profile
        ▼
4. Decision at connection + continuous re-evaluation
```

**Exam trap:** a posture profile that exists but isn't referenced by any Access Policy rule enforces nothing. If a scenario says "posture is configured but non-compliant devices still get in," check the rule binding — not the profile contents.

---

# Machine Tunnel and Partner Tenant Evaluation

Two settings control whether a posture type applies **before user login**, not just during the normal user session:

## Apply to Windows Machine Tunnel
Evaluates posture on the pre-Windows-login machine tunnel (ZCC 3.9.0+). Applies only to: Client Certificate, Certificate Trust, File Path, Registry Key, Firewall, Full Disk Encryption, Domain Joined, AzureAD Domain Joined, Server Validated Client Certificate, OS Version, and Zscaler Client Connector Version. **Not every posture type is eligible for machine-tunnel evaluation** — EDR/AV detection checks and the CrowdStrike ZTA scores are notably absent from this list, since those generally require a logged-in user context to query.

## Apply to macOS Machine Tunnel
Same concept for macOS (ZCC 4.5.0+), with a narrower eligible list: CrowdStrike ZTA Score, Full Disk Encryption, File Path, Firewall, Domain Joined, OS Version, and Zscaler Client Connector Version.

## Apply When Added as Partner Tenant
ZCC 4.6+ Windows only. Evaluates posture on the Private Access tunnel used to connect to a **partner tenant** — relevant in multi-tenant or partner-access ZPA scenarios.

---

# Evaluation Frequency

**Frequency (In Minutes)** — ZCC 4.4+ Windows, 4.5+ macOS — controls how often ZCC re-evaluates each posture check. Range: 2–15 minutes, default 15.

> [!warning] Some checks bypass the frequency setting entirely
> **Process Check, Detect Carbon Black, Detect CrowdStrike, Detect SentinelOne, and Detect Microsoft Defender** evaluate **immediately** the moment posture changes on the device, regardless of the configured frequency — in addition to their normal scheduled re-evaluation. If a scenario describes near-instant posture revocation for one of these five specific checks but the Frequency field is set to 15 minutes, that's expected behavior, not a misconfiguration — these five are architected for immediate detection.

---

# Posture Check Quick Reference

|Posture Type|Platforms|Key Mechanic|
|---|---|---|
|Certificate Trust|Win/macOS/Linux/Android/iOS|Device must trust an uploaded internal root/intermediate CA cert|
|File Path|Windows, macOS|Checks a specific file path exists — **unavailable if profile targets more than one platform**|
|Registry Key|Windows|Path or Value match; runs in **user context**|
|Client Certificate|Win/macOS/Linux|Full cert + private key validation; **Android support ending at ZCC 5.0**|
|Firewall|Windows, macOS|At least one of public/private/domain profile active; **ignores third-party firewalls**|
|Full Disk Encryption|Win/macOS/Android/Linux|Linux variant requires a File Path to check a specific partition|
|Domain Joined|Windows, macOS|Matches a specified domain/workgroup|
|Process Check|Win/macOS/Linux|Process path + signer certificate thumbprint, optional version|
|Detect Carbon Black / CrowdStrike / SentinelOne|Windows, macOS|Presence check; thumbprint-based with an installed+running fallback|
|CrowdStrike ZTA Score|Windows, macOS|Combined OS+sensor risk score, 1–100, requires CrowdStrike ZTA enabled|
|CrowdStrike ZTA Device OS Score|Windows, macOS|OS-specific score only — **distinct from the combined score above**|
|CrowdStrike ZTA Sensor Setting Score|Windows, macOS|Sensor-setting score only — a **third, separate** CrowdStrike score type|
|Detect Microsoft Defender|Win/macOS/Linux|Linux variant needs an executable path|
|Detect Antivirus|Windows, macOS|**Asymmetric**: AV Name optional on Windows, mandatory on macOS|
|OS Version|Win/macOS/Linux/Android|Per-platform edition/build/distribution selection, multiple entries supported|
|Jamf Detection / Jamf Risk Level|macOS|Presence + Secure/Low/Medium/High risk tier|
|AzureAD Domain Joined|Windows|Matches a specific Tenant ID; combinable with Domain Joined (hybrid join)|
|Server Validated Client Certificate|Windows|Challenge-response cert validation, distinct from plain Client Certificate|
|Ownership Variable|Android, iOS|MDM-pushed variable, set at install time|
|Unauthorized Modification|Android, iOS|Jailbreak/root detection|
|Zscaler Client Connector Version|Windows, macOS|Minimum client version gate|

---

# Certificate-Based Checks

## Certificate Trust
Broadest-platform check. Upload a Base64-encoded `.pem`/`.cer` root or intermediate CA certificate; the device must trust it. Enabled by default — contact Zscaler Support to disable. **iOS-specific requirement:** the certificate must be trusted **directly** by the device; an intermediate certificate does not work on iOS, unlike other platforms.

## Client Certificate
Requires the client certificate (with private/public key pair) to actually be present in the user's certificate store, issued by the CA uploaded to the profile.

> [!warning] Android support ending
> As of ZCC version 5.0, **Zscaler is ending support for Client Certificate posture on all Android devices**. If a scenario or deployment plan assumes certificate-based posture for Android going forward, this is no longer a valid design once that version is in effect.

Options:
- **Non-Exportable Private Key** — checks whether the cert's private key can be exported. Posture **fails** if this is enabled and the key turns out to be exportable (the check verifies non-exportability is actually true).
- **Perform CRL Check** — detects revoked certificates via the Certificate Revocation List; a revoked cert fails posture.
- **Client Certificate Template Information** — Windows/macOS only, must match the issuing template.

CRL caching (ZCC 4.8+ Windows) can be configured locally (0–30 min interval) rather than relying on the default Microsoft API cache. Zscaler recommends this specifically when: the CRL server is unstable, the CRL server is hosted as a **Private Access application** (creating ambiguous caching behavior tied to PRA access expiration — see [[PRA]]), or near-real-time revocation checking is required. Requires contacting Zscaler Support to enable.

## Server Validated Client Certificate
Windows-only, ZCC 4.4+. Distinct mechanic from plain Client Certificate: uses a **challenge-response** flow — the server sends a challenge, the client locates its private key and the root CA on the chain, signs the challenge, and returns it. Certificate must be in either the current user or local computer personal store.

**Performance note:** uploading more than 50 certificates can add roughly 10 seconds to server response time.

---

# File and Registry Checks

## File Path
Windows or macOS. Enter a path (e.g. `C:\Program Files(x86)\Example\AV.txt`) — the device must have that path present.

> [!warning] Multi-platform gotcha
> **If a posture profile targets more than one platform, File Path is not available as an option at all.** Certificate Trust is the documented alternative for multi-platform profiles. This is easy to miss when generalizing a working single-platform profile to cover more devices.

## Registry Key
Windows only. Match Type is either **Path** (must begin with `HKEY`, e.g. `HKEY_CURRENT_USER\Software\Zscaler\App`) or **Value** (also requires Name and Data fields, e.g. Name=`ZNW_State`, Data=`CONNECTING`).

> [!warning] Runs in user context
> The registry check executes in the **user's** context, not the system's. If your permission model blocks standard users from reading system-level keys (e.g. under `HKLM`), the check will fail for reasons unrelated to the actual posture — you must create the registry key somewhere the user's own context can actually read it.

---

# Network and Encryption Checks

## Firewall
Windows or macOS. Checks all three Windows firewall profiles (public, private, domain) — **at least one** active passes the check.

> [!warning] Third-party firewalls don't count
> ZCC does not check for third-party firewall presence. A device with a reputable third-party firewall but Windows Firewall fully disabled **fails** this check. Don't assume "the device has a firewall" is equivalent to "this check passes."

## Full Disk Encryption
Windows, macOS, Android natively; Linux from ZCC 1.4+. Checks BitLocker/FileVault/equivalent status.

**Linux is structurally different:** rather than a simple enabled/disabled check, Linux requires a **File Path** value, and the check verifies whether the disk or partition containing that path is encrypted — a path-based check, not a device-wide one.

---

# Endpoint Security Detection Checks

## The Shared Version + Exact Match Pattern

Several checks (Process Check, Detect Carbon Black, Detect CrowdStrike, Detect SentinelOne, Detect Microsoft Defender, Zscaler Client Connector Version) share the same optional version-matching mechanic on Windows (ZCC 4.9+ for the security-vendor checks):

```text
Version field: up to 3 entries, 2-to-4-version-level format (e.g. "1.8" or "1.8.0.456")
Exact Match checkbox:
   Selected → only the exact version(s) listed pass
   Cleared  → the listed version(s) AND anything later also pass
```

Example: entering `1.8.0.456, 1.9.0.789` with Exact Match **cleared** passes all versions 1.8.0.456 and later, including the entire 1.9.x line — not just the two literal versions typed in. This "cleared = floor, not a fixed set" behavior is the part most likely to trip someone up reading a scenario quickly.

## Detect Carbon Black / CrowdStrike / SentinelOne
ZCC 2.1.2+ Windows/macOS. Each validates against a list of signer certificate thumbprints for the vendor's specific executable (RepMgr.exe, CSFalconService.exe, SentinelAgent.exe on Windows; equivalent binaries on macOS). Exact thumbprint values are vendor/version-specific and change over time — check current Zscaler documentation rather than hardcoding a thumbprint list into a runbook.

> [!note] Installed-and-running fallback
> On sufficiently recent versions (ZCC 4.5+ Windows, ZCC 4.3+ macOS, varies slightly per vendor), if **no thumbprint matches**, the posture check **still passes** as long as the security product is installed and actively running/enabled. This fallback exists specifically so a thumbprint list going stale doesn't silently break posture for everyone — worth knowing before assuming a thumbprint mismatch is a hard failure.

## Detect Microsoft Defender
ZCC 2.1.1+ Windows/macOS, 1.4+ Linux. Checks for Microsoft Defender ATP. **Linux requires an executable path** value — same path-based pattern as Linux Full Disk Encryption, distinct from the thumbprint approach used on Windows/macOS.

## Detect Antivirus
ZCC 3.6+ Windows/macOS.

> [!warning] Asymmetric requirement between platforms
> **Windows:** AV Name is optional — leave it blank and ZCC detects *any* antivirus running. **macOS: AV Name is mandatory**, and must include the system extension name specifically (use the `systemextensionsctl` command-line tool to find the exact name). Configuring this identically across both platforms in a single mental model is a common mistake — macOS requires more specific input than Windows for the equivalent check.

Windows also supports **Check if AV Definitions Are Up-to-Date** as an additional toggle.

---

# CrowdStrike ZTA — Three Distinct Scores

Easy to conflate; these are **three separately configurable checks**, not one setting with options:

|Check|Compares Against|
|---|---|
|**CrowdStrike ZTA Score**|Combined overall score (OS + sensor settings)|
|**CrowdStrike ZTA Device OS Score**|OS-specific score only|
|**CrowdStrike ZTA Sensor Setting Score**|Sensor-setting score only|

All three: enter a **Minimum Score** (1–100); ZCC passes the check if CrowdStrike's reported score for that specific dimension is **greater than or equal to** your entered minimum. All three require the CrowdStrike ZTA feature to be explicitly enabled within CrowdStrike itself — contact CrowdStrike Support, not Zscaler, to turn that on. Version requirements also differ slightly (CrowdStrike ZTA Score: ZCC 3.4+; the OS/Sensor-specific variants: ZCC 4.6+ Windows, 4.8+ macOS).

---

# OS and Domain Checks

## OS Version
Version requirements vary by platform (Win/macOS: 3.6+; Linux: 1.4+; Android: 3.7+; **no iOS support**). Configuration differs meaningfully per platform:

|Platform|Configuration Pattern|
|---|---|
|Windows|Select OS Edition + OS Build, click Add OS — repeatable for multiple editions|
|macOS|Select OS Version entry, click Add OS — repeatable|
|Linux|Select Distribution + enter OS version, click Add OS — repeatable|
|Android|Select OS Edition + Calendar-picked patch month, click Add OS — repeatable|

## Domain Joined
Windows or macOS. Enter the domain/workgroup to match.

## AzureAD Domain Joined
Windows only. Enter the Tenant ID (a GUID) to match against.

> [!note] Hybrid join is a real, distinct state
> A device can be **only** Domain Joined, **only** AzureAD Domain Joined, or **both** (hybrid join). These aren't mutually exclusive — a posture profile can require either or both states depending on the design. Worth remembering given how much of this vault already covers Entra ID (see [[Identity Providers]], [[PRA]]) — this is the device-trust-side equivalent of that same identity platform.

## Jamf Detection / Jamf Risk Level
macOS only, ZCC 4.3+. Two separate sub-checks: **Jamf Detection** (is Jamf running at all) and **Jamf Risk Level** (Jamf's own Secure/Low/Medium/High risk tier for the device).

> [!warning] MDM unenrollment doesn't fully stop the Jamf daemon
> Removing an MDM profile from a device does **not** guarantee the Jamf daemon stops running — it can persist. To fully remove it on unenrollment, admins must add the command `sudo jamf -removeFramework` to Jamf Pro. Skipping this step can leave a device reporting stale Jamf posture data after it's supposedly been unenrolled.

---

# Mobile-Specific Checks

## Ownership Variable
Android (ZCC 1.7.1+) or iOS (1.8.2+). An alphanumeric value (≤32 characters) pushed to the device via MDM **at install time** — the variable must be set through MDM deployment configuration, not entered manually after the fact.

## Unauthorized Modification
Android or iOS. Detects jailbreak/root. Known issues exist on old versions specifically (iOS 1.5.2, Android 1.5.0) — check current release notes if running legacy mobile client versions.

---

# Client Version Gate

## Zscaler Client Connector Version
Windows/macOS, ZCC 4.8+. Same Version + Exact Match mechanic as the security-vendor checks above, but with its own constraint: **only accepts version values 4.8 and higher** — can't be used to gate against older client baselines.

---

# Posture Profiles by Population

## Standard Workforce
Managed device, encryption enabled, AV active

## Developers
Managed device, EDR active, approved operating system

## Privileged Administrators
Managed device, EDR active, encryption enabled, certificate validation, MFA

---

# Access Outcomes

```text
Fully compliant     → full access
Partially compliant → limited access, subset of apps, step-up auth
Non-compliant       → deny, quarantine, remediation page
```

---

# Continuous Verification

Trust is not permanent — posture re-evaluates per the Frequency setting (or immediately, for the five checks noted above), and access is revoked or restricted if a device falls out of compliance mid-session.

---

# Troubleshooting

## Device Fails Posture Assessment — General

|Symptom|Likely check|
|---|---|
|Recently reimaged device denied|Certificate missing, not yet MDM-enrolled|
|Intermittent denial|EDR disconnected from its own console|
|One OS version affected|OS version/build threshold|
|Denied after security tool update|Thumbprint mismatch — check for installed-and-running fallback eligibility first|
|Works on some devices only|Encryption not enabled on all volumes|

## Multi-Platform Profile Missing File Path Option
Expected behavior, not a bug — switch to Certificate Trust.

## Registry Key Check Fails Despite Key Existing
Check whether the key requires system-level (`HKLM`) access that the user's own context can't read — the check runs as the user, not the system.

## Posture Passes Despite No Thumbprint Match
Check whether the client version qualifies for the installed-and-running fallback — this is expected behavior for Carbon Black/CrowdStrike/SentinelOne detection on sufficiently recent ZCC versions, not a bypassed check.

## macOS Antivirus Check Fails With a Blank AV Name
Expected — unlike Windows, macOS requires an explicit AV Name including the system extension name. Use `systemextensionsctl` to find it.

## Stale Jamf Posture After Unenrollment
Confirm `sudo jamf -removeFramework` was run in Jamf Pro during unenrollment — otherwise the daemon may still be reporting.

## Workflow
```text
Posture failure
      │
      ▼
Identify failed check (console/logs)
      │
      ▼
Validate actual endpoint state
      │
      ▼
Remediate
      │
      ▼
Retest — confirm re-evaluation
```

---

# Design Best Practices

- **Test in monitoring mode before enforcing**
- Start simple: managed + encrypted + AV
- Add EDR, certificates, browser posture gradually
- Keep requirements achievable — unrealistic checks cause mass lockouts
- Separate populations into distinct profiles
- Know which checks are Machine-Tunnel-eligible if pre-login posture matters for your design
- Don't assume identical configuration works across platforms — several checks (Detect Antivirus, Client Certificate on Android, File Path with multiple platforms) have real platform-specific asymmetries

---

# Key Takeaways

1. Identity alone is not sufficient for trust.
2. A posture profile enforces nothing until referenced by an Access Policy rule.
3. Not every posture type works on every platform — several have hard platform restrictions or version floors worth checking before designing around them.
4. Five specific checks (Process Check, Carbon Black, CrowdStrike, SentinelOne, Microsoft Defender detection) bypass the Frequency setting and evaluate immediately on change.
5. CrowdStrike ZTA has three distinct, separately-configured score types — don't conflate them.
6. Continuous verification revokes access when posture degrades mid-session.

---

# Related Notes

- [[Access Policies]]
- [[ZPA]]
- [[ZCC]]
- [[PRA]]
- [[Identity Providers]]
- [[EDR]]
- [[Certificate Authentication]]
- [[Zero Trust]]
- [[Common Gotchas]]
