# PAC Files

## Overview

A **PAC (Proxy Auto-Configuration) file** is a JavaScript file that tells a browser which proxy — if any — to use for a given URL.

Related:

- [[Traffic Forwarding]]
- [[ZIA]]
- [[ZCC]]

---

> [!important] ZDTE exam weight
> PAC files appear in **four of six** blueprint domains:
> - **Architecture and Design** — design a PAC deployment strategy for globally dispersed users
> - **Implementation and Deployment** — identify a correct PAC file to accomplish a scenario
> - **Troubleshooting and Support** — investigate a PAC file for mismatched logic
> - **Automation and Optimization** — optimize the logic of a PAC file
>
> Expect to *read* PAC JavaScript and pick the right one, or spot why one is broken. Practice reading, not memorizing.

---

# Required Structure

Every PAC file defines one function. The browser calls it for each request.

```javascript
function FindProxyForURL(url, host) {
    return "DIRECT";
}
```

## Return Values

|Return|Meaning|
|---|---|
|`DIRECT`|Bypass proxy, connect directly|
|`PROXY host:port`|Use this proxy|
|`PROXY a:80; PROXY b:80`|Try a, fall back to b|
|`PROXY a:80; DIRECT`|Try proxy, fall back to direct|

**Exam-relevant:** semicolon-separated entries are a *failover chain*, evaluated left to right. `DIRECT` placed last is a fail-open; omitting it is fail-closed.

---

# Evaluation Order Is the Whole Game

PAC logic is **top-down, first match wins, then `return` exits**. Nearly every "mismatched logic" troubleshooting scenario is one of these:

## Failure Pattern 1: Unreachable rule

```javascript
if (isPlainHostName(host)) return "DIRECT";
if (host == "intranet.company.com") return "DIRECT";
```

The second line is dead if the first already matched — order specific rules before general ones.

## Failure Pattern 2: Broad match swallowing an exception

```javascript
if (dnsDomainIs(host, ".company.com")) return "PROXY gateway:80";
if (host == "vpn.company.com") return "DIRECT";
```

`vpn.company.com` never reaches line 2. **Exceptions must come first.**

## Failure Pattern 3: Missing default

If no branch returns, behavior is undefined/inconsistent. Always end with an explicit `return`.

---

# Core Functions to Recognize

|Function|Matches|
|---|---|
|`isPlainHostName(host)`|No dots — e.g. `intranet`|
|`dnsDomainIs(host, ".x.com")`|Host is in that domain|
|`shExpMatch(url, "*pattern*")`|Wildcard match on URL or host|
|`isInNet(host, ip, mask)`|Host resolves inside a subnet|
|`myIpAddress()`|Client's own IP|
|`isResolvable(host)`|DNS resolves|
|`weekdayRange()` / `timeRange()`|Time-based branching|

---

# Optimization (Automation domain, 11%)

The exam asks you to make PAC logic *more efficient*. The optimization principles:

## 1. Avoid DNS-dependent functions

`isInNet()`, `isResolvable()`, and `dnsResolve()` force a DNS lookup on **every request**, blocking page load. This is the number one PAC performance problem.

**Slow:**
```javascript
if (isInNet(host, "10.0.0.0", "255.0.0.0")) return "DIRECT";
```

**Faster — string match, no DNS:**
```javascript
if (shExpMatch(host, "10.*")) return "DIRECT";
```

## 2. Order by frequency

Put the most commonly matched rules at the top so most requests exit early.

## 3. Collapse repeated conditions

**Before:**
```javascript
if (dnsDomainIs(host, ".a.com")) return "DIRECT";
if (dnsDomainIs(host, ".b.com")) return "DIRECT";
if (dnsDomainIs(host, ".c.com")) return "DIRECT";
```

**After:**
```javascript
if (shExpMatch(host, "*.a.com") ||
    shExpMatch(host, "*.b.com") ||
    shExpMatch(host, "*.c.com")) return "DIRECT";
```

## 4. Cache `myIpAddress()`

Assign it once to a variable rather than calling it in multiple branches.

---

# Global Deployment Strategy

For **globally dispersed users** (an explicit blueprint objective), the design question is how each user reaches the *nearest* enforcement node.

Common approaches:

- **Zscaler-hosted PAC** — Zscaler serves a PAC that resolves the user to a geographically appropriate node automatically. Lowest maintenance.
- **Custom PAC with `myIpAddress()` branching** — route by source subnet to a regional gateway. More control, more maintenance.
- **Per-location PAC URLs** — different PAC per region, distributed via GPO/MDM.

Design tradeoff to be ready to articulate: hardcoding regional proxies gives control but breaks when users travel; Zscaler-hosted geo-resolution handles roaming automatically.

---

# Typical Bypass Categories

Content commonly returned as `DIRECT`:

- Internal/RFC1918 destinations
- Certificate-pinned applications
- Authentication endpoints (avoid a chicken-and-egg loop)
- Real-time media where inspection adds latency

---

# Troubleshooting Workflow

## Some users/locations can't reach an app

1. Confirm which PAC the client actually loaded (they may differ by location)
2. Walk the logic top-down for the failing host — find the **first** matching line
3. Check whether an earlier broad rule shadows the intended rule
4. Verify the return value is a reachable proxy and port
5. Check for a missing trailing default return

---

# Related Notes

- [[Traffic Forwarding]]
- [[ZIA]]
- [[ZCC]]
- [[Troubleshooting Methodology]]
