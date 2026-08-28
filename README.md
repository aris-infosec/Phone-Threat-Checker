# Phone-Threat-Checker

### Your Phone Number Is a Master Key: An OSINT/Correlation Threat Model

*Notes from an ongoing cybersecurity professional.*

---

## Table of Contents

- [Main Argument](#main-argument)
- [Why Phone Numbers Expose Your Identity](#why-phone-numbers-expose-your-identity)
- [The Proposed Solution](#the-proposed-solution)
- [Three Levels of Phone-Number Privacy](#three-levels-of-phone-number-privacy)
- [Which Level to Use](#which-level-to-use)
- [Platform-Specific Constraints](#platform-specific-constraints)
- [Beyond the Number: Device & Pattern-Level Exposure](#beyond-the-number-device--pattern-level-exposure)
- [Practical Europe-Specific Solutions](#practical-europe-specific-solutions)
- [The Security/Convenience Trade-off](#the-securityconvenience-trade-off)
- [Overall Message](#overall-message)

---

## Main Argument

**Phone numbers have become digital identity keys.**

- A phone number is no longer just for calls and texts — it's a persistent, hard-to-rotate identifier (MSISDN) that platforms use to bind online activity to real-world identity.
- Unlike a password or session token, you can't easily rotate a phone number without real-world cost (losing contacts, re-verifying dozens of accounts). That's what makes it structurally different from other credentials — and why it's such an effective long-term tracking key.

The biggest concern is when services require a phone number for:

| Use Case | Risk |
|---|---|
| Login | Direct identity binding |
| 2FA / OTP | Ties recovery to a single point of failure |
| Account recovery | Cross-links accounts |
| KYC-style verification | Embeds legal identity |

Each use case embeds the number into a different backend system, so a single number can end up seeding dozens of independent identity records that all point back to the same person.

---

## Why Phone Numbers Expose Your Identity

Numbers are carrier-registered with KYC data (legal name, billing address, sometimes government ID). This binding happens once, at SIM activation, and typically doesn't change even as you switch apps, devices, or operating systems.

Platforms/brokers can correlate your number with:

- **Real name and address**
- **Other accounts (identity graphs)** — e.g., the same number used to sign up for a shopping site and a dating app links those two accounts even if the usernames are unrelated
- **Uploaded contact lists (shadow profiles)** — someone who has you saved in their phone and uploads their contacts can create a profile of you even if you've never used that platform
- **Data-broker records** — aggregated from breaches, marketing lists, and public filings
- **Social/professional network** — repeated co-location with the same numbers over time infers relationships even without either party disclosing them

Once a number is tied to an account, activity on it becomes trivially attributable to you.

> **Access model is usually purchase, not breach.** Brokers buy bulk ad-tech/location data from apps you already gave permission to. A buyer rarely queries *"give me Jane Doe's number"* — instead they buy *"everything associated with number X"* and let the correlation happen after the fact. This exposure exists **legally and by design** — no hacking or court order required, just a commercial transaction.

---

## The Proposed Solution

**Separate your real number from your online identity.**

- Keep your normal number for people/orgs you already trust — this number shouldn't appear in any online form, sign-up flow, or public listing.
- Create a **private/secondary virtual number** for:
  - Websites
  - 2FA
  - New contacts
  - Deliveries
  - Side projects / untrusted services

**Goal:** stop your primary number from being the universal join key across accounts — every service that only ever sees the virtual number can't contribute to a profile that resolves back to your real identity.

This is a segmentation strategy, conceptually identical to using separate credentials/API keys per service instead of one master key reused everywhere.

---

## Three Levels of Phone-Number Privacy

### 🟢 Level 1 — Private Virtual Number + Forwarding

Get a VoIP/DID number, forward calls/SMS to your real phone.

**Use for:** 2FA, registrations, deliveries, non-critical services.

**Advantages**
- Easy setup — no new app or device required
- Real number stays hidden from every service using this number
- Forwarding is transparent — you keep using your phone as normal

**Limitations**
- Calling back from your real line leaks your real caller ID — this is **inbound-only** protection, not two-way
- The VoIP provider account becomes critical infrastructure — secure it with a unique password and hardware/TOTP MFA, **not SMS-based MFA**
- Choose a provider that doesn't forward SMS content to your email inbox — that reintroduces the correlation you're trying to avoid

### 🟡 Level 2 — Full Virtual Phone Line

Operate the private number via a SIP/VoIP app instead of forwarding — genuine two-way use.

**Requires:** data/Wi-Fi, VoIP service, SIP app, credentials.

Effectively two independent lines on one device — like running two separate identities in parallel rather than routing one through the other. No outbound caller-ID leak, since calls originate from the SIP app directly.

**New attack surface at this level**
- SIP credential security — treat like any other account login
- Softphone app vulnerabilities — keep updated; prefer providers with SRTP/TLS-encrypted transport
- Provider-side compromise remains the single point of failure, now with higher stakes

### 🔴 Level 3 — Full Isolation

No traditional carrier voice/SMS line at all. Wi-Fi or data-only SIM; all comms via VoIP/SIP + virtual numbers.

A data-only SIM still has a device identity (IMSI) but no voice/SMS routing (MSISDN) tied to it — this specifically reduces exposure to attacks targeting the voice/SMS signaling layer.

**Advantages**
- Maximum separation from the conventional phone network
- Harder to reverse-lookup via standard phone-number-to-identity databases
- Can run multiple virtual numbers in parallel for different contexts (work, personal, high-risk research/contacts)

**Summary**

| Level | Description | Effort |
|---|---|---|
| 1 | Forwarding, easiest setup | Low |
| 2 | Full two-way virtual line | Medium |
| 3 | Removes traditional number entirely | High |

---

## Which Level to Use

| Profile | Recommended Level | Why |
|---|---|---|
| Most people | **Level 1** | Covers the highest-volume, lowest-effort correlation risk for minimal setup cost |
| Multiple identities (business/personal) | **Level 2** | Worth it once actively communicating under more than one identity |
| High-threat individuals | **Level 3** | Appropriate for journalists, researchers on sensitive topics, executives — anyone facing a motivated, resourced adversary |

---

## Platform-Specific Constraints

- Some platforms reject VoIP numbers outright — commonly **Meta, Google, Discord**.
  - **Why:** VoIP numbers are cheap and disposable, a favorite for spam/bot/fraud accounts — platforms reject them to protect fraud models and identity-graph fidelity.
  - If rejected, make an explicit decision: give the real number, or skip the service — don't default to disclosure out of friction-avoidance.
- **Financial institutions:** use your real number. They already hold your legal identity for KYC/AML purposes, so compartmentalizing adds negligible benefit — and a VoIP number can actively trigger fraud-detection friction.
- **Google exception:** a hardware security key (FIDO2/WebAuthn) can replace phone-based verification in many flows post-enrollment.

---

## Beyond the Number: Device & Pattern-Level Exposure

### Device Identifiers

- **IMEI** — broadcast to cell towers regardless of app-level privacy settings; a telecom-layer signal you can't disable from settings
- **Ad-tech IDs** (IDFA / GAID) — embedded in ad SDKs, independent of your cellular signal; you can be profiled even on Wi-Fi-only with cellular disabled
- **On-device auth tokens** — create yet another identifier chain, separate from your number and ad ID
- **Passive "Find My"-style beacons** — low-power signals that can persist and be located even when the device is powered off

### Pattern of Life

- Predictable routines (same gym, same commute, same coffee shop) let an adversary forecast your location with high confidence — the physical-world analog of a weak, guessable password.
- **Mitigation:** rotate routine elements irregularly. Doesn't need to be daily — just frequent enough that no single observation window reveals a stable rule.
- Scale to your actual exposure — public figures and high-visibility individuals benefit disproportionately more.

### Public-Record / Address Exposure

Home address, property ownership, and vehicle registration are often publicly searchable by default in many jurisdictions.

**Fixes**
- Broker opt-out/removal services (effectiveness varies by jurisdiction)
- Statutory address-confidentiality programs, available for at-risk professions
- **Misattribution** — a tradecraft term for making an asset appear linked to you without you actually using it:
  - Keep the registered address on file, but don't actually live there
  - Same logic applies to vehicles — a registered vehicle that isn't your daily driver can't establish your real-time location

---

## Practical Europe-Specific Solutions

### Level 1 — EU-Friendly VoIP/DID Providers
- **Germany/EU:** Sipgate — local DID numbers with call/SMS forwarding
- **UK:** Voipfone — same purpose, under UK GDPR post-Brexit

When comparing providers, check: (a) legal jurisdiction, (b) whether SMS content is forwarded to email (avoid this), (c) TOTP/hardware-key MFA support vs. SMS-only.

### Level 2 — SIP/Softphone Apps
- **Linphone** (open-source) paired with a SIP trunk from an EU VoIP provider
- Prioritize SRTP/TLS support for encrypted call transport

### Level 3 — Privacy-Hardened Devices Available in Europe

| Device | Notes |
|---|---|
| **Pixel + GrapheneOS** | Strongest actively-maintained privacy/security OS combo as of 2026; self-flash or buy pre-flashed (e.g., NitroPhone) |
| **Fairphone + /e/OS (Murena)** | European-made, repairable, de-Googled OS pre-installed; lower-effort than self-flashing |
| **Volla Phone** | German-made, multi-boot de-Googled, EU-sovereignty focus |

Pair with a **data-only SIM** (ask for "data-only" or "IoT/M2M") + a VPN for network-layer separation.

### Address/Public-Record Removal (EU-Specific)
- **GDPR Article 17** (right to erasure) and **Article 15** (right of access) let EU residents request disclosure and deletion from data brokers
- Paid removal services (e.g., Incogni) exist for those who'd rather not file requests manually
- **UK residents:** UK GDPR via the Data Protection Act 2018, enforced by the ICO, functions similarly as a separate post-Brexit regime

> ⚠️ **Note on paid privacy services:** Treat "buy a privacy service" with the same scrutiny as any security purchase. A provider that itself becomes a centralized store of your sensitive data is a new single point of failure. Prefer providers with clear published data-retention and deletion policies.

---

## The Security/Convenience Trade-off

- Every measure above costs convenience for lower correlation risk — there's no free option; you're trading exposure against effort.
- You're always choosing where you sit on that line, whether deliberately or by default.
- Controls don't need to be permanent or absolute — scale posture to periods of elevated risk (role change, spike in visibility, credible threat) rather than adopting everything wholesale forever.

---

## Overall Message

> **Your phone number should not be the master key to your digital identity — and neither should your device fingerprint or your daily routine.**

**Basic strategy:**
1. Keep your real number private
2. Create a separate number for online services
3. Move 2FA off SMS onto TOTP or hardware keys
4. Vary predictable patterns; misattribute what you can't suppress

**Level 1 + baseline hygiene is the practical floor for most people.** Levels 2–3 and physical-pattern discipline scale up for higher-threat profiles.

---

*This is a personal knowledge base, not professional security advice. Adapt the threat model to your actual risk profile.*
