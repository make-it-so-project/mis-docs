---
read_when: working on client enrollment, first device setup, additional device registration, or client confirmation flows
---

# Use Case: Register a mis-client

## Purpose

Enable a mis-user to enroll a new device or application as a registered
trusted approval surface (mis-client) for their account.

---

## Context

Before a user can receive and act on approval requests, at least one
mis-client must be registered and ACTIVE. Client registration is the
enrollment step that precedes any use of Session Connect or approval flows.

---

## Actors

**Human User**
Initiates and confirms the registration of their new device.

**New mis-client**
The device or application being registered (e.g., mobile app, WebApp/PWA).

**Existing mis-client**
An already-registered ACTIVE trusted device belonging to the same user,
used to confirm the new registration where available.

**mis-backend**
Creates, validates, and activates the client registration entry.

---

## Preconditions

- The mis-user account exists.
- The user can authenticate to the mis-backend.
- The new client can generate or establish key material.
- Optional: the user has at least one existing ACTIVE registered client.

---

## Main Flow — Additional Client Registration

Used when the user already has at least one ACTIVE registered client.

1. User opens the new client and initiates registration.
2. User authenticates with their mis-user credentials on the new client.
3. The new client generates a cryptographic key pair. The private key
   remains on the client and MUST NOT be transmitted.
4. The new client submits a registration request to the mis-backend,
   including its public key, `client_type`, and `display_name`.
5. The mis-backend creates a PENDING registration entry for the new client.
6. The mis-backend sends a confirmation request to all existing ACTIVE
   clients: "New client '[display name]' is requesting registration.
   Approve or deny."
7. The user reviews the request on an existing trusted client and approves.
8. The mis-backend sets the new client status → ACTIVE.
9. The mis-backend sends security notifications to all active clients,
   including the newly registered one: "New client registered."

---

## Alternative Flow — First Client Bootstrap

Used when the user has no existing ACTIVE registered clients.

1. User opens the first client and initiates registration.
2. User completes strong bootstrap authentication (e.g., WebAuthn/passkey
   with user verification, or out-of-band verified link).
3. The client generates a cryptographic key pair. The private key remains
   on the client.
4. The client submits a registration request including its public key.
5. The mis-backend validates the bootstrap authentication and creates the
   client entry directly as ACTIVE (no existing client to confirm).
6. The mis-backend sends a security notification to the user via an
   out-of-band channel (e.g., registered email) confirming first enrollment.

---

## Alternative Flow — Existing Client Unavailable for Confirmation

Used when an existing client exists in the backend but is unreachable
or the user no longer has access to it.

1. Steps 1–5 of the Main Flow proceed normally.
2. The confirmation request is sent but the user cannot act on it
   (device lost, app deleted, etc.).
3. The PENDING registration expires after the defined time window.
4. The new client remains PENDING and cannot be activated.
5. The mis-backend notifies the user that the registration request expired.
6. The user must retry registration or initiate a recovery flow.

Note: Account recovery (regaining access when all clients are lost)
is intentionally out of scope. See [client-registration.md](../architecture/client-registration.md).

---

## Alternative Flow — Registration Denied

1. Steps 1–5 of the Main Flow proceed normally.
2. The user reviews the confirmation request on an existing trusted client
   and denies it.
3. The mis-backend marks the PENDING entry as denied.
4. The new client is not activated.
5. The mis-backend notifies all active clients: "New client registration
   was denied."

---

## Postconditions

On success:
- The new client has status ACTIVE in the mis-backend.
- The client's public key is registered and associated with the user.
- The client may now generate pairing codes and be selected in Session Connect.
- All active clients have received a security notification.

On failure:
- The PENDING entry is expired or denied.
- No new trusted surface has been added.
- The user is informed of the outcome.

---

## Security Notes

- The private key MUST NOT leave the new client at any point in this flow.
- A PENDING client MUST NOT receive approval requests.
- Confirmation from an existing client is the preferred confirmation model
  for additional client registration.
- First client bootstrap MUST use strong authentication because no existing
  trusted client is available to confirm.
- All registration events MUST be logged.
- Security notifications for all lifecycle events MUST be sent inform-only
  and MUST NOT contain credentials.

---

## Related Documents

- [Client Registration](../architecture/client-registration.md) — lifecycle model, states, and security requirements
- [Client Identity and Secure Communication](../architecture/client-identity-and-secure-communication.md) — key material and identity model
- [Session Connect](../architecture/session-connect.md) — depends on an ACTIVE registered client
- [ADR-0006: WebApp/PWA MVP Client Assurance](../adr/0006-webapp-pwa-mvp-client-assurance.md)
- [ADR-0007: WebCrypto Client Key and WebAuthn Step-up](../adr/0007-webcrypto-client-key-and-webauthn-step-up.md)
- [ADR-0008: Client Registration Confirmation Model](../adr/0008-client-registration-confirmation-model.md)
