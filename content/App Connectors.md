# App Connectors

## Overview

**App Connectors** are lightweight components deployed inside private environments providing connectivity between [[ZPA]] and internal applications, without exposing those applications to the internet.

Related:

- [[ZPA]]
- [[Application Segments]]

---

> [!important] ZDTE exam weight
> Blueprint objectives landing here:
> - "Given an example user's location and bandwidth, and **the chart from the Help Doc on Sizing**, identify the appropriate App Connector configuration to ensure correct sizing and high availability"
> - "Given a scenario in which a ZPA App Connector needs to be deployed via a virtual machine, identify the appropriate deployment process"
>
> Note the blueprint says the sizing chart is **provided in the question**. You are not expected to memorize the table — you are expected to *read it correctly and apply HA logic*. Practice the reasoning, not the numbers.

---

# The Outbound-Only Model

```text
App Connector
      │
      ▼  outbound TLS only
ZPA Service Edge
```

**Why this matters on the exam:** App Connectors initiate *outbound* connections. No inbound firewall rules, no DMZ, no public IP, no exposed listening ports. If a scenario asks what inbound ports to open, the answer is **none** — and any option requiring inbound rules is wrong.

Firewall requirement is therefore: permit **outbound** connectivity from the connector to Zscaler infrastructure.

---

# Sizing and High Availability

## The Reasoning Pattern

When given a sizing chart plus a user/bandwidth scenario:

1. Determine required aggregate throughput (users × per-user bandwidth)
2. Read the chart for the connector spec that meets it
3. **Add capacity for HA** — this is the step candidates miss

## The N+1 Principle

```text
Never size to exactly the required capacity.
```

If one connector meets the requirement, deploy **two**. If failover must preserve full performance, each connector must independently carry the full load — not half of it.

**Exam trap:** an option that sizes two connectors at 50% capacity each "because there are two" fails on failover — losing one leaves the site at half capacity. Correct HA sizing means the surviving connector can still carry the load.

Minimum recommended per site or per critical application path:

```text
2 App Connectors
```

Benefits: failover, in-place maintenance/upgrade, resilience.

---

# VM Deployment Process

Blueprint objective on deploying via virtual machine. The general sequence:

```text
1. Create a Provisioning Key in the ZPA console
       │  (tied to an App Connector Group)
       ▼
2. Deploy the connector VM image to the hypervisor/cloud
       │
       ▼
3. Provide the provisioning key during first boot
       │
       ▼
4. Connector registers outbound to ZPA
       │
       ▼
5. Verify connector shows healthy in the console
       │
       ▼
6. Assign the App Connector Group to a Server Group
```

**Key concepts to hold:**

- The **provisioning key** binds the connector to a specific App Connector Group — it determines where the connector lands logically.
- Enrollment is outbound; the connector "calls home," you never point ZPA at the connector.
- A connector is useless until its group is referenced by a Server Group used by an Application Segment.

Supported deployment targets include on-premises hypervisors, AWS, Azure, and Google Cloud.

---

# Connector Groups

Organize connectors logically — commonly by location or environment:

```text
Production - US East
Production - EMEA
Development
DR
```

Connector Groups are bound to **Server Groups**, which are in turn referenced by **Application Segments**. See the hierarchy in [[Application Segments]].

Grouping by geography matters for latency: users should reach apps via a connector near the application, not near the user.

---

# Monitoring

Metrics that appear in operational scenarios:

- Connector health / status
- CPU and memory utilization
- Application reachability from the connector
- Connectivity to ZPA

---

# Troubleshooting

## Application Unreachable

1. **Connector health** — is it online and reporting?
2. **Server Group assignment** — is a Server Group attached to the segment, and does it contain a healthy connector group?
3. **DNS from the connector** — the connector must resolve the app's FQDN; this is a frequent root cause
4. **Network routing** — can the connector actually reach the app's IP and port?
5. Segment definition (see [[Application Segments]])

## Connector Won't Register

1. Outbound connectivity permitted?
2. Provisioning key valid, unexpired, not over its enrollment limit?
3. Time sync / certificate validity
4. DNS resolution for Zscaler endpoints

## Latency to an Application

Check whether traffic is traversing a geographically distant connector — a connector in the wrong region is a classic cause.

---

# Best Practices

- Minimum two connectors per critical path, each sized for full load
- Distribute across availability zones or physical hosts
- Separate environments into distinct connector groups
- Place connectors near the applications they serve
- Monitor health continuously and alert on degradation
- Document ownership per connector group

---

# Related Notes

- [[ZPA]]
- [[Application Segments]]
- [[Access Policies]]
- [[Traffic Forwarding]]
