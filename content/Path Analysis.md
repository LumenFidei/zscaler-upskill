# Path Analysis

## Overview

**Path Analysis** is a [[ZDX]] capability that visualizes the network path a user's traffic takes from their device to an application, identifying where latency or loss is introduced along the way.

Related:

- [[ZDX]]
- [[User Experience Score]]
- [[Service Edge]]

---

# Purpose

When a user reports slow performance, Path Analysis helps determine whether the issue is local, in transit, or at the destination.

---

# Path Segments

```text
Device
  │
  ▼
Gateway
  │
  ▼
ISP
  │
  ▼
Zscaler Service Edge
  │
  ▼
Application
```

Each hop is measured independently, allowing administrators to isolate the segment responsible for degraded performance.

---

# Metrics Collected Per Hop

- Latency
- Packet loss
- Jitter (where applicable)

---

# Common Findings

## Local Network Issue

High latency or loss between device and gateway, unrelated to Zscaler or the application.

## ISP Issue

Degradation observed between gateway and the internet path, before reaching the Zscaler [[Service Edge]].

## Zscaler Path Issue

Degradation observed at or immediately around the Service Edge hop.

## Application Issue

Good performance up to the Service Edge, with degradation only on the final hop to the application.

---

# Workflow

```text
User Reports Issue
        │
        ▼
Run Path Analysis
        │
        ▼
Identify Degraded Hop
        │
        ▼
Direct Investigation to Correct Team (Endpoint, ISP, Zscaler, App Owner)
```

---

# Best Practices

- Run Path Analysis at the time of the reported issue, not after the fact, when possible
- Compare against a known-good baseline path
- Use alongside [[User Experience Score]] to prioritize investigation
- Escalate ISP-segment issues externally rather than treating them as Zscaler issues

---

# Related Notes

- [[ZDX]]
- [[User Experience Score]]
- [[Service Edge]]
- [[Endpoint Monitoring]]
