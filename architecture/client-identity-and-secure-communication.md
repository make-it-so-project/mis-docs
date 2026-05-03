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

Challenge/response MUST be used:
- at registration activation (before setting status → ACTIVE)
- for step-up verification on sensitive operations (if required by policy)

---

## Signed Client Requests

Sensitive requests from a mis-client to the mis-backend SHOULD include
a client signature over the request payload or a canonical representation
of the request.

The backend verifies the signature using the stored public key.

This prevents request forgery by an attacker who intercepts a session
token but does not hold the client private key.

---

## Backend-Side Validation Requirements

Before processing any request that relies on client identity, the
mis-backend MUST:

1. Verify that `client_id` exists in `registered_clients`
2. Verify that the client belongs to the expected `user_id`
3. Verify that client status is ACTIVE
4. Verify the client signature or proof of possession where required
5. Verify that `assurance_level` satisfies policy requirements for the
   requested operation

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

---

## Session and Token Handling Boundaries

Tokens issued to a mis-client for authenticated sessions:

- MUST be short-lived
- MUST be scoped to the authenticated client and user
- MUST NOT be shared between clients
- MUST be invalidated on revocation

Tokens MUST NOT embed or carry the client private key.
Tokens MUST NOT be sufficient alone for high-risk approvals
where step-up authentication is required.

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
