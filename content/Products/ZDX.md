# Zscaler Digital Experience (ZDX)

## Overview

**Zscaler Digital Experience (ZDX)** is Zscaler's Digital Experience Monitoring (DEM) platform that provides end-to-end visibility into user experience across devices, networks, internet services, SaaS applications, and private applications.

ZDX helps IT and operations teams identify whether performance issues originate from:

- The endpoint
    
- Local network
    
- ISP
    
- Internet path
    
- Zscaler service edge
    
- SaaS provider
    
- Private application
    

---

# Objectives

- Improve user experience
    
- Reduce Mean Time To Resolution (MTTR)
    
- Identify root causes quickly
    
- Monitor endpoint health
    
- Monitor SaaS performance
    
- Monitor private application performance
    

---

# Architecture

```text
User Device
     │
     ▼
ZCC + ZDX Telemetry
     │
     ▼
ZDX Cloud
     │
 ┌───┼───────────────┐
 │   │               │
 ▼   ▼               ▼
Device Network     Application
Metrics Metrics     Metrics
```

---

# Major Components

## Endpoint Monitoring

Collects:

- CPU usage
    
- Memory utilization
    
- Disk utilization
    
- Process activity
    
- System health
    

---

## Network Monitoring

Measures:

- Latency
    
- Jitter
    
- Packet loss
    
- DNS performance
    
- Gateway response
    

---

## Application Monitoring

Monitors:

- SaaS applications
    
- Internal applications
    
- Web applications
    

---

## Path Analysis

Visualizes:

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
Zscaler Edge
  │
  ▼
Application
```

---

# Device Metrics

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
    

---

# Application Monitoring

## SaaS Platforms

Examples:

- Microsoft 365
    
- Salesforce
    
- ServiceNow
    
- Google Workspace
    
- Zoom
    

## Metrics

- Response time
    
- Availability
    
- Transaction latency
    

---

# User Experience Score

## Purpose

Provide a summarized experience rating.

Factors include:

- Device health
    
- Network quality
    
- Application performance
    

---

# Root Cause Analysis

## Workflow

```text
User Complaint
      │
      ▼
Experience Dashboard
      │
      ▼
Device?
Network?
Application?
      │
      ▼
Root Cause
```

---

# Alerts

## Common Alert Types

- High latency
    
- Excessive packet loss
    
- Poor Wi-Fi
    
- Application outage
    
- ISP degradation
    

---

# Dashboards

## Executive View

- Overall experience score
    
- User trends
    
- Application health
    

## Operations View

- Device health
    
- Network performance
    
- Active incidents
    

---

# Troubleshooting

## Slow Microsoft 365

Check:

- Endpoint health
    
- ISP latency
    
- DNS performance
    
- Microsoft service health
    

## Poor Zoom Quality

Check:

- Jitter
    
- Packet loss
    
- Wi-Fi signal strength
    

---

# Best Practices

- Monitor critical SaaS apps
    
- Create alert thresholds
    
- Baseline user experience
    
- Review recurring issues
    
- Correlate ZDX with ZIA and ZPA logs
    

---

# Related Notes

- [[ZCC]]
    
- [[ZIA]]
    
- [[ZPA]]
    
- [[User Experience Score]]
    
- [[Path Analysis]]
    
- [[Endpoint Monitoring]]