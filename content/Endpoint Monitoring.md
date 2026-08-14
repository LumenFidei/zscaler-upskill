# Endpoint Monitoring

## Overview

**Endpoint Monitoring** is the [[ZDX]] capability responsible for collecting device-level health and performance telemetry to support root cause analysis and experience scoring.

Related:

- [[ZDX]]
- [[User Experience Score]]
- [[Device Posture]]

---

# Purpose

Determine whether a poor user experience originates from the device itself, rather than the network or application.

---

# Metrics Collected

## Hardware

- CPU utilization
- Memory utilization
- Disk utilization

## Operating System

- Version
- Build
- Patch level

## Connectivity

- Wi-Fi quality
- Signal strength
- Gateway health

## Process Activity

- Resource-intensive processes
- Background application load

---

# Endpoint Monitoring Flow

```text
Device Agent (ZCC)
        │
        ▼
Telemetry Collection
        │
        ▼
ZDX Cloud
        │
        ▼
Device Health Dashboard
```

---

# Distinguishing Endpoint vs. Other Layers

```text
Poor Experience Reported
           │
           ▼
Check Endpoint Metrics
           │
     ┌─────┴─────┐
     │           │
Device Healthy  Device Degraded
     │           │
Investigate    Root Cause
Network/App    Identified
```

---

# Common Root Causes Identified

- High CPU consumption from a background process
- Insufficient memory causing application slowness
- Outdated OS or drivers affecting network performance
- Poor Wi-Fi signal strength

---

# Relationship to Device Posture

While [[Device Posture]] focuses on security compliance (encryption, AV, management status), Endpoint Monitoring focuses on performance health. The two are complementary but answer different questions.

---

# Best Practices

- Baseline normal resource utilization per device type
- Alert on sustained resource exhaustion, not brief spikes
- Correlate endpoint findings with [[Path Analysis]] before escalating to network or application teams
- Review trends after OS or agent upgrades

---

# Related Notes

- [[ZDX]]
- [[User Experience Score]]
- [[Path Analysis]]
- [[Device Posture]]
