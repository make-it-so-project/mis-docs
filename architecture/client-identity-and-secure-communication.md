---
read_when: working on client authentication, key material, secure communication between mis-client and mis-backend, or WebApp/PWA client assurance
---

# Client Identity and Secure Communication

## Purpose

This document defines the cryptographic identity model for registered
mis-clients, and the secure communication requirements between
mis-client and mis-backend.

It specifies how a client proves its identity, how key material is
managed, what the backend validates, and what MUST NOT be transmitted
or stored.

---

## Client Identity Model

Each registered mis-client has a persistent cryptographic identity.

A client is identified by a `client_id` and a corresponding asymmetric
key pair. The backend stores the client's **public key** at registration.
The client retains the **private key** exclusively.

The client_id alone is not sufficient to act on behalf of a client.
The client MUST prove possession of the private key to authenticate
requests.

### Client Identity Data Model

```
mis_client:
  client_id            — unique identifier (assigned by backend at registration)
  user_id              — owning mis-user
  client_type          — web_pwa | ios_native | android_native
  assurance_level      — basic | standard | high
  public_key           — client public key (PEM or JWK format)
  key_algorithm        — e.g., EC P-256 or Ed25519
  key_created_at       — timestamp of key generation
  status               — pending | active | revoked
  display_name         — human-readable name for this device
  notification_endpoint — address for push/notification delivery
  created_at
  last_seen_at
  revoked_at
```

---

## Client Key Pair Model

Each mis-client generates an asymmetric key pair as part of enrollment.

| Key | Location | Transmittable |
|---|---|---|
| Public key | Registered in mis-backend | Yes — sent during registration |
| Private key | Stored in client only | MUST NOT be transmitted |

The private key MUST NOT leave the client under any circumstances.
The backend MUST NOT request, accept, or store the private key.

---

## WebApp/PWA-Specific Key Handling

A WebApp/PWA client MAY generate an asymmetric key pair using browser
cryptographic capabilities (e.g., the Web Cryptography API).

Requirements for WebApp/PWA key material:

- the key pair SHOULD be generated as non-extractable where the browser
  and platform support this
- the private key is stored in browser-controlled storage associated
  with the WebApp origin
- the private key MUST NOT be exported to JavaScript or transmitted
  to any server
- clearing browser storage, uninstalling the PWA, or changing the origin
  will destroy the private key and effectively revoke the client identity

Consequence: A WebApp/PWA client provides **persistent client identity**
as long as the browser storage remains intact. This is not equivalent to
native hardware-backed key storage.

A WebApp/PWA client MUST be registered with:
- `client_type = web_pwa`
- `assurance_level = basic` (unless an explicit upgrade path is defined)

See [ADR-0006](../adr/0006-webapp-pwa-mvp-client-assurance.md) and
[ADR-0007](../adr/0007-webcrypto-client-key-and-webauthn-step-up.md)
for the rationale and scope of WebApp/PWA assurance.

---

## Public Key Registration

During client registration, the client submits its public key to the
mis-backend as part of the enrollment request.

The backend:

- stores the public key associated with `client_id` and `user_id`
- records `key_algorithm` and `key_created_at`
- does NOT accept or store the private key
- associates the client with the correct assurance level based on
  `client_type` and key storage evidence

---

## Challenge/Response Proof of Possession

Before accepting a client as ACTIVE, and before processing sensitive
requests, the mis-backend MAY require the client to prove possession
of the private key via a challenge/response exchange:

```
mis-backend  →  challenge (random nonce)
mis-client   →  signature over challenge using private key
mis-backend  →  verifies signature using stored public key
```

A valid signature proves that the client holds the private key
corresponding to the registered public key.

Challenge/response proves **client key possession** only. It does NOT prove
current human presence or user verification. User step-up remains WebAuthn/passkey
or equivalent user verification — see the Two Identity Layers section below.

Challenge/response MUST be used:
- at registration activation (before setting status → ACTIVE)
- for proof of client key possession on sensitive operations where required by policy

---

## Signed Client Requests

The mis-backend does not trust a request merely because it contains a
`client_id` or arrives over an authenticated web session. For
security-sensitive actions, the backend MUST validate registered client
state and require proof that the caller holds the registered private key.

The backend verifies client signatures using the stored public key. This
prevents request forgery by an attacker who intercepts a session token
but does not hold the client private key.

### Which Requests Must Be Signed or Challenge-Bound

| Request | Requirement |
|---|---|
| Approve action | MUST be signed or challenge-bound |
| Deny action | MUST be signed or challenge-bound |
| Additional client registration confirmation | MUST be signed or challenge-bound |
| Client revocation | MUST be signed or challenge-bound |
| Session reactivation approval (`approval_required` mode) | MUST be signed or challenge-bound |
| Recovery-related client activation or trust reset | MUST be signed or challenge-bound |
| Any request that changes client trust or approval state | MUST be signed or challenge-bound |
| Sensitive client settings or notification endpoint changes | SHOULD be signed or challenge-bound |
| Non-sensitive reads, pending approval summaries, status checks | MAY rely on authenticated client session plus TLS |

### Canonical Signing Inputs

The signed material for a client request MUST include enough stable fields
to prevent request substitution or payload tampering.

Required fields in signed material:

- `client_id`
- `user_id`
- `method`
- `path`
- `request_id`
- `timestamp` or backend-issued `challenge_id` / nonce
- `body_hash`
- relevant resource identifier (e.g., `action_id`, `approval_request_id`)

For approval and denial decisions, the signed material MUST additionally bind:

- `approval_request_id`
- `action_id`
- `decision` (`approve` or `deny`)
- `details_hash` or `action_payload_hash`
- `decision_nonce` or `challenge_id`
- `expires_at`, if present

The exact canonicalization format is an implementation detail. The
architecture requires that enough stable fields are signed to prevent
substitution or tampering, not a specific serialization standard.

---

## Replay and Duplicate Protection

Replay protection is required even when HTTPS/TLS is used. Approval
decisions and client lifecycle mutations must be safe against duplicate
submission, retry ambiguity, and stale request reuse.

- Approval and denial decisions MUST be non-replayable.
- Security-sensitive client mutations MUST be non-replayable.
- Signed or challenge-bound requests MUST include `request_id` and either
  a `timestamp` or a backend-issued `nonce` / `challenge_id`.
- The mis-backend MUST reject duplicate `request_id` values within an
  appropriate replay window. Replay checks SHOULD be scoped to
  `user_id` + `client_id` + `request_id`.
- Approval decisions MUST be idempotent at the action/approval request level:
  a single `approval_request_id` MUST NOT be approved or denied more than once.
- Expired approval requests MUST NOT accept late decisions.

---

## Approval Payload Binding

An approval or denial decision must be bound to the exact approval request
the user was shown. This prevents substitution of a different action for
the one the user reviewed.

The mis-backend creates an `approval_request_id` for a specific action.
The approval request shown to the user is bound to stable action content.
The client's signed decision must refer to that `approval_request_id`.

Required fields in an approval request:

| Field | Purpose |
|---|---|
| `approval_request_id` | Unique identifier for this approval request |
| `action_id` | The action being approved or denied |
| `summary` | Human-readable action description |
| `details_hash` / `action_payload_hash` | Commitment to the action content |
| `risk_level` | Policy-determined risk classification |
| `agent_id` | Agent that submitted the action (if available) |
| `session_id` | Session in which the action was submitted (if available) |
| `user_id` | The mis-user who must approve |
| `client_id` | The mis-client that will submit the decision |
| `expires_at` | Approval request expiration |

The mis-backend MUST reject decisions where:

- `action_id`, `approval_request_id`, `client_id`, or `user_id` do not
  match backend state
- the approval request is expired, already decided, or associated with
  a REVOKED or PENDING client
- `details_hash` / `action_payload_hash` does not match stored action content

The binding source for approval content is the mis-backend, not the agent.
The client MUST NOT treat agent-supplied action content as the authoritative
description of what is being approved.

---

## Backend-Side Validation Requirements

Before processing any request that relies on client identity, the
mis-backend MUST:

1. Verify that `client_id` exists in `registered_clients`
2. Verify that the client belongs to the expected `user_id`
3. Verify that client status is ACTIVE
4. Verify the client session token is valid where applicable
5. Verify the client signature or proof of possession where required
6. Verify that `request_id` has not been replayed within the applicable replay window
7. Verify that the `timestamp`, nonce, or `challenge_id` is valid where applicable
8. Verify that `body_hash` matches the received payload where applicable
9. Verify that `assurance_level` satisfies policy requirements for the
   requested operation
10. Verify that the requested operation is permitted for the client's
    current state and user context

Requests from PENDING or REVOKED clients MUST be rejected.

---

## The Two Identity Layers

A WebApp/PWA client operates with two distinct identity layers that MUST
NOT be conflated:

### Client Key Possession

Proves that this browser/app instance holds the registered client private key.

- established at registration
- persistent as long as browser storage is intact
- does NOT prove that the current user is present or verified

### User Step-up Authentication

Proves that a human user is currently present and has verified their identity.

- required for high-risk operations (e.g., approving sensitive actions,
  registering new clients)
- implemented via WebAuthn/passkey or equivalent user verification mechanism
- separate from client key possession

**Client key possession alone MUST NOT be treated as sufficient
for high-risk approvals.**

The Policy Engine MAY require step-up authentication before routing
certain action types to any mis-client, independent of `assurance_level`.

See [ADR-0007](../adr/0007-webcrypto-client-key-and-webauthn-step-up.md).

---

## Transport Assumptions

All communication between mis-client and mis-backend MUST occur over
TLS (HTTPS). Unencrypted transport MUST NOT be used.

The mis-backend MUST use TLS certificates from a trusted certificate
authority. Certificate pinning MAY be implemented in native clients
for additional assurance.

### Backend-to-Client Authenticity

**MVP:** The mis-client authenticates the mis-backend through HTTPS/TLS
and trusted certificate validation. For WebApp/PWA clients, same-origin
browser security is part of the MVP trust assumption. Backend-to-client
application-level signatures are not required for the MVP.

**Target / Future:** Native clients MAY implement certificate pinning for
additional assurance. The mis-backend MAY sign approval payloads. The
mis-client MAY verify backend signatures for high-assurance deployments.
Backend-signed approval payloads may strengthen protection against
compromised intermediaries but are not MVP-required.

---

## Session and Token Handling Boundaries

Tokens issued to a mis-client for authenticated sessions:

- MUST be short-lived
- MUST be scoped to the authenticated `client_id` and `user_id`
- MUST NOT be shared between clients
- MUST be invalidated on client revocation

Tokens MUST NOT embed or carry the client private key.

Session tokens are useful for authenticated client sessions. They are
not sufficient alone for approval and denial decisions or other
high-risk client actions. These operations MUST additionally require
client key proof (signed or challenge-bound request) and, where policy
requires, user step-up authentication.

---

## Threats and Mitigations

| Threat | Mitigation |
|---|---|
| Network attacker modifies client request | HTTPS/TLS; signed or challenge-bound sensitive requests |
| Attacker steals session token | Client key proof required for high-risk actions; token scoped to `client_id` + `user_id`; short lifetime |
| Approval decision replayed | `request_id`, nonce or `challenge_id`, single-use `approval_request_id`, expiration |
| Approval decision substituted for a different action | Approval payload binding to `action_id` and `details_hash` / `action_payload_hash` |
| Revoked client attempts a decision | ACTIVE status validated before processing any request |
| Notification channel used to approve | Notification channel is inform-only; approval decisions travel exclusively through the mis-client |
| Client key possession confused with user verification | Separate user step-up layer via WebAuthn/passkey or equivalent; neither layer replaces the other |
| Backend authenticity concern for native clients | HTTPS/TLS in MVP; optional certificate pinning or backend-signed payloads in target architecture |

---

## What MUST NOT Be Stored or Transmitted

| Prohibited item | Reason |
|---|---|
| Client private key (to backend) | Breaks the trust model; backend must not hold private key material |
| Client private key (in logs) | Key exposure; must never appear in any log |
| Approval decisions (via notification channel) | Decisions travel exclusively through the trusted mis-client |
| Credentials of any kind (via notification channel) | Notifications are inform-only |
| OTPs or session keys (outside approved flow) | Must only travel through the trusted mis-client |

---

## Related Documents

- [Client Registration](client-registration.md) — enrollment lifecycle that precedes identity use
- [Session Connect](session-connect.md) — session binding that relies on registered client identity
- [Control Plane Architecture](control-plane.md) — validates client identity before routing approvals
- [Notification Channel](notification-channel.md) — out-of-band inform-only channel; not a trust surface
- [ADR-0006: WebApp/PWA MVP Client Assurance](../adr/0006-webapp-pwa-mvp-client-assurance.md)
- [ADR-0007: WebCrypto Client Key and WebAuthn Step-up](../adr/0007-webcrypto-client-key-and-webauthn-step-up.md)
