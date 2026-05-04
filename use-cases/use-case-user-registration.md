---
read_when: working on user onboarding, registration flows, or first client bootstrap during account creation
---

# Use Case: Register a mis-user

## Purpose

Enable a private individual to create a make-it-so account and enroll
their first trusted approval surface (mis-client) as a single atomic
self-service operation.

---

## Context

Before a user can receive and act on approval requests, two things must
exist: a mis-user account and at least one ACTIVE registered mis-client.

User Registration and First Client Bootstrap are combined into one flow.
Separating them would create a meaningless intermediate state: an account
with no usable approval surface.

The registration flow uses a magic link sent to the user's email address
to verify email ownership, then leads directly into passkey creation —
one continuous user journey.

---

## Actors

**Human User**
A private individual registering for make-it-so for the first time.

**mis-client WebApp**
The browser instance the user is registering from. Becomes the first
registered trusted approval surface on successful completion.

**mis-backend**
Validates the registration flow, creates the mis_user and first mis_client
records atomically, generates recovery codes, and sends the security
notification.

**Email system**
Delivers the magic link to the user's email address.

---

## Preconditions

- No existing mis-user account for the provided email address.
- The user has access to the email inbox for the provided address.
- The user's device supports passkey creation (platform authenticator
  or hardware security key).

---

## Main Flow

1. User navigates to the registration page on the mis-client WebApp.

2. User enters their email address and submits the registration form.

3. mis-backend generates a time-limited, single-use registration token
   and sends a magic link to the provided email address.
   The mis-backend returns the same response regardless of whether the
   address is already registered (prevents user enumeration).

4. User opens the email and clicks the magic link.

5. mis-client WebApp validates the registration token with the mis-backend.
   The passkey creation screen is presented immediately — no intermediate
   confirmation page.

6. User creates a passkey using the platform authenticator
   (WebAuthn registration ceremony).
   The mis-client WebApp simultaneously generates a WebCrypto key pair
   for persistent client identity. Both private keys remain on the device.

7. mis-client submits the registration completion request to the
   mis-backend, including:
   - the registration token (email ownership proof)
   - the WebAuthn credential (passkey public key and credential ID)
   - the client public key (WebCrypto)
   - `client_type = web_pwa`
   - `display_name` (auto-detected or user-provided)

8. mis-backend validates the token and the WebAuthn credential, then
   atomically creates:
   - `mis_user` record with `status = active`
   - `mis_client` record with `status = active`,
     `client_type = web_pwa`, `assurance_level = basic`

9. mis-backend generates 2 recovery codes for the new mis-user account
   and stores only their hashes.

10. Recovery codes are displayed once. The user must acknowledge that
    they have stored them securely before proceeding.

11. mis-backend sends a security notification to the verified email
    address confirming account creation and first client enrollment.

12. User is authenticated and may begin using make-it-so.

---

## Alternative Flow — Registration Token Expired

At step 5: the registration token has exceeded its TTL.

1. mis-client WebApp informs the user that the link has expired.
2. User is directed back to step 1 to request a new magic link.
3. The expired token is invalidated. No records are created.

---

## Alternative Flow — Passkey Creation Cancelled or Failed

At step 6: the user cancels the passkey creation or the platform
authenticator fails.

1. The registration token remains valid until its TTL expires.
2. The user may retry passkey creation without requesting a new link,
   provided the token is still within its validity window.
3. If the token has since expired, the user must restart from step 1.

---

## Alternative Flow — Email Already Registered

At step 3: the email address is already associated with a mis-user account.

1. mis-backend returns the same generic response as for a new address.
2. No magic link is sent.
3. The existing account is not modified or affected.

---

## Alternative Flow — Validation Failure at Completion

At step 8: the mis-backend rejects the registration completion request
(invalid WebAuthn credential, malformed client key, or other error).

1. No records are created.
2. The user is informed that registration could not be completed.
3. The user may retry from step 1.

---

## Postconditions

On success:

- `mis_user` exists with `status = active`.
- First `mis_client` exists with `status = active`, `client_type = web_pwa`,
  `assurance_level = basic`.
- The user's email address is verified and registered as the notification
  address.
- 2 recovery codes have been generated and acknowledged by the user.
- Recovery codes are stored as hashes; plain-text values are not retained
  and cannot be retrieved later.
- A security notification has been sent to the verified email.
- The user is authenticated and the mis-client may participate in
  Session Connect and approval flows.

On failure:

- No `mis_user` or `mis_client` records exist for this registration attempt.
- The user is informed of the outcome and may retry.

---

## Security Notes

- The registration token MUST be time-limited and single-use.
- The mis_user record MUST NOT be created until the full flow completes.
- The passkey private key and the WebCrypto private key MUST NOT be
  transmitted to the mis-backend at any point.
- Email verification (magic link click) and passkey creation are combined
  into one flow; both are required for successful registration.
- Recovery codes MUST be generated after successful registration, displayed
  once, and acknowledged by the user.
- Recovery codes MUST be stored as hashes; plain text MUST NOT be retained
  by the mis-backend after issuance.
- The security notification after registration is inform-only and MUST
  NOT contain credentials, tokens, recovery codes, or approval data.
- All registration events MUST be logged for audit purposes.

---

## Related Documents

- [User Registration](../architecture/user-registration.md) — full registration model and security requirements
- [Client Registration](../architecture/client-registration.md) — lifecycle model for the first and subsequent clients
- [Client Identity and Secure Communication](../architecture/client-identity-and-secure-communication.md) — key material model
- [Session Connect](../architecture/session-connect.md) — depends on an ACTIVE registered client
- [ADR-0007: WebCrypto Client Key and WebAuthn Step-up](../adr/0007-webcrypto-client-key-and-webauthn-step-up.md)
- [ADR-0008: Client Registration Confirmation Model](../adr/0008-client-registration-confirmation-model.md)
- [ADR-0009: User Registration Model](../adr/0009-user-registration-model.md)
