# Threat Hunting

## Overview

**Threat Hunting** is the proactive process of searching an environment for signs of attacker activity that have not yet triggered automated alerts.

Threat hunting complements detection technologies such as [[Deception]] by investigating hypotheses rather than waiting passively for alerts.

Related:

- [[Deception]]
- [[Lateral Movement]]
- [[Credential Theft]]
- [[SOC Workflow]]

---

# Purpose

- Identify undetected intrusions
- Reduce dwell time
- Validate detection coverage
- Improve detection content based on findings

---

# Threat Hunting Process

```text
Hypothesis
    │
    ▼
Data Collection
    │
    ▼
Analysis
    │
    ▼
Findings
    │
    ▼
Response / Detection Improvement
```

---

# Common Data Sources

- Endpoint telemetry
- Authentication logs
- Network traffic logs
- Deception alerts
- DNS logs

Related: [[Deception]], [[DNS Security]]

---

# Hunting Techniques

## Hypothesis-Driven Hunting

Starts from a specific attacker behavior assumption (e.g. "an attacker may be using stolen credentials to access finance systems").

## Indicator-Based Hunting

Searches for known indicators of compromise (IOCs) across logs.

## Anomaly-Based Hunting

Looks for statistical deviations from normal behavior baselines.

---

# Using Deception in Threat Hunting

Because legitimate users should never interact with decoys or lures, any [[Deception]] alert provides a high-confidence starting point for a hunt:

```text
Decoy Interaction
        │
        ▼
Pivot to Endpoint Telemetry
        │
        ▼
Trace Origin and Scope
```

---

# Common Hunt Targets

- [[Lateral Movement]]
- [[Credential Theft]]
- Privilege escalation
- Data staging and exfiltration

---

# Best Practices

- Document hunt hypotheses and outcomes
- Feed findings back into detection engineering
- Prioritize hunts around privileged systems
- Coordinate with [[SOC Workflow]] for escalation

---

# Related Notes

- [[Deception]]
- [[Lateral Movement]]
- [[Credential Theft]]
- [[SOC Workflow]]
