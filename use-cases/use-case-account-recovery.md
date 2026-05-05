---
read_when: working on account recovery flows, recovery code validation, or re-establishing access for an existing user
---

# Use Case: Account Recovery

## Purpose

Enable an existing mis-user to regain access to their account after
losing both their passkey and all registered mis-clients, using a
pre-generated recovery code as the second authentication factor.

---

## Context

A user who loses their device or clears browser storage loses both
their WebAuthn passkey credential and their registered mis-client
identity (WebCrypto key). The mis-backend may still contain previous
mis_client records with status ACTIVE, but if the user no longer controls
those clients, they cannot be used to confirm new client registration.
Without the ability to authenticate or to use any usable registered client,
the user cannot enroll a new client through the standard additional client
registration flow (which requires confirmation from an existing ACTIVE client).

Account Recovery bypasses this dependency by using a recovery code
generated at registration time as a second factor alongside email
ownership proof. Recovery ends with First Client Bootstrap, establishing
a new trusted approval surface.

---

## Actors

**Human User**
An existing mis-user who has lost access to their passkey and all
registered clients.

**mis-client WebApp**
The browser instance being enrolled as the new first trusted client.

**mis-backend**
Validates the recovery factors, executes the atomic recovery state
changes, and generates new recovery codes.

**Email system**
Delivers the recovery magic link to the user's registered email address.

---

## Preconditions

- A mis-user account exists for the provided email address.
- The user has access to the email inbox for that address.
- The user has at least one unused recovery code.
- The user's device supports passkey creation.

---

## Main Flow

1. User navigates to the recovery page on the mis-client WebApp.

2. User enters their email address and submits.

3. mis-backend issues a time-limited, single-use recovery initiation
   token and sends a magic link to the registered email address.
   The mis-backend returns the same response regardless of whether
   the email address has a registered account (prevents enumeration).

4. User opens the email and clicks the magic link.

5. Recovery completion page is presented immediately. User enters
   one of their two recovery codes.

6. mis-backend validates:
   - the initiation token (valid, not expired, not previously used)
   - the recovery code (hash match, status = active)

7. User creates a new passkey using the platform authenticator
   (WebAuthn registration ceremony). The mis-client WebApp simultaneously
   generates a new WebCrypto key pair for client identity. Both private
   keys remain on the device and MUST NOT be transmitted.

8. mis-client submits the recovery completion request to the mis-backend:
   - recovery initiation token
   - recovery code (final validation)
   - new WebAuthn credential
   - new client public key (WebCrypto)
   - `client_type = web_pwa`
   - `display_name`

9. mis-backend atomically:
   - marks the used recovery code as `used`
   - invalidates all remaining recovery codes
   - invalidates the previous passkey credential(s)
   - revokes all previous mis_client records for the user
   - creates a new `mis_client` record (`status = active`,
     `client_type = web_pwa`, `assurance_level = basic`)
   - generates 2 new recovery codes

10. New recovery codes are displayed once. User must acknowledge that
    they have stored them securely before proceeding.

11. mis-backend sends a security notification to the verified email
    confirming account recovery and new client enrollment.

12. User is authenticated and may resume using make-it-so.

---

## Alternative Flow — Initiation Token Expired

At step 6: the recovery initiation token has exceeded its TTL.

1. User is informed that the link has expired.
2. User must restart from step 1.
3. The expired token is invalidated. No state changes occur.

---

## Alternative Flow — Invalid Recovery Code

At step 6: the recovery code does not match or has already been used.

1. User is informed that the recovery code is invalid.
2. User may retry with their remaining code (if one is still available).
3. If no valid code is available, user is directed to contact support.

---

## Alternative Flow — Passkey Creation Cancelled or Failed

At step 7: the user cancels passkey creation or the platform
authenticator fails.

1. The recovery initiation token remains valid until its TTL expires.
2. The user may retry passkey creation without requesting a new link,
   provided the token is still within its validity window.
3. If the token has expired, the user must restart from step 1.

---

## Alternative Flow — No Recovery Codes Available

The user has used or lost both recovery codes.

1. Self-service recovery is not available.
2. The user must contact support for assisted recovery.

---

## Alternative Flow — Completion Request Fails

At step 9: the mis-backend rejects the recovery completion request
(validation failure, WebAuthn error, or other).

1. No state changes occur.
2. User is informed and may retry from step 1.

---

## Postconditions

On success:

- `mis_user` record is unchanged (user_id, email, display_name retained).
- Previous passkey credential is invalidated.
- All previous `mis_client` records are REVOKED by the recovery flow.
- New `mis_client` exists with `status = active`, `client_type = web_pwa`,
  `assurance_level = basic`, and is the only ACTIVE registered client after recovery.
- 2 new recovery codes have been generated and acknowledged by the user.
- Security notification has been sent to the verified email.
- User is authenticated and the new mis-client may participate in
  Session Connect and approval flows.

On failure:

- No state changes have occurred.
- The user is informed and may retry or contact support.

---

## Security Notes

- Both the recovery magic link AND a recovery code are required.
  Email access alone MUST NOT be sufficient to complete recovery.
- The recovery initiation token MUST be time-limited and single-use.
- Recovery codes MUST be stored as hashes; plain text MUST NOT be
  retained by the mis-backend after issuance.
- A used recovery code MUST be immediately marked as used and MUST
  NOT be accepted again.
- All state changes MUST be atomic — no partial recovery state.
- The new passkey private key and WebCrypto private key MUST NOT be
  transmitted to the mis-backend at any point.
- Successful recovery MUST revoke all previous mis_client records for the
  user before or as part of activating the new client. No lost or stale
  client may remain an ACTIVE trusted approval surface after recovery.
- All client revocations performed during recovery MUST be logged.
- The security notification is inform-only and MUST NOT contain
  credentials or recovery codes.
- All recovery events MUST be logged for audit purposes.

---

## Related Documents

- [Account Recovery](../architecture/account-recovery.md) — full recovery model and security requirements
- [Client Registration](../architecture/client-registration.md) — First Client Bootstrap mechanism used in recovery
- [User Registration](../architecture/user-registration.md) — initial recovery code generation at registration
- [Client Identity and Secure Communication](../architecture/client-identity-and-secure-communication.md) — key material model
- [ADR-0010: Account Recovery Model](../adr/0010-account-recovery-model.md) — decision record
