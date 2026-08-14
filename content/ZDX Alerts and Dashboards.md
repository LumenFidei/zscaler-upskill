# ZDX Alerts and Dashboards

## Overview

**ZDX Alerts and Dashboards** provide the visibility layer through which administrators monitor digital experience, receive proactive notifications, and investigate degradation.

Related:

- [[ZDX]]
- [[User Experience Score]]
- [[Path Analysis]]

---

# Purpose

Turn raw device, network, and application telemetry into actionable, timely information for IT and operations teams.

---

# Alert Types

## High Latency

Triggered when network latency exceeds defined thresholds.

## Excessive Packet Loss

Triggered when packet loss impacts application usability.

## Poor Wi-Fi

Triggered by degraded wireless signal quality.

## Application Outage

Triggered when a monitored application becomes unreachable or unresponsive.

## ISP Degradation

Triggered when performance issues are isolated to the internet service provider segment.

---

# Alert Workflow

```text
Threshold Breached
        │
        ▼
Alert Generated
        │
        ▼
Notification Sent
        │
        ▼
Administrator Investigates
        │
        ▼
Root Cause via Path Analysis / Endpoint Monitoring
```

---

# Dashboards

## Executive View

Summarized for leadership visibility:

- Overall experience score
- User trends
- Application health

## Operations View

Detailed for day-to-day monitoring:

- Device health
- Network performance
- Active incidents

---

# Alert Tuning Considerations

## Threshold Setting

Thresholds that are too sensitive generate alert fatigue; thresholds that are too loose miss real degradation.

## Scope

Alerts can be scoped by location, department, or application to avoid overly broad notifications.

## Notification Routing

Route alerts to the team best positioned to act (network team for ISP issues, app owner for application outages, etc.).

---

# Best Practices

- Baseline normal performance before setting alert thresholds
- Scope alerts to avoid noise from low-impact fluctuations
- Route alerts to the correct owning team automatically where possible
- Review dashboard trends weekly, not only when alerts fire
- Correlate ZDX alerts with ZIA and ZPA logs for full context

---

# Related Notes

- [[ZDX]]
- [[User Experience Score]]
- [[Path Analysis]]
- [[Endpoint Monitoring]]
