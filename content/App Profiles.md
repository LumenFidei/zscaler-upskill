# App Profiles

## Overview

**App Profiles** are ZCC configuration objects that determine which forwarding, authentication, and behavior settings apply to a given user, group, or device population.

App Profiles are the mechanism by which administrators assign different ZCC behavior to different populations from a single client package.

Related:

- [[ZCC]]
- [[Tunnel 2.0]]
- [[Traffic Forwarding]]
- [[Trusted Network Detection]]

---

# Purpose

Without App Profiles, all ZCC users would receive identical forwarding and authentication behavior. App Profiles allow differentiated configuration for:

- Employees vs. contractors
- Managed vs. BYOD devices
- Regional user populations
- Pilot vs. production rollouts

---

# Assignment Logic

```text
User / Device
      │
      ▼
Identity Attributes
      │
      ▼
App Profile Match
      │
      ▼
Applied Configuration
```

App Profiles are typically assigned based on group membership, OU, or device platform.

---

# Common Components

## Forwarding Profile

Determines how traffic is routed (Tunnel 2.0, PAC, VPN-based, etc.). See [[Tunnel 2.0]].

## Trusted Network Detection Settings

Defines how ZCC detects trusted/corporate networks. See [[Trusted Network Detection]].

## Authentication Settings

- Automatic vs. interactive login
- Reauthentication interval
- MFA enforcement

## UI Controls

- Hide UI
- Read-only UI
- Allow logout
- Notification behavior

## Platform-Specific Settings

Separate profiles may exist for Windows, macOS, iOS, and Android.

---

# Design Patterns

## Population Separation

```text
Employees Profile
Contractors Profile
Executives Profile
```

## Staged Rollout

```text
Pilot Profile
      │
      ▼
Expanded Profile
      │
      ▼
Production Profile
```

---

# Troubleshooting

## Wrong Configuration Applied

Check:

1. Group/OU membership
2. Profile assignment order
3. Platform-specific overrides
4. Cached configuration on the endpoint

## Profile Not Downloading

Check:

1. Authentication status
2. Connectivity to Zscaler cloud
3. Client version compatibility

---

# Best Practices

- Keep the number of profiles manageable
- Use descriptive, consistent naming
- Test new profiles with a pilot group before wide deployment
- Document which population maps to which profile

---

# Related Notes

- [[ZCC]]
- [[Tunnel 2.0]]
- [[Trusted Network Detection]]
- [[Traffic Forwarding]]
