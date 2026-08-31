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
>
> The internals content below (Manager vs. runtime architecture, RHEL host boundaries, update mechanics) is deep operational detail unlikely to be tested directly, but it's exactly the kind of knowledge that separates "passed the exam" from "can actually run this in production" — and it clarifies several things the exam's deployment objective touches only at the surface.

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

# Internals: The Two-Layer Software Architecture

> [!warning] Source note
> Everything from here through "Optional Newer Management Capabilities" comes from a technical Q&A document with **no visible source attribution** — no Zscaler Help Portal branding, no named community contributor, nothing identifying its origin. Unlike the official Help Portal PDFs or the attributed community.zscaler.com guides elsewhere in this vault, I can't independently verify this against a citable source. It reads as detailed, internally consistent, and technically plausible — but treat it as unverified operational detail rather than confirmed vendor documentation until you've checked it against current official Zscaler docs, especially for anything version-specific.

A customer-hosted App Connector (commonly on RHEL) actually runs as **two distinct software components**, not one:

|Console field|Software component|Purpose|
|---|---|---|
|**Manager Update**|`zpa-connector`|Supervises the connector, downloads and launches the runtime, manages software upgrades|
|**Software Update**|`zpa-connector-child`|Establishes ZPA microtunnels, communicates with internal applications, performs health checks and application learning|

**Simple mental model:** the Manager is the updater/supervisor; the child process is the actual data-plane connector.

```text
Hypervisor / cloud infrastructure
        │
        ▼
RHEL operating system
        │
        ▼
RHEL systemd
        │
        ▼
zpa-connector.service
        │
        ▼
/opt/zscaler/bin/zpa-connector   (the Manager)
        │
        ▼
zpa-connector-child               (the data-plane runtime)
```

`zpa-connector` is a host-level **application service** — not a hypervisor or virtualization component. The RPM installs a standard systemd unit (`zpa-connector.service`) managed by RHEL's normal systemd, launching `/opt/zscaler/bin/zpa-connector`. This is architecturally the same pattern as installing any other third-party daemon on RHEL — an EDR agent, a monitoring agent, an application server.

**What the Manager (`zpa-connector`) actually does:**
- Enrolls and identifies the connector
- Communicates with ZPA control infrastructure
- Downloads the appropriate `zpa-connector-child` software
- Starts and supervises the child process
- Coordinates connector software upgrades

**What the child (`zpa-connector-child`) actually does:** the bulk of the real data-plane work — establishing microtunnels, communicating with internal applications, health checks, application learning.

> [!important] Exam-adjacent clarification
> A **Manager Update failure does not necessarily mean the App Connector is offline.** Connector Status, Software Update, and Manager Update are three separately-trackable states — a connector can be fully operational (Connector Status: healthy) while its Manager is behind on updates. However, a failed/outdated Manager should still be addressed, because the Manager is what enables *future* connector software upgrades at all.

---

# Host Management Boundary — Who Owns What

For a customer-hosted RHEL App Connector, the responsibility split is narrower than it might seem — Zscaler's directly-managed footprint is genuinely small.

## Customer-Managed (Assume you own all of this)

- The VM, cloud instance, or physical host
- Hypervisor and cloud infrastructure
- RHEL subscription and repositories
- Kernel and kernel modules
- systemd itself
- OpenSSL and the OS CA trust store
- glibc, NetworkManager, DNS, NTP, routing
- Local firewall and SELinux configuration
- SSH access
- EDR, vulnerability management, and monitoring agents
- Disk, CPU, memory, capacity
- OS patching and reboots
- Backups, snapshots, disaster recovery (of the host itself)

## Zscaler-Managed

- The `zpa-connector` Manager process and its update mechanism
- The `zpa-connector-child` runtime and its update schedule
- Runtime image/version artifacts under `/opt/zscaler/var/`
- Cloud-delivered connector configuration (group membership, version profile, reachability info, Service Edge selection)

## Shared Responsibility

- **Connector enrollment identity** — the connector generates its private key locally and creates a certificate-signing request; ZPA coordinates certificate issuance and enrollment. The private key stays local. Your organization still controls the provisioning process and can choose a Zscaler-issued enrollment CA or your own.

```text
Customer-managed infrastructure
    Hypervisor / cloud VM
    RHEL operating system
    Standard RHEL packages and services
             │
             ▼
Zscaler application layer
    zpa-connector.service
        └── zpa-connector (Manager)
                └── zpa-connector-child (runtime)
                        └── ZPA microtunnels
```

**Bottom line:** outside of optional newer prebuilt-image features (below), Zscaler directly controls the Manager, the child runtime, its runtime artifacts, and cloud-delivered configuration/enrollment. Zscaler does **not** ordinarily administer the underlying RHEL host or hypervisor.

---

# Update Mechanics and Terminology

Three genuinely distinct update types get tracked separately, and conflating them is the most likely source of confusion in this area:

|Update type|What it updates|Initiated by|
|---|---|---|
|**Manager Update**|`zpa-connector` RPM/service|Zscaler (during maintenance window, if enabled) **or** your RHEL patching process|
|**Software Update**|`zpa-connector-child` runtime|Zscaler, per the App Connector group's software-update schedule|
|**OS Update**|RHEL packages, libraries, kernel|Your organization's RHEL patching process (or Zscaler, only via the limited-availability automated RHEL OS update feature — see below)|

## Automated Manager Updates

Available during the App Connector group's configured maintenance window. **Requires both the Manager and App Connector software to be a specific minimum version or later** (documented at 25.46.3+ in the source material — verify the current minimum against official docs, since version floors like this shift over releases). Hovering over the Manager Update status shows the target version and upgrade window.

## Manager Updates Are Not Locked to Zscaler's Automation

This is the single most important operational nuance here: **disabling Zscaler's automated Manager-update mechanism does not version-lock the RPM.** Your RHEL patching process can still update `zpa-connector` independently:

```bash
sudo dnf upgrade zpa-connector
# or
sudo yum upgrade zpa-connector
```

A broader `sudo dnf update` / `sudo yum update -y` will **also** update the Manager RPM if the Zscaler repository is enabled and a newer package is available and not excluded/version-locked. For strict change control, verify explicitly:

```bash
sudo dnf check-update zpa-connector
rpm -q zpa-connector
```

|Update path|Requires ZPA automatic Manager update enabled?|Initiated by|
|---|---|---|
|ZPA Manager Update during maintenance window|Yes|Zscaler/ZPA control plane|
|`dnf upgrade zpa-connector`|No|Your RHEL administrator/automation|
|General `dnf update` / `yum update`|No|Your OS patching process|
|Rebuilding the VM from a newer connector image|No|Your infrastructure team|

## Useful Inspection Commands

```bash
rpm -q zpa-connector                 # installed version
rpm -qi zpa-connector                # package detail
rpm -ql zpa-connector                # everything the RPM installed
systemctl status zpa-connector       # service state
systemctl cat zpa-connector          # the systemd unit itself
ps -ef | grep '[z]pa-connector'      # parent and child processes running
```

---

# Reboot Behavior — Service Restart vs. Full Host Reboot

A distinction worth being precise about, since the operational impact differs a lot:

|Zscaler action|What restarts|Full RHEL reboot?|Expected impact|
|---|---|---|---|
|App Connector software update|`zpa-connector-child` (and potentially the service stack)|**No**|Connector temporarily unavailable|
|Manager update|`zpa-connector.service` and its child|**No**|Brief connector interruption|
|Disaster Recovery activation/test|`zpa-connector.service`|**No**|Existing sessions through that connector drop|
|Routine app/server-group/policy changes|Configuration refreshed|**No**|Usually no outage|
|Automated RHEL OS update (limited availability feature)|RHEL host/VM|**Yes**|Full boot cycle|
|Recovery after a failed connector update|VM reboot may be *recommended*|Possibly — administrator-driven|Troubleshooting action, not normal behavior|

A pure Manager-package update pairs with a **service restart**, not a host reboot:

```bash
sudo yum update zpa-connector
sudo systemctl restart zpa-connector
```

Compare to the documented RHEL host-update procedure, which explicitly ends in `sudo reboot` — a materially different operation triggered only by OS-level updates, not Manager or Software updates.

## In a Properly Redundant Connector Group

```text
One connector restarts (Manager or Software update)
        │
        ▼
Other connectors continue accepting new connections
        │
        ▼
Updated connector reconnects, returns to service
        │
        ▼
Applications remain available — provided remaining
connectors have sufficient capacity
```

This is the operational payoff of the N+1 sizing principle above — it's not just an HA nicety, it's what makes rolling Manager/Software updates non-disruptive in practice.

## Proving Which One Actually Happened

```bash
uptime -s                                    # when RHEL last booted
cat /proc/sys/kernel/random/boot_id          # unique ID for current boot
journalctl --list-boots                      # current + previous boots
journalctl -u zpa-connector --since "24 hours ago"   # service restart history
systemctl show zpa-connector -p ActiveEnterTimestamp -p ExecMainStartTimestamp
```

**Interpretation:** new `zpa-connector` PID + unchanged boot ID = service/process restart only. Changed boot ID + new journal boot entry = full RHEL reboot actually occurred.

---

# Runtime Artifacts

The Manager maintains cached runtime state under `/opt/zscaler/var/`, including files such as `image.bin`, `version`, and `metadata`. These are **not** additional RHEL services — they're the Manager's cached runtime payload and version bookkeeping.

> [!warning] Don't let generic tooling touch these
> These files should be treated as **Zscaler-owned application state**, not something your configuration-management or vulnerability-remediation tooling should modify or delete. Deleting them causes the Manager to contact the ZPA cloud and re-download the default App Connector software from scratch — disruptive if triggered accidentally by a well-meaning but unaware automated process.

---

# Disaster Recovery

Activating Disaster Recovery or DR Test Mode restarts `zpa-connector` services and can **drop existing user connections through that connector** — but the underlying host itself remains running throughout. Same category of event as a Manager/Software update in terms of host impact: service-level disruption, not a host reboot.

---

# Optional Newer Management Capabilities

Two newer, more expansive management features exist for certain deployment models — worth knowing they exist, but don't assume they apply universally:

## Automated RHEL OS Updates
Available for certain **Zscaler-provided prebuilt images** only — **not supported** for App Connectors installed manually from an RPM onto a custom RHEL build. Introduced as a limited-availability feature, so actual availability may depend on tenant and licensing. Even where available, Zscaler's general guidance still treats the host OS as the organization's responsibility.

## Remote OS Network Configuration
Recent App Connector releases added centralized control of certain local network settings (routes, DNS, SSH service state, and related items) from the ZPA control plane, when the feature is enabled. This is still initiated under your tenant's own administrative control — not arbitrary vendor administration of your host.

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
- **Manager Update** and **Software Update** status — tracked and can fail independently of overall Connector Status (see Internals above)

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

## Manager Update Shows Failed

Don't assume the connector itself is down — check Connector Status and Software Update separately first, since all three can fail independently. Common causes for a Manager Update failure specifically: low disk space, package/dependency conflicts, YUM/DNF locks, repository connectivity issues, or certificate trust problems on the host. Address it anyway even if the connector is otherwise healthy — a stuck Manager blocks all *future* upgrades.

## Determining Whether an Update Caused a Service Restart or a Full Reboot

Use the boot-ID/journalctl method under "Reboot Behavior" above rather than assuming — this matters for accurately explaining an outage window in a postmortem or change record.

## Runtime Behaving Oddly After Manual File Changes Under `/opt/zscaler/var/`

If `image.bin`, `version`, or `metadata` were touched by external tooling, expect the Manager to re-download the default App Connector software on next check-in — this presents as an unexpected, unscheduled software refresh.

---

# Best Practices

- Minimum two connectors per critical path, each sized for full load
- Distribute across availability zones or physical hosts
- Separate environments into distinct connector groups
- Place connectors near the applications they serve
- Monitor health continuously and alert on degradation
- Document ownership per connector group
- **Exclude `/opt/zscaler/var/` from generic configuration-management, backup-restore, or vulnerability-remediation automation** that isn't specifically aware of what it's touching
- If disabling Zscaler's automated Manager updates for change-control reasons, explicitly verify your RHEL patching process isn't updating `zpa-connector` anyway via a broad `dnf update` — disabling one path doesn't disable the other

---

# Related Notes

- [[ZPA]]
- [[Application Segments]]
- [[Access Policies]]
- [[Traffic Forwarding]]
- [[Troubleshooting Methodology]]
- [[Common Gotchas]]
