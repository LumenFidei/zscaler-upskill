# User Experience Score

## Overview

The **User Experience Score** is a summarized metric produced by [[ZDX]] that represents the overall quality of a user's digital experience by combining device, network, and application performance signals.

Related:

- [[ZDX]]
- [[Path Analysis]]
- [[Endpoint Monitoring]]

---

# Purpose

Provide a single, easy-to-interpret indicator of experience quality, rather than requiring administrators to manually correlate dozens of raw metrics.

---

# Contributing Factors

## Device Health

- CPU utilization
- Memory utilization
- Disk usage

## Network Quality

- Latency
- Jitter
- Packet loss
- Wi-Fi signal strength

## Application Performance

- Response time
- Availability
- Transaction latency

---

# Scoring Model

```text
Device Health
      +
Network Quality
      +
Application Performance
      =
User Experience Score
```

Scores are typically presented on a normalized scale (e.g. 0-100), with thresholds defining good, degraded, and poor experience bands.

---

# Using the Score

## Trend Analysis

Track score changes over time to identify degrading experience before users report issues.

## Comparative Analysis

Compare scores across:

- Locations
- Departments
- Device types
- Applications

---

# Root Cause Drill-Down

A low User Experience Score is a starting point, not an endpoint. Administrators typically drill down using [[Path Analysis]] and [[Endpoint Monitoring]] to identify which layer is responsible.

```text
Low Score
    │
    ▼
Device Issue?
    │
    ▼
Network Issue?
    │
    ▼
Application Issue?
```

---

# Best Practices

- Baseline normal score ranges per user population
- Alert on sustained score degradation, not single-point dips
- Correlate score drops with ZIA/ZPA logs for full context
- Review scores regularly, not only during incidents

---

# Related Notes

- [[ZDX]]
- [[Path Analysis]]
- [[Endpoint Monitoring]]
