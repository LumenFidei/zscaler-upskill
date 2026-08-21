# PRA & Clientless Access for Entra ID Guest Users

## Overview

Organizations often need to give **clientless applications** — those configured via Browser Access, User Portal, or [[PRA]] — to third-party users who aren't part of the organization directly, but exist as **guest users** in Microsoft Entra ID (Entra ID).

By default, this breaks in a specific way: ZPA authenticates based on the domain in the username, and a guest user's identity in Entra ID doesn't cleanly map to that expectation without configuration changes on both ZPA and Entra ID.

> [!important] ZDTE tie-in
> This is a Security and Compliance / Identity scenario that combines [[Identity Providers]] concepts with [[PRA]] and clientless ZPA access. The core exam-relevant fact: ZPA determines which IdP to authenticate against based on the domain portion of the username — guest UPNs break this assumption unless corrected.

Related:

- [[PRA]]
- [[Identity Providers]]
- [[ZPA]]
- [[Authentication]]

---

# The Problem

When a third-party user is invited into Entra ID as a guest, their UserPrincipalName (UPN) typically looks like:

```text
Original sign-in identity:   awin.raj@company.com
Entra ID Guest UPN:          awin_raj#EXT#@company.onmicrosoft.com
```

ZPA uses the **domain** in the username to determine which IdP to authenticate against. If the guest is provisioned to ZPA using their original email domain rather than their actual Entra ID guest UPN, authentication fails — commonly surfacing as a `401: Authentication Failed` error.

```text
User Signs In
      │
      ▼
ZPA Reads Username Domain
      │
      ▼
Determines IdP to Authenticate Against
      │
      ▼
Mismatch: ZPA Username ≠ Entra ID UPN
      │
      ▼
401 Authentication Failed
```

---

# The Fix

## Step 1: Enable Arbitrary Domains on the IdP

On the ZPA Admin Portal:

```text
Authentication > User Authentication > IdP Settings
      │
      ▼
Edit IdP Configuration
      │
      ▼
Enable "Use with Arbitrary Domains"
```

This allows ZPA to authenticate users whose domain doesn't match a pre-registered authentication domain — necessary because most organizations haven't registered their `.onmicrosoft.com` domain as an authentication domain.

---

## Step 2: Correct the Provisioning Mapping (Entra ID)

Under **Provisioning > Mappings** in Entra ID, the ZPA username attribute mapping needs to be changed so it sources from the guest's actual UPN rather than a derived value.

```text
Zscaler Attribute:  username
      │
      ▼
Change mapping FROM: userPrincipalName (default)
Change mapping TO:   originalUserPrincipalName
```

This forces Entra ID to send the original UPN created when the guest was invited, rather than a value that won't match what ZPA expects.

---

## Step 3: Correct SAML Attributes & Claims (Enterprise Application)

Under the ZPA Enterprise Application in Entra ID:

```text
Single sign-on > Attributes & Claims
      │
      ▼
Unique User Identifier
      │
      ▼
Change FROM: user.userprincipalname
Change TO:   user.localuserprincipalname
```

This ensures Entra ID passes the correct `#EXT#@company.onmicrosoft.com`-style UPN during SAML authentication, matching what ZPA now expects after Step 2.

---

# End State

```text
Username on ZPA  =  UserPrincipalName on Entra ID
```

Once both the provisioning mapping and the SAML attribute mapping are corrected, guest users authenticate successfully and gain access to their assigned clientless applications (Browser Access, User Portal, PRA).

---

# Troubleshooting

## Guest User Gets 401: Authentication Failed

Check:

1. Whether "Use with Arbitrary Domains" is enabled on the IdP config
2. Whether the Provisioning > Mappings `username` attribute points to `originalUserPrincipalName`
3. Whether Attributes & Claims `Unique User Identifier` points to `user.localuserprincipalname`

---

## Guest User Not Appearing as Assignable

Check:

1. Guest user is actually present in Entra ID (invited and accepted)
2. Guest user is assigned to the Zscaler Private Access Enterprise Application
3. SCIM provisioning sync has completed (check SCIM sync logs for the user)

---

# Best Practices

- Make both the provisioning mapping change and the SAML attribute change together — one without the other still breaks authentication
- Document the `.onmicrosoft.com` domain behavior for any organization onboarding B2B/guest users, since this is a common first-time gotcha
- Validate with a single pilot guest user before rolling the mapping change out broadly, since it affects how all guest identities are resolved

---

# Related Notes

- [[PRA]]
- [[Identity Providers]]
- [[ZPA]]
- [[Authentication]]
- [[ZPA Policy Design]]
