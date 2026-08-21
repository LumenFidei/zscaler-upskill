# SSL Inspection Certificate Chain (Root & Intermediate CA)

## Overview

[[SSL Inspection]] relies on a controlled man-in-the-middle (MITM) process to decrypt and inspect encrypted traffic. To do this without breaking client trust, Zscaler uses a two-tier certificate hierarchy: a **Root Certificate Authority (CA)** and an **Intermediate CA**.

Understanding the distinct role of each is essential for diagnosing certificate trust issues.

> [!important] ZDTE tie-in
> Certificate chain understanding is a recurring Security and Compliance / Troubleshooting topic. The key exam distinction: the Root CA establishes trust on the client but never directly signs website certificates — the Intermediate CA does that dynamically, at inspection time.

Related:

- [[SSL Inspection]]
- [[ZIA]]
- [[ZCC]]

---

# Root CA vs. Intermediate CA

|Aspect|Root CA|Intermediate CA|
|---|---|---|
|Role|Trusted anchor|Dynamically signs website certs during inspection|
|Installed on client?|Yes — trusted certificate store|No — not installed directly|
|Validity period|Very long (often decades)|Shorter, rotated more frequently|
|Signs website certs directly?|No|Yes|
|Rotation frequency|Rare|More common, for security/lifecycle reasons|

---

# Why Two Tiers

```text
Root CA
   │
   ▼ (signs)
Intermediate CA
   │
   ▼ (dynamically signs, per site)
Website Certificate (e.g. cnn.com)
```

Because the client trusts the Root CA, and the Root CA has signed the Intermediate CA, the client transitively trusts certificates issued by the Intermediate CA — including the on-the-fly website certificates the proxy generates during inspection.

---

# What Happens During an Inspected Connection

```text
Client Connects to Secure Site
          │
          ▼
Proxy Establishes TLS Session with Real Destination
          │
          ▼
Proxy Generates a New Certificate for the Requested Domain
          │
          ▼
Proxy Signs the New Certificate with the Intermediate CA
          │
          ▼
Client Receives Proxy-Generated Certificate
          │
          ▼
Client Validates Chain: Website Cert → Intermediate CA → Root CA
```

If the Root CA is trusted on the client, this validation succeeds silently — no browser warning.

---

# Deployment

The Zscaler Root CA certificate can be installed through the **App Profile** in the ZCC Admin Portal (Windows/macOS policy), alongside other SSL Inspection–related settings such as:

- Install Zscaler SSL Certificate (toggle)
- Forwarding Profile assignment
- Tunnel Internal Client Connector Traffic
- Cache System Proxy
- ZIA Posture Profile

---

# Verifying the Chain

## On the Client (Certificate Store)

Check the trusted certificate store (e.g. Windows Certificate Manager, under Trusted Root Certification Authorities) for the Zscaler Root CA and confirm its expiration date is far in the future (Root CAs commonly run for decades).

## In the Browser (Live Inspection)

When visiting an inspected site, view the certificate chain in the browser's certificate viewer. You should see:

```text
Zscaler Root CA
   └── Zscaler Intermediate CA (zscalertwo.net or similar)
         └── Website Certificate (e.g. cnn.com)
```

The Root CA's expiration date will differ from — and typically be much later than — the Intermediate CA's expiration date. This is expected and confirms the two-tier design is working correctly.

---

# Troubleshooting

## Certificate Warning / Untrusted Certificate Error

Check:

1. Whether the Zscaler Root CA is actually installed in the client's trusted certificate store
2. Whether the App Profile has "Install Zscaler SSL Certificate" enabled
3. Whether deployment (Intune, Jamf, GPO) actually pushed the certificate to the device

---

## Application Fails Under SSL Inspection

Check:

1. Whether the application uses certificate pinning (a documented [[SSL Inspection]] exclusion case)
2. Whether the app requires an SSL Inspection bypass rule
3. Whether the Intermediate CA has recently rotated and the app cached an old chain

---

## Chain Looks Wrong in Browser

Check:

1. That the Root CA shown matches the one installed on the endpoint
2. That the Intermediate CA is current and not expired
3. Whether SSL Inspection is even active for that category/destination

---

# Best Practices

- Verify Root CA installation as a standard step in any new device onboarding, not just when problems occur
- Document expected certificate-pinned applications separately so failures aren't mistaken for deployment issues
- When rotating the Intermediate CA, communicate the change — cached chains or pinned apps may need updates
- Use the browser's live certificate viewer as a fast troubleshooting step before escalating

---

# Related Notes

- [[SSL Inspection]]
- [[ZIA]]
- [[ZCC]]
- [[Troubleshooting Methodology]]
