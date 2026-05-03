---
read_when: working on WebApp/PWA client key management, user authentication, step-up for sensitive operations, or WebAuthn/passkey integration
---

# ADR-0007: WebCrypto Client Key and WebAuthn Step-up

## Status

Accepted

## Date

2026-05-03

## Context

A registered WebApp/PWA mis-client needs a persistent cryptographic
identity so the mis-backend can verify that a request comes from the
same browser/app instance that was originally registered.

Additionally, for sensitive operations (Client Registration, high-risk
approvals), proof of the current user's presence and intent is required.
Client key possession alone does not establish that a human user is
actively operating the client at that moment.

Two browser security capabilities are relevant:

- **Web Cryptography API** (WebCrypto): enables generation and use of
  cryptographic key pairs within the browser, with support for
  non-extractable private keys stored in browser-controlled storage.
- **Web Authentication API** (WebAuthn): enables passkey/FIDO2-style
  authentication that proves user presence and user verification through
  hardware-backed or platform authenticators.

The question is how to use these two capabilities together, and what
each proves in the context of the mis-client trust model.

---

## Decision Drivers

- persistent client identity for a WebApp/PWA without transmitting secrets
- proof of user presence for sensitive operations
- clear separation of what each mechanism proves
- no conflation of client key possession with user step-up
- compatibility with the `assurance_level` model from ADR-0006

---

## Considered Options

### Option 1: Use only web sessions and cookies

Rely on standard session tokens issued after user login.

#### Advantages
- simple implementation
- no key management required

#### Disadvantages
- no persistent client identity; any browser with a valid session token
  can act as the registered client
- session tokens can be stolen and replayed from a different context
- does not provide proof of possession of a specific registered client key
- does not provide proof of user presence beyond initial login

#### Assessment
Insufficient for the trusted approval surface model. A registered client
must prove its own persistent identity, not just a user session.

---

### Option 2: Use only WebAuthn/passkeys for all client identity needs

Use a passkey as both the client identity and user verification mechanism.

#### Advantages
- single mechanism; no separate key management
- strong user verification built in
- hardware-backed where platform supports it

#### Disadvantages
- WebAuthn is designed for user authentication, not persistent client
  identity management
- a passkey may be synced across devices (e.g., iCloud Keychain,
  Google Password Manager), which means possession does not uniquely
  identify a single device instance
- does not map cleanly to the `client_id` model where one browser
  instance is one registered client

#### Assessment
Not a complete fit. WebAuthn is valuable for step-up but does not
replace persistent per-instance client identity.

---

### Option 3: WebCrypto client key for identity plus WebAuthn for step-up (selected)

Use browser-generated WebCrypto key material for persistent client
identity, and WebAuthn/passkey for user presence proof and step-up.

#### Advantages
- clear separation of concerns: client key proves instance identity,
  WebAuthn proves user presence
- WebCrypto private key can be non-extractable, staying within the
  browser context
- WebAuthn provides strong user verification for sensitive operations
- maps cleanly to the two-layer identity model

#### Disadvantages
- two mechanisms to implement and maintain
- WebCrypto key storage is browser-managed; clearing storage destroys
  the client identity

#### Assessment
Best fit for the mis-client identity and approval trust model.

---

## Decision

Use **WebCrypto-style key material** for persistent WebApp client identity
and **WebAuthn/passkey-style authentication** for user presence verification
and step-up.

### WebCrypto Client Key

The WebApp mis-client generates an asymmetric key pair using browser
cryptographic capabilities.

Requirements:

- the key pair SHOULD be generated as non-extractable
- the private key is stored in browser-controlled storage
  associated with the WebApp origin
- the private key MUST NOT be exported to JavaScript
- the private key MUST NOT be transmitted to the mis-backend or any
  other party
- the public key is submitted to the mis-backend during Client Registration

What this proves:

> **Client key possession** — this browser/app instance holds the
> registered client private key.

What this does NOT prove:

> Whether a human user is currently present or has verified their identity.

### WebAuthn / Passkey Step-up

User presence and user verification are established via WebAuthn/passkey
or an equivalent platform authenticator mechanism.

Step-up MUST be used for:

- Client Registration (first bootstrap and additional client enrollment)
- high-risk approval operations where policy requires user verification
- any operation that requires proof that a human is currently present

What this proves:

> **User presence and/or user verification** — a human user has interacted
> with a platform authenticator at this moment.

What this does NOT prove:

> Which specific browser/app instance is making the request.

### Combined Use

The two mechanisms complement each other:

| Layer | Mechanism | Proves |
|---|---|---|
| Client identity | WebCrypto key pair | This browser instance is the registered client |
| User presence | WebAuthn/passkey | A human user is present and verified |

**Client key possession alone MUST NOT be treated as sufficient for
high-risk approvals.** The Policy Engine determines when step-up is required.

### Scope Boundary

This ADR applies to WebApp/PWA clients (`client_type = web_pwa`).

For future native clients (`ios_native`, `android_native`), equivalent
or stronger platform mechanisms (e.g., Secure Enclave, Keystore) apply
and will be defined in a separate ADR.

---

## Consequences

### Positive Consequences

- WebApp client has persistent identity independent of session tokens
- user step-up is available for high-risk operations
- the two-layer model is explicit and extensible
- native clients can layer hardware-backed equivalents on the same model

### Negative Consequences

- clearing browser storage destroys the client identity and effectively
  revokes the client
- WebCrypto non-extractable key support varies across browsers and platforms
- two mechanisms must be implemented; neither replaces the other

### Follow-up Implications

- the Policy Engine should define which action types require step-up
  regardless of `assurance_level`
- native client key handling (Secure Enclave, Keystore) should be defined
  in a future ADR when native clients are built

---

## Rationale Summary

WebCrypto client keys provide persistent WebApp instance identity without
transmitting secrets. WebAuthn/passkeys provide human presence proof for
sensitive operations. Neither replaces the other. Their combination maps
directly to the two distinct trust questions the mis-client must answer:
"Is this the registered client?" and "Is a human present and verified?"
