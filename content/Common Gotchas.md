# Common Gotchas

## Overview

Field-reported friction points with Zscaler deployments, drawn from public sources — vendor documentation, official product docs from third parties (Docker), and public user/admin reviews. Each entry includes the fix or workaround as documented.

This note is deliberately separate from [[Troubleshooting Methodology]]: that note teaches the *diagnostic process*. This one is a lookup table of specific, recurring, known issues.

Related:

- [[SSL Inspection]]
- [[ZCC]]
- [[ZIA Policy Design]]
- [[Traffic Forwarding]]

---

> [!important] ZDTE exam weight
> Real-world gotchas and blueprint objectives overlap more than you'd expect — the exam draws its troubleshooting scenarios from exactly this kind of field-reported friction, not abstract hypotheticals. Each section below calls out which domain and objective it maps to. One section (General Friction) doesn't map to a scored objective at all — flagged honestly rather than forced, since not everything that matters operationally is something the exam tests.

---

# SSL Inspection Breaks Developer Tooling

**By far the most consistently reported friction point.**

## The Problem

Many developer tools maintain their own certificate trust store instead of using the operating system's — Git, npm, pip, Python's `requests` library, Java applications, Node.js, curl, Docker, and CI/CD agents commonly ship a bundled CA list rather than reading the OS trust store. When SSL Inspection decrypts and re-signs traffic with the Zscaler root certificate, these tools reject the connection because their private bundle doesn't recognize it.

Typical errors:

```text
certificate verify failed
unable to get local issuer certificate
x509: certificate signed by unknown authority
self signed certificate in certificate chain
```

Docker is affected seriously enough that **Docker's own official documentation** now includes a dedicated guide for this exact scenario.

## Fix / Workaround

Import the Zscaler root CA into the **specific trust store the failing tool actually uses** — not just the OS store:

|Tool|Where to add the cert|
|---|---|
|Docker|Copy into build context, `update-ca-certificates` in the image, or `/etc/docker/certs.d`|
|Git|Configure `http.sslCAInfo` to a bundle including the Zscaler root, or check whether it's using Schannel (Windows), OpenSSL, or a custom CA file|
|npm|Set the CA config to a bundle including the Zscaler root — avoid `strict-ssl=false` as a fix, it disables verification entirely|
|Python (pip/requests)|Add to the `certifi` bundle path, or set `REQUESTS_CA_BUNDLE`|
|Java|Import into the active `cacerts` file used by the JRE/JDK — the OS store is not consulted|
|Node.js|Set `NODE_EXTRA_CA_CERTS` to a file including the Zscaler root|

Zscaler maintains predefined **"Developer Tools"** and **"System Tools"** URL categories specifically to make it easier to identify and exempt or correctly certificate-provision this traffic.

**Escalation signal:** if the root cert is missing from a managed device, keeps disappearing, is expired, or differs between machines, that's a deployment/MDM problem, not a per-tool config problem — stop patching individual tools and check certificate distribution.

> [!important] ZDTE tie-in
> **Security and Compliance (14%)** — SSL Inspection exception design is squarely in this domain. Certificate-pinned and trust-store-isolated applications are the textbook case for an SSL Inspection **exclusion**, not a per-device workaround. If a scenario describes a whole category of tools breaking under inspection, the exam-correct answer is usually "add an SSL Inspection exception for that category," not "fix each app."
>
> **Troubleshooting and Support (15%)** — a certificate-chain error like the ones above is one of the clearest "which layer failed" signals in [[Troubleshooting Methodology]]: a cert error means the client *reached* Zscaler and SSL Inspection is actively intercepting — a fundamentally different failure than a timeout or a block page.

---

# Client Connector Conflicts With Other Network Software

## The Problem

ZCC installs as a Layered Service Provider (LSP) in the Windows network stack. If another VPN client or security agent also registers as a layered provider, the two can compete over the same traffic flows, corrupting packets rather than cleanly cooperating.

## Fix / Workaround

- Order the network agents deliberately (LSP chain order matters)
- Configure the third-party VPN to **split-tunnel** so it doesn't attempt to grab Zscaler's own traffic
- Add ZCC's processes to an allowlist in any AV/EDR product, and exclude Zscaler's own domains from any *other* SSL inspection product running simultaneously — inspecting Zscaler's inspection traffic is a common self-inflicted failure

> [!important] ZDTE tie-in
> **Operations and Monitoring (16%)** — this is a direct instance of the blueprint's ZCC/ZEN connectivity monitoring objective: *"given a scenario about ZCC and ZEN connectivity monitoring, identify the action that should be taken based on the monitoring information."* Symptom pattern to recognize: tunnel intermittently up/down or throughput degraded with no clean root cause in Zscaler's own logs — the actual cause is sitting one layer down, in the OS network stack, competing with another agent.
>
> **Troubleshooting and Support (15%)** — a good example of a failure that *isn't* a policy problem at all. Localizing "which layer" per [[Troubleshooting Methodology]] correctly rules out ZIA/ZPA policy early here — the symptom shows up as generic instability, not a block page or denial, which is itself a diagnostic clue.

---

# Roaming Users Stuck on a Distant Node

## The Problem

A user can end up authenticated to a Zscaler enforcement node in the wrong region entirely — for example, a user physically in one country remaining pinned to a node on a different continent after travel, due to DNS resolution behavior or ISP routing choices made before the location changed.

## Symptom

General, otherwise-unexplained latency. Not a bandwidth problem — extra hops and geographic distance are the cause, and no local network troubleshooting fixes it.

## Fix / Workaround

Check `ip.zscaler.com` — it reports which city's node the session is actually connected to. If that city is far from the user's real location:

- Force reconnection (restart ZCC, or disconnect/reconnect)
- Check for stale DNS caching pointing at an old regional resolution
- In persistent cases, this is a Traffic Forwarding / geo-resolution configuration issue worth escalating rather than repeatedly reconnecting

> [!important] ZDTE tie-in
> **Architecture and Design (22%)** — this is precisely the failure mode the blueprint's global-deployment objective is testing for: *"design a PAC deployment strategy for globally dispersed users."* A hardcoded regional PAC assignment is what causes a traveling user to stay pinned to the wrong node; Zscaler-hosted geo-resolution is the design that self-corrects. If a scenario describes a user "stuck" on a distant node after relocating, the design-level answer is about **how nodes get selected**, not a one-time fix for that user.
>
> **Operations and Monitoring (16%)** — reading `ip.zscaler.com` output and correctly interpreting "wrong region = latency, not bandwidth" is exactly the ZCC/ZEN monitoring-interpretation skill the exam tests, distinct from the design question above.

---

# Real-Time Media (Zoom / Teams) Degradation

## The Problem

Voice and video traffic routed through full SSL inspection adds latency and processing overhead that real-time media is especially sensitive to — choppy calls, dropped audio, frozen video.

## Fix / Workaround

- Apply Zscaler's published UCaaS bypass list so Teams/Zoom media traffic goes **direct**, bypassing the tunnel entirely rather than being inspected
- If media traffic must stay tunneled for policy reasons, **lower the tunnel MTU** to match the actual discovered path MTU — fragmentation from an oversized MTU is a common secondary cause of choppy real-time media even after bypass is configured

See [[Traffic Forwarding]] for split tunnel / bypass mechanics generally.

> [!important] ZDTE tie-in
> **Implementation and Deployment (22%)** — bypass/exception design for latency-sensitive traffic sits directly under the blueprint's traffic-forwarding-method objectives. Knowing that certain traffic classes are *designed* to bypass rather than be forced through inspection is part of choosing the right forwarding method for a given population.
>
> **Automation and Optimization (11%)** — MTU tuning is an optimization lever, not a policy change, and it's easy to overlook. The exam's optimization objectives reward recognizing when the fix is a *transport-layer* parameter rather than a rule change — the same instinct tested by PAC logic optimization in [[PAC Files]].

---

# Network Transition Dropouts (Wired → Wi-Fi / Hotspot)

## The Problem

Switching a laptop from a wired connection to Wi-Fi or a mobile hotspot can leave ZCC "stuck" mid-transition, with the user losing internet access entirely for up to a minute while the client re-establishes forwarding. Reported as less seamless on macOS and Linux than on Windows.

## Fix / Workaround

- No universal client-side fix documented publicly; treat as a known limitation of software-based network breakouts versus hardware-based ones
- If this happens consistently for a specific user, check Trusted Network Detection settings — a TND re-evaluation triggered by the network change can be part of what's stalling reconnection
- Keep ZCC updated — this class of issue is the kind vendors iterate on across client releases

> [!important] ZDTE tie-in
> **Troubleshooting and Support (15%)** — a timeout-style symptom (not a block page) tied to a specific trigger event (network change) is a good drill for the "localize before you fix" discipline in [[Troubleshooting Methodology]]. The trigger event itself — network transition — points straight at Trusted Network Detection re-evaluation as the first thing to check, ahead of anything policy-related.
>
> **Implementation and Deployment (22%)** — indirectly touches ZCC profile/update management: staying current on client version is the only broadly-documented mitigation, which ties back to the blueprint's update-policy objective covered in [[ZCC]].

---

# Cloud App Control Can Silently Override URL Filtering

## The Problem

This isn't a bug — it's documented, non-obvious behavior that catches admins off guard. If a Cloud App Control rule **explicitly allows** a cloud application, Zscaler applies **only** the Cloud App Control policy for that request. A URL Filtering rule blocking the same destination does not also get evaluated.

## Symptom

A URL Filtering block rule exists, looks correctly configured, and simply doesn't work — for one specific cloud app.

## Fix / Workaround

Check Cloud App Control for an explicit allow on the same application before assuming the URL Filtering rule is broken. If both controls are needed (allow the app, but restrict specific actions within it), do the restricting **inside** Cloud App Control's own action-level rules (block upload, allow view) rather than relying on a URL Filtering rule as a backstop — it may never be evaluated.

See the correction and full evaluation order in [[ZIA Policy Design]] and [[ZIA]].

> [!important] ZDTE tie-in
> **Implementation and Deployment (22%)** — this is the exact scenario behind the blueprint's named objective: *"given a scenario, identify if URL Policy, CloudApp Policy, or some combination is the most appropriate filtering."* This gotcha is what happens when that choice is made incorrectly, or when the two engines' interaction isn't understood — treating them as independent, additive layers is the wrong mental model.
>
> **Troubleshooting and Support (15%)** — maps directly onto *"given a scenario in which legitimate access to an app is being prevented by a block policy... identify where the blocking happened."* The inverse case here (a block that should be happening but isn't) uses the same skill in reverse: identify which engine actually has authority over the decision before troubleshooting the one that looks broken.

---

# DLP Rule Order Isn't Always What Decides the Outcome

## The Problem

By default, DLP rules evaluate top-down with the **first match** winning — standard first-match-wins behavior. But if an organization has **"Evaluate All Rules" mode** enabled, behavior changes: all matching rules are evaluated, and the **most restrictive action wins** regardless of order. Rule order only becomes the deciding factor as a tiebreaker between multiple rules that share the same action.

## Fix / Workaround

Before troubleshooting "wrong DLP rule triggered" by reordering rules, confirm which evaluation mode is active. Reordering rules has no effect on the outcome under Evaluate All Rules mode unless the competing rules already share an action.

> [!important] ZDTE tie-in
> **Implementation and Deployment (22%)** — directly under the blueprint's DLP setup objective: *"given a scenario about setting up ZIA DLP policies, identify how to set up and enable the policies."* Evaluation mode is a setup-time decision with runtime consequences — worth confirming as step zero in any DLP scenario, not an afterthought.
>
> **Exam trap to note explicitly:** this is one of the few places in the whole platform where "first match wins" — the default assumption drilled throughout [[ZIA Policy Design]] and [[PAC Files]] — is not automatically true. A DLP question that doesn't mention the evaluation mode is implicitly asking you to know the *default* (first-match), but a question that describes multiple rules all seemingly relevant is a signal to consider whether Evaluate All Rules is in play.

---

# General Friction (Cost, Complexity, UI)

Recurring but less technical complaints worth being aware of when scoping a deployment:

- **Latency overhead** — traffic routing through cloud inspection points before reaching its destination is an inherent architectural cost, more noticeable for bandwidth-heavy or latency-sensitive workloads
- **Configuration complexity** — initial policy setup and integration with legacy infrastructure is commonly described as a steep initial lift
- **Cost** — frequently described as expensive relative to organizations that don't need the full feature set
- **Troubleshooting opacity** — admins report that determining *why* a specific site was blocked can require checking multiple engines (URL Filtering, Cloud App Control, Cloud Firewall, DLP) rather than one obvious place — reinforcing the value of checking logs to identify which engine actually matched, per [[Troubleshooting Methodology]]

> [!note] Not a scored ZDTE topic
> Unlike every other section in this note, none of these map cleanly to a specific blueprint objective — they're operational and business context, not testable technical scenarios. Included for completeness rather than exam prep. The one exception worth a mental note: "troubleshooting opacity" is really just an informal restatement of why [[Troubleshooting Methodology]]'s layer-localization approach exists in the first place.

---

# Related Notes

- [[SSL Inspection]]
- [[ZCC]]
- [[Traffic Forwarding]]
- [[ZIA Policy Design]]
- [[ZIA]]
- [[DLP]]
- [[PAC Files]]
- [[Troubleshooting Methodology]]
