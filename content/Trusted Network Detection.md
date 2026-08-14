# Trusted Network Detection

## Overview

**Trusted Network Detection (TND)** is a ZCC capability used to determine whether a device is currently connected to a trusted corporate network, such as an office LAN.

TND results influence forwarding behavior, allowing traffic to bypass Zscaler cloud forwarding when the device is already on a trusted, secured network.

Related:

- [[ZCC]]
- [[Traffic Forwarding]]
- [[App Profiles]]

---

# Purpose

Not all networks need traffic forwarded to the Zscaler cloud. Corporate networks that already have local security controls may be treated differently than untrusted or public networks.

```text
Device Network
      │
      ▼
Trusted?
      │
 ┌────┴────┐
 │         │
Yes        No
 │         │
Bypass    Forward to Zscaler Cloud
```

---

# Detection Methods

## Gateway Detection

Identifies trusted networks by matching the default gateway IP address.

## DNS Detection

Confirms trust by resolving a known internal DNS record that only resolves on the corporate network.

## HTTP Detection

Validates trust by reaching an internal web resource reachable only from inside the network.

## Certificate Detection

Validates trust using a certificate presented by an internal resource.

---

# Evaluation Flow

```text
ZCC Startup / Network Change
           │
           ▼
Run TND Checks
           │
           ▼
Match Criteria?
           │
     ┌─────┴─────┐
     │           │
   Trusted     Untrusted
     │           │
  Bypass      Forward
```

---

# Impact on Forwarding

When a network is detected as trusted:

- ZIA forwarding may be bypassed or reduced
- ZPA behavior may remain unaffected (application access still enforced)
- Split tunnel exclusions may apply

---

# Common Use Cases

- Office networks with existing perimeter security
- Data centers with local egress controls
- Reducing unnecessary cloud forwarding for internal-only traffic

---

# Troubleshooting

## Device Not Detected as Trusted

Check:

1. Gateway IP configuration
2. DNS record resolution
3. HTTP resource reachability
4. Certificate validity
5. App Profile TND assignment

## Device Incorrectly Detected as Trusted

Check:

1. Overlapping gateway IP ranges
2. DNS record leakage (e.g. via VPN split DNS)
3. Stale TND configuration

---

# Best Practices

- Use multiple detection methods together for accuracy
- Keep TND criteria current as network infrastructure changes
- Test TND behavior after office network changes
- Document all trusted network definitions

---

# Related Notes

- [[ZCC]]
- [[Traffic Forwarding]]
- [[App Profiles]]
- [[Tunnel 2.0]]
