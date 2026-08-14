# Branch Connector

## Overview

**Branch Connector** is the term used throughout this vault's wikilinks to refer to Zscaler's site-connectivity appliance. The full reference note for this product lives at [[ZBC]] (Zscaler Branch Connector).

This note covers deployment sizing, connector grouping, and design details that supplement [[ZBC]].

Related:

- [[ZBC]]
- [[Traffic Forwarding]]
- [[App Connectors]]
- [[Site Connectivity]]

---

# Relationship to ZBC

Branch Connector and Zscaler Branch Connector (ZBC) refer to the same product. Use [[ZBC]] for core architecture, objectives, and troubleshooting; use this note for sizing and grouping guidance.

---

# Sizing Considerations

Factors that determine how many Branch Connectors a site requires:

- Number of users at the site
- Aggregate bandwidth requirements
- Number of applications discovered
- High availability requirements

---

# Connector Grouping

## Purpose

Group Branch Connectors so that policy and routing can be applied consistently across a site or region.

## Example Grouping

```text
Region: EMEA
   ├── Branch Connector Group: London
   ├── Branch Connector Group: Frankfurt
   └── Branch Connector Group: Dublin
```

---

# Deployment Sizing Pattern

```text
Small Branch:   1-2 Connectors
Medium Branch:  2-4 Connectors
Large Site / DC: 4+ Connectors, load balanced
```

---

# Design Considerations

- Deploy at least two connectors per site for failover
- Separate connector groups by environment (production, DR, regional)
- Align sizing with expected peak bandwidth, not average bandwidth
- Reassess sizing after application discovery is complete

---

# Related Notes

- [[ZBC]]
- [[Traffic Forwarding]]
- [[App Connectors]]
- [[Site Connectivity]]
- [[Zero Trust]]
