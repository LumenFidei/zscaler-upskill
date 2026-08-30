# Identity Providers

## Overview

An **Identity Provider (IdP)** authenticates users and supplies the identity attributes Zscaler services use to make access decisions.

Related:

- [[Authentication]]
- [[ZCC]]
- [[ZPA]]
- [[PRA]]
- [[Policy Evaluation]]

---

# Authentication Flow

```text
User
 │
 ▼
Zscaler (ZCC / ZPA / ZIA)
 │
 ▼
Identity Provider
 │
 ▼
Authentication Result (SAML assertion)
 │
 ▼
Policy Evaluation
```

---

# Common Identity Providers

## Microsoft Entra ID
The most common enterprise cloud identity platform in Zscaler deployments. Used for SSO, MFA, conditional access, and — a common real-world requirement — federating **guest/external users**, covered in detail below.

## Okta
Cloud identity provider. Federation, user lifecycle, MFA.

## Ping Identity
Enterprise federation platform.

## Active Directory Federation Services (ADFS)
Legacy on-premises federation solution.

---

# Authentication Methods

## SAML
The most common enterprise federation method. Provides identity assertion, group information, and the specific attribute/claim values covered below.

## Certificate Authentication
Validates identity using a certificate rather than (or in addition to) a password. See [[Certificate Authentication]].

## MFA
Additional verification layer. See [[MFA]].

---

# Domain-Based IdP Selection

A mechanic worth understanding on its own, since it's the root cause behind the guest-user problem below: Zscaler services determine **which IdP to authenticate a user against based on the domain portion of the username**. A username's domain has to match a domain the IdP is configured to handle, or authentication fails before any policy is even evaluated.

This is normally invisible — most users' email domain matches their organization's registered authentication domain exactly. It becomes a real problem for identities whose domain doesn't cleanly match anything explicitly configured — most commonly, **guest users**.

---

# Guest and External User Federation (Entra ID)

A documented, non-obvious configuration case: providing access to clientless ZPA resources — [[PRA]], Browser Access, User Portal — to third-party users who exist in Entra ID as **guest users** rather than native tenant members.

> [!note] Source
> Based on a Zscaler community-contributed configuration guide (community.zscaler.com), not official Zscaler product documentation. Treat as a field-tested pattern rather than a vendor-guaranteed procedure; verify against current official guidance before production use.

## The Problem

Guest users in Entra ID are assigned a "local" UserPrincipalName in a specific, distinctive format:

```text
Real email (how the guest actually logs in):
    awin.raj@company.com

Guest's actual UPN inside the IdP:
    awin_raj#EXT#@company.onmicrosoft.com
```

If the guest is provisioned to ZPA (or ZIA) without adjustment, the service sees the username as the guest's real email — `awin.raj@company.com` — while Entra ID's actual UPN for that identity remains the `#EXT#@...onmicrosoft.com` form. These two don't match, and since domain-based IdP selection (above) depends on the username matching what the IdP expects, authentication fails.

**Symptom:** a generic `401: Authentication Failed` — nothing in the error message points specifically at "guest user" or "UPN mismatch," which is what makes this hard to diagnose without knowing the pattern in advance.

## The Fix

### Part 1 — ZPA Side: Allow Arbitrary Domains

Most organizations never explicitly register their tenant's `.onmicrosoft.com` domain as an authentication domain — there's usually no reason to, for native users. But guest UPNs live in that domain.

**ZPA admin portal → Authentication → User Authentication → IDP Settings** → edit the relevant IdP configuration → enable **Use with Arbitrary Domains**.

Without this, ZPA restricts authentication to only the domains explicitly defined for that IdP — which typically excludes the `.onmicrosoft.com` domain guest UPNs use, regardless of anything else being configured correctly.

### Part 2 — Entra ID Side: Align Two Separate Attribute Mappings

Provisioning (SCIM) and authentication (SAML) are two independent data paths between Entra ID and Zscaler. **Both** need to agree on the UPN format, or the mismatch just moves from one side to the other.

**2a. SCIM Provisioning Mapping**

Entra ID → the ZPA/ZIA Enterprise Application → **Provisioning → Mappings**. Change the mapping so the Zscaler `userName` attribute pulls from Entra ID's **`originalUserPrincipalName`** rather than the default `userPrincipalName`.

```text
Before: Zscaler userName  ←  user.userprincipalname
                              (guest's real email — wrong for this case)

After:  Zscaler userName  ←  user.originaluserprincipalname
                              (the UPN created at invite time — correct)
```

This makes Entra ID provision the user to Zscaler using the UPN that existed when the guest was originally invited, not their real external email address.

**2b. SAML Attributes & Claims**

Entra ID → the same Enterprise Application → **Single Sign-On → Attributes & Claims**. Change the **Unique User Identifier** claim from `user.userprincipalname` to **`user.localuserprincipalname`**.

```text
Before: Unique User Identifier  ←  user.userprincipalname
After:  Unique User Identifier  ←  user.localuserprincipalname
```

This makes Entra ID *assert* the same `#EXT#@company.onmicrosoft.com` form during actual SAML login that was used for SCIM provisioning in step 2a. Skipping this step is the most common way this fix ends up "half-applied" — the provisioning side is corrected, but Entra ID still authenticates the user with the other UPN format, so the mismatch persists.

## End State

```text
Username on ZPA  =  UserPrincipalName on Entra ID
```

Once both mappings agree, the guest authenticates normally and reaches whatever clientless ZPA resource they've been assigned to — PRA, Browser Access, or User Portal — the same as any other federated user.

## Prerequisite Check

Before troubleshooting the mapping details above, confirm the basics are actually in place:

- The guest user genuinely exists in Entra ID as a **guest user** (not pending invitation)
- The guest is assigned to the relevant Enterprise Application (ZPA/ZIA)

---

# Official ZCC Mechanisms for Username Consistency

Distinct from (and complementary to) the community-sourced Entra ID guest-user fix above, ZCC itself has **official, built-in** mechanisms addressing the same underlying class of problem — the username ZCC/ZPA has on record not matching what the IdP actually expects during SAML authentication. Configured at **Infrastructure → Connectors → Client → App Supportability**.

## Automatically Populate Username for IdP Authentication

Two non-exclusive methods for pre-filling the username field on the IdP's own login form:

|Method|Mechanism|
|---|---|
|**Using JavaScript**|ZCC injects JavaScript into the IdP page during the SAML workflow to autofill the username field|
|**Using `login_hint` SAML attribute**|ZCC sends a `login_hint` parameter to **both** ZIA and ZPA during authentication, which pass it to the IdP as a SAML Subject (`NAMEID`) to pre-populate the username|

> [!warning] Precedence when both are enabled
> If both methods are enabled simultaneously, **`login_hint` takes precedence over JavaScript**.

## Username Format (login_hint only, ZCC 4.9+ Windows)

When using `login_hint`, you can also control *which* username format gets sent:

|Format|Behavior|
|---|---|
|**SAM Account Name**|Combines the SAM account name with the registry domain (e.g. `jsmith` + `safemarch.com` → `jsmith@safemarch.com`)|
|**User Principal Name**|Sends the signed-in identity from a domain-joined device. **Falls back to SAM Account Name if the device isn't domain-joined.**|

This exists specifically to reduce SSO disruptions when the IdP expects a UPN rather than a SAM-style name — which is directly relevant to the guest-user UPN mismatch problem above. Where the community-guide fix works by correcting Entra ID's own attribute mappings, **Username Format: User Principal Name** works from the ZCC side by ensuring the *correct* username format is what gets sent in the first place. Depending on the exact failure, one or both may be needed — they're not mutually exclusive fixes for the same class of problem.

## Register Device with ZPA IdP Username

A separate, related toggle: normally a device registers in the Zscaler Admin Console using the username it was **enrolled** with. This setting instead registers the device using the username entered for **ZPA authentication** specifically.

**Why this matters for the guest-user scenario:** if a guest's enrollment identity and their ZPA-authenticated identity differ (exactly the situation the UPN mismatch above describes), this setting determines *which* of those two usernames the device ends up associated with in the console — directly affecting how the device shows up in reporting, and potentially which posture/access policies correctly match it.

---

# Identity in ZPA

Used for application access, policy matching, and least-privilege enforcement. See [[Access Policies]].

# Identity in ZIA

Used for internet policy, reporting, and user-based controls.

---

# Troubleshooting

## Authentication Failure — General
1. IdP availability
2. SAML configuration
3. Certificate validity
4. User attributes
5. Group mapping

## Authentication Failure — Guest / External User Specifically
If the failing user is a guest in Entra ID (or a similar federated-external pattern in another IdP), don't stop at the general checklist above — go straight to the domain-based IdP selection mismatch covered above. A generic `401` from a guest user is the specific signature to recognize, not a random SAML config problem to debug from scratch. Also check whether **Username Format** (User Principal Name vs. SAM Account Name) is configured to match what the IdP actually expects — this is a ZCC-side lever independent of the Entra ID attribute-mapping fix, and either misconfigured piece alone can reproduce the same symptom.

---

# Best Practices

- Centralize identity
- Require MFA
- Keep groups clean
- Audit access regularly
- For guest/external user access: document the "Use with Arbitrary Domains" + dual-mapping fix explicitly somewhere your team will find it — this is exactly the kind of one-time, non-obvious configuration that gets forgotten and re-discovered painfully the next time a third-party user needs onboarding

---

# Related Notes

- [[Authentication]]
- [[ZPA]]
- [[PRA]]
- [[Device Posture]]
- [[Policy Evaluation]]
- [[Access Policies]]
