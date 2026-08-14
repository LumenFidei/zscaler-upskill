# Data Loss Prevention (DLP)

## Overview

**Data Loss Prevention (DLP)** is a ZIA security service that inspects outbound traffic to detect and prevent sensitive data from leaving the organization.

DLP evaluates content within web traffic, uploads, and SaaS transactions against defined data classification rules.

Related:

- [[ZIA]]
- [[ZIA Policy Design]]
- [[SSL Inspection]]

---

# Purpose

Prevent exposure or exfiltration of:

- Financial data
- Personal data
- Intellectual property
- Regulated data (healthcare, payment card, etc.)

---

# Architecture

```text
Outbound Traffic
       │
       ▼
SSL Decryption
       │
       ▼
Content Extraction
       │
       ▼
DLP Engine
       │
       ▼
Policy Match
       │
       ▼
Allow / Block / Quarantine / Notify
```

DLP inspection depends on [[SSL Inspection]] to see encrypted content.

---

# Detection Engines

## Exact Data Match (EDM)

Matches specific structured records (e.g. customer database rows) rather than generic patterns.

## Indexed Document Match (IDM)

Fingerprints specific documents so any partial reuse is detected.

## Pattern and Regex Matching

Detects structured identifiers such as:

- Credit card numbers
- Social Security Numbers
- Passport numbers

## Predefined Dictionaries

Built-in keyword and phrase sets for common data types.

## Machine Learning Classifiers

Identify sensitive content types (e.g. source code, financial statements) without exact matching.

---

# Data Types

## Personal Information

- SSNs
- National IDs
- Passport numbers

## Financial Information

- Credit card numbers
- Banking details

## Intellectual Property

- Source code
- Proprietary documents
- Engineering designs

---

# Policy Components

- Rule criteria (users, groups, destinations)
- Data classification engine
- Severity
- Action
- Notification settings

---

# Actions

|Action|Result|
|---|---|
|Allow|Permit transfer|
|Block|Prevent transfer|
|Quarantine|Hold content for review|
|Notify|Alert user or admin|

---

# Common Use Cases

- Blocking upload of source code to personal cloud storage
- Detecting SSNs in outbound email or web forms
- Preventing exfiltration of fingerprinted confidential documents

---

# Troubleshooting

## Expected Block Not Occurring

Check:

1. SSL Inspection coverage for the destination
2. Rule order and criteria
3. Engine configuration (EDM/IDM/dictionary)
4. Exclusions

## False Positives

Check:

1. Dictionary sensitivity
2. Pattern specificity
3. Rule scoping

---

# Best Practices

- Start in monitor/log-only mode before enforcing blocks
- Classify data by sensitivity tier
- Combine multiple engines for coverage
- Review DLP logs and tune rules regularly
- Ensure SSL Inspection is enabled for DLP-relevant categories

---

# Related Notes

- [[ZIA]]
- [[ZIA Policy Design]]
- [[SSL Inspection]]
- [[Cloud Firewall]]
