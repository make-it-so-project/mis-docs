---
read_when: working on mis-client implementation choices, client assurance levels, or MVP approval surface decisions
---

# ADR-0006: WebApp/PWA MVP Client Assurance

## Status

Accepted

## Date

2026-05-03

## Context

The make-it-so platform requires a trusted approval surface (mis-client)
through which users receive and act on approval requests.

For the MVP, a native iOS or Android application would provide the highest
assurance: hardware-backed key storage, OS-enforced app identity, and
platform-level device binding. However, native app development requires
significantly more time and resources than a WebApp/PWA.

A WebApp/PWA can be delivered faster and is sufficient to validate the
core approval flow and user experience. However, a WebApp/PWA has
fundamentally different security properties than a native client.

The key question is whether a WebApp/PWA may be used as a mis-client
for the MVP, and if so, under what constraints.

---

## Decision Drivers

- MVP speed: a WebApp/PWA can be built and deployed more quickly
- Validation of the approval flow concept before native app investment
- Maintaining the integrity of the trusted approval surface model
- Preventing future contributors from treating all clients as equivalent
- Preserving the path to native high-assurance clients

---

## Considered Options

### Option 1: Build native iOS and Android clients immediately

Delay the MVP until native clients are ready.

#### Advantages
- maximum client assurance from the start
- hardware-backed key storage

#### Disadvantages
- high development cost and time
- delays validation of the core concept
- premature investment before product-market fit

#### Assessment
Too costly for the MVP phase. Appropriate as a future target.

---

### Option 2: Use only a WebApp/PWA without explicit assurance distinction

Treat all clients identically regardless of client type.

#### Advantages
- simple data model; no assurance levels to manage

#### Disadvantages
- masks a meaningful security difference
- prevents policy decisions from depending on assurance level
- may lead to over-trusting WebApp/PWA clients in sensitive contexts
- creates technical debt when native clients are introduced

#### Assessment
Unacceptable. The assurance distinction must be explicit from the start
to prevent the model from degrading silently.

---

### Option 3: Use WebApp/PWA for MVP with explicit client type and assurance level (selected)

Allow a WebApp/PWA client for the MVP, but mark it explicitly with
`client_type = web_pwa` and assign an explicit `assurance_level`.

#### Advantages
- enables MVP delivery without delaying the project
- maintains the trusted approval surface model by making client type explicit
- policy decisions can depend on `assurance_level`
- native clients can be introduced later without changing the model

#### Disadvantages
- requires defining and enforcing `client_type` and `assurance_level` from the start
- some approval scenarios may require native-level assurance in the future

#### Assessment
Best balance of MVP speed and architectural integrity.

---

## Decision

The MVP MAY implement mis-client as a WebApp/PWA.

WebApp/PWA clients MUST be explicitly marked:

```
client_type     = web_pwa
assurance_level = basic
```

### Assurance Model

| Level | Meaning |
|---|---|
| `basic` | WebApp/PWA with browser-managed key storage |
| `standard` | Native app with OS-managed key storage |
| `high` | Native app with hardware-backed key storage (e.g., Secure Enclave, StrongBox) |

For the MVP, `basic` assurance is acceptable for initial low- to medium-risk
approval flow validation. High-risk production approvals may require stronger
step-up or higher-assurance native clients.

### Client Type Model

| Type | Description |
|---|---|
| `web_pwa` | WebApp or Progressive Web App running in a browser |
| `ios_native` | Native iOS application |
| `android_native` | Native Android application |

### Policy Implications

- The Policy Engine MAY evaluate `assurance_level` when determining
  approval requirements for high-risk actions.
- Future policy rules MAY require `assurance_level = high` for specific
  action types, effectively requiring a native client.
- High-risk approvals MAY additionally require step-up authentication
  regardless of `assurance_level`.

### Path to Native Clients

Native iOS and Android clients remain the target for future high-assurance
deployments. Introducing native clients does not require changing the
existing data model — `client_type` and `assurance_level` are already
defined to accommodate them.

---

## Consequences

### Positive Consequences

- MVP can proceed with a WebApp/PWA without compromising the architectural model
- `client_type` and `assurance_level` are explicit from the start
- policy rules can evolve to depend on assurance level
- native clients can be added without model changes

### Negative Consequences

- `web_pwa` clients offer lower assurance than native clients
- browser storage can be cleared, destroying the client identity
- non-extractable key storage is browser-dependent and not guaranteed

### Follow-up Implications

- native iOS and Android clients should be built before high-risk
  approval scenarios are enabled in production
- a policy for which action types require which `assurance_level` should
  be defined in a future ADR

---

## Rationale Summary

A WebApp/PWA with explicit `client_type` and `assurance_level` markers
enables MVP delivery while preserving the integrity of the trusted approval
surface model. Masking the assurance difference would create silent
technical debt. Making it explicit allows the policy model to evolve safely.
