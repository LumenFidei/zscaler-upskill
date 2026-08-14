# Logging and Reporting

## Overview

How Zscaler services record activity and stream it to external systems for visibility, troubleshooting, and compliance.

Related:

- [[ZIA]]
- [[ZPA]]
- [[ZDX]]

---

> [!important] ZDTE exam weight
> Blueprint objectives landing here:
> - "Given a scenario about **Nanolog Streaming Service (NSS)** Feed setup for SIEM for a given type of data, identify the Zscaler log fields to bring over to the SIEM"
> - "Given a scenario about **Log Streaming Service (LSS)**, for a given type of data, identify the Zscaler log fields to bring over to the SIEM"
> - "Given a **Zscaler Audit Log**, interpret the information"
>
> The NSS vs LSS distinction is a near-certain exam item. Know which belongs to which product.

---

# NSS vs LSS — Know the Difference

| | **NSS** | **LSS** |
|---|---|---|
|Full name|Nanolog Streaming Service|Log Streaming Service|
|Product|**ZIA**|**ZPA**|
|Streams|Web, firewall, DNS, DLP, tunnel logs|App access, connector, audit, user activity logs|
|Deployed as|NSS VM in your environment|LSS receiver via **App Connector**|

**One-line memory hook:** NSS = internet side (ZIA). LSS = private side (ZPA).

An option that proposes NSS for ZPA app-access logs, or LSS for ZIA web logs, is wrong.

---

# NSS (ZIA)

## Architecture

```text
ZIA Nanolog
      │
      ▼
NSS VM (customer-hosted)
      │
      ▼
SIEM
```

## Feed Types

An NSS feed is configured per **log type** — select the type first, then the fields:

- Web logs
- Firewall logs
- DNS logs
- DLP incidents
- Tunnel logs
- Alerts

## Configuration Elements

- **Feed name and status**
- **SIEM destination** — IP/FQDN and port
- **Log type**
- **Output format** — field selection and delimiter
- **Filters** — restrict by user, location, category, action to control volume

**Exam reasoning:** questions give a use case and ask which fields to send. Match fields to the *investigative question* — don't send everything; volume and cost matter.

## Field Selection by Use Case

|Use case|Fields that matter|
|---|---|
|DLP investigation|user, timestamp, DLP engine/dictionary, severity, action, destination, file name|
|Web access forensics|user, timestamp, URL, URL category, action, request method, response code, bytes|
|Firewall / non-web|user, source IP, destination IP, destination port, protocol, network application, action|
|Threat detection|user, threat name/category, action, destination, URL|
|Bandwidth analysis|user, location, bytes sent/received, application, timestamp|

---

# LSS (ZPA)

## Architecture

```text
ZPA
  │
  ▼
LSS Receiver (runs on an App Connector)
  │
  ▼
SIEM
```

**Exam-relevant:** LSS runs on App Connector infrastructure — it does not need a separate appliance the way NSS does.

## Log Types

- **User Activity** — application access events and decisions
- **User Status** — authentication and session state
- **App Connector Status** — connector health events
- **Private Service Edge Status**
- **Audit Logs** — administrative changes
- **Browser Access**

## Field Selection by Use Case

|Use case|Fields that matter|
|---|---|
|Access denial investigation|username, application, policy name, action/decision, timestamp, posture result|
|Connector health|connector name, status, timestamp, group|
|Session forensics|username, app segment, session duration, bytes, client IP|
|Compliance audit|admin user, changed object, before/after values, timestamp|

---

# Audit Logs

Blueprint objective: *"Given a Zscaler Audit Log, interpret the information."*

Audit logs record **administrative actions**, not user traffic. When reading one, extract:

- **Who** — admin account performing the action
- **When** — timestamp
- **What object** — policy, rule, segment, user, connector
- **Action type** — create, modify, delete, activate
- **Before / after values** — what actually changed
- **Source** — IP or interface of the change

**Common exam framing:** a policy stopped working; the audit log shows what changed, who changed it, and when. The correct answer traces the failure to a specific modification.

Practical read: if a rule's behavior changed at time T, look for an audit entry near T touching that rule or an object it references — including **rule order** changes, which break policy without changing any rule's content.

---

# Log Categories by Product

## ZIA
Web, firewall, DNS, DLP, sandbox, tunnel

## ZPA
User activity, user status, connector status, audit, browser access

## ZCC
Tunnel, authentication, policy, connectivity, device posture

## ZDX
Device health telemetry, network path metrics, application performance

---

# Cross-Product Correlation

Real incidents span products. A user reporting "I can't reach the app":

```text
ZCC logs          → is the tunnel up, is the user authenticated?
ZIA web logs      → was it blocked by URL/firewall policy?
ZPA user activity → was it denied by access policy or posture?
ZDX               → is it a performance problem, not a block?
```

Knowing *which log answers which question* is the operational skill the exam tests.

---

# Retention and Volume

- Retention varies by service and licence tier
- Streaming to SIEM is the mechanism for retention beyond console limits
- Use NSS/LSS **filters** to control volume

---

# Best Practices

- Stream to SIEM for retention and correlation
- Filter at the feed, not just in the SIEM
- Select fields by investigative purpose
- Review audit logs as part of change management
- Correlate across products rather than reading one in isolation

---

# Related Notes

- [[ZIA]]
- [[ZPA]]
- [[ZCC]]
- [[ZDX]]
- [[Troubleshooting Methodology]]
