# Cloud Sandbox

## Overview

**Cloud Sandbox** is a ZIA security service that analyzes unknown or suspicious files in an isolated environment to determine whether they are malicious before allowing them to reach the user.

Related:

- [[ZIA]]
- [[ZIA Policy Design]]
- [[DNS Security]]

---

# Purpose

Signature-based detection cannot catch previously unseen threats. Cloud Sandbox addresses this gap by executing and observing file behavior.

---

# Architecture

```text
File Download
      │
      ▼
Known Threat?
      │
 ┌────┴────┐
 │         │
Yes        No
 │         │
Block   Sandbox Analysis
             │
             ▼
        Malicious?
             │
       ┌─────┴─────┐
       │           │
     Yes           No
       │           │
     Block       Deliver
```

---

# Supported Content

- Executables
- Office documents
- PDFs
- Archives

---

# Analysis Techniques

## Static Analysis

Examines file structure, metadata, and embedded content without executing it.

## Dynamic Analysis

Executes the file in an isolated environment and observes its behavior (network calls, file system changes, process creation).

## Threat Intelligence Correlation

Compares file characteristics and behavior against known threat indicators.

---

# Policy Design Considerations

## First-Seen File Handling

Decide whether first-seen files are held pending analysis or delivered immediately with retroactive blocking.

## File Type Scope

Not all file types require sandboxing; scope inspection to executable and document types most likely to carry threats.

## Quarantine vs. Block

Choose whether suspicious files are blocked outright or quarantined for review.

---

# Troubleshooting

## Expected Detection Not Occurring

Check:

1. File type inclusion in sandbox policy
2. Sandbox rule order
3. SSL Inspection coverage (required to inspect downloaded content over HTTPS)

## Delayed File Delivery

Check:

1. Sandbox analysis mode (hold vs. deliver-then-verify)
2. File size and analysis queue

---

# Best Practices

- Enable sandboxing for all executable and macro-enabled document types
- Pair with [[SSL Inspection]] to ensure encrypted downloads are analyzed
- Use hold mode for high-risk categories, deliver-then-verify for low-risk ones
- Review sandbox verdicts and tune exclusions periodically

---

# Related Notes

- [[ZIA]]
- [[ZIA Policy Design]]
- [[SSL Inspection]]
- [[DLP]]
