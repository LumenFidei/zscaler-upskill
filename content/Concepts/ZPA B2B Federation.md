# ZPA B2B Federation

## Overview

**ZPA B2B Federation** allows organizations to share private application access with external "guest" users — partners, subsidiaries, or organizations involved in mergers, acquisitions, and divestitures — without exposing those applications to the public internet. It provides zero trust application access between organizations by federating two separate ZPA tenants.

> [!important] ZDTE tie-in
> ZPA B2B Federation is an Architecture and Design / Implementation and Deployment topic that also touches [[Migration from VPN to ZPA]] scenarios — it directly replaces the site-to-site VPN and IPsec tunnel model traditionally used for partner/M&A connectivity.

Related:

- [[ZPA]]
- [[Application Segments]]
- [[Zero Trust]]
- [[Migration from VPN to ZPA]]

---

# Why It Exists

Traditional partner/M&A connectivity relies on:

- Site-to-site VPNs
- Coordinated firewall and NAT rules
- Sometimes shipping hardware to a partner's data center

```text
Traditional Partner Connectivity

Org A Network
      │
      ▼
Site-to-Site VPN
      │
      ▼
Org B Network
```

This exposes flat network access and requires slow, manual coordination.

---

# ZPA B2B Federation Model

```text
ZPA Customer A                          ZPA Customer B
     │                                        │
Users / Private Apps                  Users / Private Apps
     │                                        │
     ▼                                        ▼
   ZCC ────────► Zero Trust Exchange ◄──────── ZCC
                (Access & Policy via
                 ZPA Tenant Federation)
```

Both organizations remain on their own ZPA tenants. The Zero Trust Exchange brokers access and policy between the two tenants — neither organization's applications are exposed to the other's network, and no site-to-site VPN is required.

---

# Benefits

|Benefit|Description|
|---|---|
|Eliminated attack surface|Internal apps stay invisible to the public internet and to the partner's network — no listening ports, no discoverable IPs|
|Simplified onboarding|No firewall/NAT coordination or hardware shipping; onboarding happens at business speed|
|Secure bi-directional connectivity|Zero Trust Exchange brokers access both ways for workload-to-workload communication|
|Reduced operational cost|Removes site-to-site VPN and IPsec tunnel management overhead|

---

# ZPA B2B Solution Comparison

|Feature|Primary Use Case|Ideal Partner Profile|Endpoint Requirement|
|---|---|---|---|
|ZPA B2B Federation|Seamless user-to-app access across organizations|Both sides are ZPA customers|ZCC connected to primary tenant|
|Partner Login (ZCC Multi-Tenant)|User-to-app access across organizations, simple/temporary access|Both sides ZPA|ZCC with a secondary "Partner Tenant" profile added|
|ZPA B2B Extranet|Server-to-server, branch, or bulk network connectivity (replacing legacy site-to-site IPsec VPNs)|Partner does not use ZPA|No endpoint agent — requires an IPsec tunnel from the partner's gateway to Zscaler|

> [!important] ZDTE tie-in
> This comparison table is a strong exam target — know which model fits which partner profile. B2B Federation and Partner Login both assume the partner is a ZPA customer; B2B Extranet is the option when the partner is not.

---

# Roles

## Host

The organization that owns the application being shared.

## Guest

The partner organization whose users require access to the host's application.

---

# Deployment Steps

## Feature Enablement

B2B Federation currently requires a feature flag enablement — raise a support ticket for **both** the Guest and Host partner tenants separately before configuring.

## Step 1: Establish Federation via Secure Token Exchange

```text
Navigate to: Infrastructure → B2B Exchange → Partners
        │
        ▼
Add Partner → Generate Access Token
        │
        ▼
Partner Verifies Access Token
        │
        ▼
Federation Status: Active
```

Token validity is configurable (in hours), and once generated, federation status can be managed as **Active**, **Paused**, or **Terminated**.

---

## Step 2: Publish Application Segments With the Partner Tenant

The application must be published on the **Host** account. Once the Application Segment exists, the Federated Partner is assigned via the segment's Federation icon.

Key Application Segment attributes relevant to B2B:

|Attribute|Purpose|
|---|---|
|Segment Group|Logical grouping of the shared application(s)|
|Server Group(s)|Backend server group serving the application|
|Federated By / Partners|Identifies which partner tenant this segment is federated with|
|Double Encryption|Optional additional encryption layer|
|Source IP Anchor|Related to [[SIPA]] — can be enabled if the partner needs a fixed source IP|

---

## Step 3: Configure Access Policies on the Guest Side

The **Guest** tenant configures the access policy that governs which of its users can reach the federated application, enforcing standard zero trust access controls (identity, device posture, context) on top of the federation relationship.

---

# Troubleshooting

## Partner Cannot See Shared Application

Check:

1. Federation status is **Active**, not Paused or Terminated
2. Application Segment was actually assigned to the Federated Partner (Federation icon)
3. Guest-side access policy actually includes the federated application

---

## Access Token Fails to Verify

Check:

1. Token validity window (tokens expire based on the configured hours)
2. Whether the correct tenant generated vs. verified the token (Host generates, Guest verifies, or vice versa depending on direction)

---

## Feature Not Available

Check:

1. Whether the B2B Feature Enablement ticket was raised for **both** Host and Guest tenants
2. ZPA edition/licensing supports B2B Federation

---

# Best Practices

- Scope shared Application Segments narrowly — don't federate broad segment groups when only specific apps are needed
- Use Pause rather than Terminate for temporary suspensions (e.g., during an M&A integration pause) to avoid re-establishing federation from scratch
- Choose the right B2B model up front: Federation/Partner Login for ZPA-to-ZPA partners, B2B Extranet when the partner has no ZPA presence
- Treat B2B Federation as a Zero Trust extension, not a blanket trust relationship — access policy still governs what Guest users can actually reach

---

# Related Notes

- [[ZPA]]
- [[Application Segments]]
- [[Zero Trust]]
- [[Migration from VPN to ZPA]]
- [[SIPA]]
- [[ZPA Policy Design]]
