# DNS Security

## Overview

**DNS Security** protects users by analyzing domain resolution requests and preventing connections to malicious infrastructure.

Related:

- [[ZIA]]
- [[Traffic Forwarding]]

---

> [!important] ZDTE exam weight
> Blueprint objective: *"Given a scenario in which an organization requires blocking of specific types of sites, identify how to utilize **DNS Security Policies** to accomplish this."*
>
> The key exam skill is knowing **when DNS Control is the right tool versus URL Filtering** — they overlap, and choosing the wrong one is the trap.

---

# DNS Control vs URL Filtering — The Decision

| |**DNS Control**|**URL Filtering**|
|---|---|---|
|Operates at|Domain resolution|Full URL, after connection|
|Sees|Domain name only|Full path, query string|
|Works for|**Any protocol**, including non-web|Web traffic|
|Requires SSL inspection|No|For full URL visibility on HTTPS, yes|
|Blocks|Before connection is made|At/after request|

## When DNS Control Is the Answer

- Blocking must apply to **non-web protocols** as well as browsing
- You want to stop the connection **before it's established**
- Blocking command-and-control or DNS tunneling
- SSL inspection is not in place for that traffic
- Blocking at the domain level is sufficient (no need for path granularity)

## When URL Filtering Is the Answer

- The requirement is **path-specific** (allow `site.com/public`, block `site.com/admin`)
- Category-based browsing control with user override/caution actions
- You need actions DNS can't express — caution, continue, isolate

**Exam trap:** if a scenario needs to block only part of a site, DNS Control cannot do it — it only knows the domain. If a scenario needs to block a protocol the browser isn't using, URL Filtering alone won't reach it.

Many correct answers use **both** in combination.

---

# DNS Security Architecture

```text
User Device
      │
      ▼
DNS Request
      │
      ▼
Zscaler Cloud
      │
      ▼
Threat Analysis / Policy Match
      │
      ▼
Allow / Block / Redirect / Log
```

---

# Threat Categories

## Malware Domains
Known malicious infrastructure.

## Phishing Domains
Credential theft attempts.

## Command and Control
Attacker communication channels — a primary DNS Control use case, since C2 often does not use a browser.

## DNS Tunneling
Data exfiltration encoded in DNS queries. Detected by query pattern analysis, not domain reputation.

## Newly Registered Domains
High-risk by age; frequently used in phishing campaigns before reputation exists.

---

# DNS Policy Actions

|Action|Result|
|---|---|
|Allow|Resolve normally|
|Block|Prevent resolution|
|Redirect|Return a controlled response|
|Log / Monitor|Record without blocking|

**Response code control:** DNS Control can specify the response code returned on a block, which affects how the client application behaves — a silent failure versus an explicit refusal produces different user-visible symptoms.

---

# Rule Criteria

DNS rules can match on:

- User, group, department
- Location / sublocation
- Destination domain or DNS category
- Request type (record type)
- Protocol

---

# Building a Blocking Policy

For "block specific types of sites" scenarios:

```text
1. Identify the category or domains to block
2. Determine scope — which users/groups/locations
3. Create the DNS rule with that criteria
4. Set the action (block) and response behavior
5. Order the rule correctly relative to existing rules
6. Create exceptions ABOVE the block rule if needed
7. Verify with DNS logs
```

**Rule order matters** — first match wins. An exception placed below a broad block never executes. This is the same failure pattern as [[PAC Files]] logic and ZIA policy ordering.

---

# Troubleshooting

## DNS Failure

1. Endpoint DNS settings — is the client actually using Zscaler resolution?
2. Traffic forwarding — is DNS reaching the cloud? (See [[Traffic Forwarding]])
3. DNS policy — which rule matched?
4. DNS logs — confirm the decision and the matching rule

## Site Blocked Unexpectedly

Check DNS logs first to determine whether the block was DNS-layer or URL-layer — the remediation differs entirely. If DNS blocked it, no amount of URL policy adjustment will fix it.

## Application Fails but Browsing Works

Suspect a DNS block affecting a non-web protocol the URL policy never touches.

---

# Best Practices

- Enable DNS inspection broadly
- Use DNS Control for protocol-agnostic blocking
- Pair with URL Filtering for granular web control
- Monitor blocked domains for false positives
- Review exceptions regularly

---

# Related Notes

- [[ZIA]]
- [[ZIA Policy Design]]
- [[Traffic Forwarding]]
- [[PAC Files]]
