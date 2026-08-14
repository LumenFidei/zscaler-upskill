# SaaS Security

## Overview

**SaaS Security** covers the controls ZIA applies to protect the use of sanctioned and unsanctioned SaaS applications, spanning access control, data protection, and shadow IT visibility.

Related:

- [[ZIA]]
- [[DLP]]
- [[ZIA Policy Design]]

---

# Purpose

SaaS adoption moves data and workflows outside traditional network boundaries. SaaS Security extends visibility and control to these applications regardless of user location.

---

# Common SaaS Platforms

- Microsoft 365
- Google Workspace
- Salesforce
- ServiceNow
- Box
- Dropbox

---

# Core Controls

## Access Control

Restrict which users and groups can access specific SaaS applications, and from which devices or locations.

## Data Protection

Apply [[DLP]] policies to data moving into, out of, and between SaaS applications.

## Shadow IT Detection

Identify unsanctioned SaaS applications in use across the organization by analyzing traffic patterns.

---

# Shadow IT Discovery Flow

```text
Outbound Traffic
        │
        ▼
Application Identification
        │
        ▼
Sanctioned?
        │
  ┌─────┴─────┐
  │           │
 Yes          No
  │           │
Monitor    Flag / Restrict
```

---

# Tenant Restriction

Common control allowing access to a corporate SaaS tenant (e.g. corporate Microsoft 365) while blocking personal tenants of the same application.

---

# Policy Design Considerations

## Sanctioned vs. Unsanctioned Applications

Maintain a clear inventory and policy distinction between approved and unapproved SaaS tools.

## Tenant Restrictions

Apply tenant-level controls for major platforms to prevent data movement to personal accounts.

## DLP Integration

Ensure SaaS-bound traffic is in scope for relevant DLP rules.

---

# Troubleshooting

## Unsanctioned App Not Being Flagged

Check:

1. Application signature/classification coverage
2. SSL Inspection coverage for the destination
3. Shadow IT reporting scope

## Legitimate SaaS Access Blocked

Check:

1. Tenant restriction configuration
2. Application category classification
3. User/group policy scope

---

# Best Practices

- Maintain an up-to-date sanctioned application list
- Apply tenant restrictions to major productivity suites
- Review shadow IT reports regularly to identify new unsanctioned tools
- Pair SaaS visibility with DLP for full data protection coverage

---

# Related Notes

- [[ZIA]]
- [[DLP]]
- [[ZIA Policy Design]]
- [[SSL Inspection]]
