# Browser Isolation

## Overview

**Browser Isolation** renders web content in a remote, isolated environment and streams only pixels to the user's browser, preventing active content from ever reaching the endpoint.

Related:

- [[ZIA]]
- [[ZIA Policy Design]]
- [[Policy Evaluation]]

---

# Purpose

Some destinations cannot be confidently classified as safe or malicious. Rather than outright blocking access (hurting productivity) or allowing it (accepting risk), Browser Isolation lets users interact with the site while containing any threat.

---

# How It Works

```text
User Requests Site
        │
        ▼
Isolation Policy Match
        │
        ▼
Remote Browser Session
        │
        ▼
Rendered Pixel Stream
        │
        ▼
User's Local Browser
```

No active content (JavaScript, downloads, plugins) executes directly on the endpoint.

---

# Common Use Cases

## Uncategorized Websites

Sites without an established reputation.

## High-Risk Categories

Categories with elevated risk, such as newly registered domains.

## Contractor and BYOD Access

Providing safer access for less-trusted device populations without full network access.

## Isolate Instead of Block

Preserving access to borderline-risk sites without a hard block.

---

# Policy Design Considerations

## Isolation vs. Block vs. Allow

Choose isolation for content that is risky but has legitimate business value; reserve block for clearly malicious categories.

## Download and Upload Controls

Decide whether file transfer is permitted within isolated sessions, and if so, whether it routes through additional inspection (e.g. [[Cloud Sandbox]]).

## Read-Only vs. Interactive Sessions

Some use cases (e.g. contractor access to sensitive sites) may warrant read-only isolation.

---

# Benefits

- Contains malware execution away from the endpoint
- Reduces risk from unknown or uncategorized sites
- Preserves user productivity compared to outright blocking

---

# Troubleshooting

## Isolation Not Triggering

Check:

1. URL category classification
2. Isolation rule order and criteria
3. User/group scope

## Site Functionality Issues in Isolation

Check:

1. Application compatibility with remote rendering
2. Download/upload policy restrictions

---

# Best Practices

- Apply isolation to uncategorized and elevated-risk categories rather than blocking outright
- Restrict file transfer within isolated sessions for higher-risk populations
- Review isolation logs to refine category scope over time

---

# Related Notes

- [[ZIA]]
- [[ZIA Policy Design]]
- [[Policy Evaluation]]
- [[Cloud Sandbox]]
