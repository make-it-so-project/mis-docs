---
read_when: working on account recovery, recovery codes, or re-establishing access for an existing user
---

# Account Recovery

## Purpose

Account Recovery enables an existing mis-user to regain access to their
account after losing the ability to authenticate or to control any usable
registered client. This can occur when the user's passkey is unavailable
and all registered clients are lost, revoked, inaccessible, or no longer
controlled by the user.

This document replaces the earlier placeholder and specifies the complete
recovery model for the MVP.

---

## Scope

This document defines:

- recovery triggers and preconditions
- the recovery code model
- the recovery flow
- post-recovery state
- security properties and requirements

This document does NOT define:

- User Registration — see [user-registration.md](user-registration.md)
- Additional Client Registration — see [client-registration.md](client-registration.md)
- Support-assisted recovery for users who have exhausted all self-service
  options — this is a separate operational process

---

## Recovery Triggers

Account Recovery is required when a user can no longer authenticate or
confirm a new client registration because:

1. **Passkey lost** — the user's WebAuthn credential is inaccessible
   (device lost, browser storage cleared, PWA reinstalled)
2. **No usable registered client available** — the user no longer controls
   any registered client that can confirm a new client registration. This
   includes device loss, browser storage loss, PWA reinstall, or revoked
   clients. The mis-backend may still contain records with status ACTIVE
   for previous clients, but if the user no longer controls those clients
   (key material destroyed or inaccessible), they cannot be used to confirm
   new client registration and must be revoked during recovery.

In practice these two conditions occur together (device loss). Recovery
addresses both simultaneously: it revokes all previous client records and
re-establishes the user's passkey and first client in a single atomic flow.

---

## Recovery Codes

Recovery codes are the second authentication factor used during account
recovery. They are independent of the lost passkey and clients.

### Data Model

```
recovery_codes:
  code_id      — unique identifier
  user_id      — owning mis-user
  code_hash    — hashed recovery code (plain text never stored after issuance)
  created_at
  used_at      — null if not yet used
  status       — active | used | invalidated
```

### Parameters

| Parameter | Value |
|---|---|
| Count per account | 2 |
| Format | `XXXX-XXXX-XXXX` (alphanumeric, hyphen-separated) |
| Lifecycle | Single-use per code |
| Storage | Hashed; plain text never retained after issuance |
| Display | Once only, immediately after generation |

### Generation

Recovery codes are generated at two points:

1. **During User Registration** — after passkey creation and first client
   activation, 2 recovery codes are generated and shown once. The user
   must acknowledge that they have stored the codes before proceeding.

2. **After successful Account Recovery** — 2 new codes replace all
   previous codes. Shown once; user must acknowledge.

Recovery codes MAY also be regenerated at any time from an active
mis-client, requiring step-up authentication (WebAuthn/passkey).
On regeneration, all existing codes are immediately invalidated.

### Storage Responsibility

The plain-text recovery codes are the user's responsibility to store
securely (e.g., in a password manager or printed in a secure location).

The mis-backend stores only the hashed form. The plain-text codes
cannot be retrieved after initial display.

If both codes are lost, self-service recovery is not available.
The user must contact support.

---

## Recovery Flow

```
User navigates to recovery page
       │
       ▼
User enters email address
       │
       ▼
mis-backend issues recovery initiation token (TTL-limited, single-use)
       │
       ▼
Recovery magic link sent to registered email address
       │
       ▼
User clicks link → lands directly on recovery completion page
       │
       ▼
User enters one recovery code
       │
       ▼
mis-backend validates: initiation token + recovery code hash
       │
       ├─ invalid → user informed; must restart or contact support
       │
       ▼
User creates new passkey (WebAuthn registration)
       │  (WebCrypto client key pair generated simultaneously)
       ▼
mis-client submits recovery completion request:
  - initiation token
  - recovery code (for final validation)
  - new WebAuthn credential
  - new client public key (WebCrypto)
  - client_type, display_name
       │
       ▼
mis-backend atomically:
  - marks recovery code as used
  - invalidates remaining recovery code(s)
  - invalidates previous passkey credential(s)
  - revokes all previous mis_client records for the user
  - creates new mis_client record (status = active)
  - generates 2 new recovery codes
       │
       ▼
New recovery codes displayed once — user must acknowledge
       │
       ▼
Security notification sent to verified email
       │
       ▼
User is authenticated; account access restored
```

### Step-by-Step Description

**Step 1 — Recovery initiation**

The user navigates to the recovery page and enters their email address.

To prevent user enumeration, the mis-backend returns the same response
regardless of whether the email address has a registered account.

**Step 2 — Magic link delivery**

The mis-backend generates a time-limited, single-use recovery initiation
token and sends a magic link to the registered email address.

**Step 3 — Recovery code entry**

The user clicks the magic link. The recovery completion page is presented
immediately. The user enters one of their two recovery codes.

The mis-backend validates:
- the initiation token (valid, not expired, not previously used)
- the recovery code (hash match, status = active)

If either validation fails, the user is informed and may restart or
contact support.

**Step 4 — Passkey and client key creation**

The user creates a new passkey using the platform authenticator
(WebAuthn registration ceremony). The mis-client WebApp simultaneously
generates a new WebCrypto key pair for client identity.

Both private keys remain on the device and MUST NOT be transmitted.

**Step 5 — Recovery completion**

The mis-client submits the recovery completion request. The mis-backend
validates all components and atomically:

- marks the used recovery code as `used`
- sets remaining recovery code(s) to `invalidated`
- invalidates the previous passkey credential
- revokes all previous mis_client records for the user
- creates a new `mis_client` record (`status = active`,
  `client_type = web_pwa`, `assurance_level = basic`)
- generates 2 new recovery codes

**Step 6 — New recovery codes**

The 2 new recovery codes are displayed once. The user must acknowledge
that they have stored them before proceeding. The codes cannot be
retrieved again.

**Step 7 — Security notification**

The mis-backend sends a security notification to the verified email:

> "Your make-it-so account has been recovered. A new client has been
> enrolled and new recovery codes have been generated."

The notification is inform-only and MUST NOT contain credentials.
See [notification-channel.md](notification-channel.md).

**Step 8 — Session established**

The user is authenticated via the new passkey and may resume using
make-it-so.

---

## Post-Recovery State

After successful recovery:

| Element | State |
|---|---|
| mis_user record | Unchanged (user_id, email, display_name retained) |
| Previous passkey credential | Invalidated |
| Previous mis_client records | REVOKED by the recovery flow |
| New mis_client | ACTIVE (`client_type = web_pwa`, `assurance_level = basic`); the only ACTIVE registered client after recovery |
| Used recovery code | Status = used |
| Remaining old recovery codes | Status = invalidated |
| New recovery codes | 2 new codes generated; shown once |
| Pending action requests | Expired (no active client was available to route them) |

---

## Relationship to First Client Bootstrap

Account Recovery uses the First Client Bootstrap mechanism (Context 2)
defined in [client-registration.md](client-registration.md).

The bootstrap mechanism is the same as in initial User Registration:
new passkey creation + new WebCrypto client key in one flow. The
surrounding context differs — recovery adds the recovery code validation
step before bootstrap begins.

---

## Security Properties

| Property | Requirement |
|---|---|
| Second factor | Recovery code required in addition to email magic link |
| Email alone | MUST NOT be sufficient to complete recovery |
| Recovery initiation token | MUST be time-limited and single-use |
| Recovery code | MUST be validated as hash match; plain text MUST NOT be stored |
| Used codes | MUST be marked as used immediately and MUST NOT be reusable |
| Code exhaustion | If both codes lost or used: no self-service path; support only |
| Previous credentials | Passkey MUST be invalidated after recovery |
| Previous clients | All previous mis_client records MUST be revoked by the recovery flow; no lost or stale client may remain an ACTIVE trusted approval surface after recovery |
| Trust reset | Recovery is a trust-reset event: after recovery, only the newly bootstrapped client is ACTIVE |
| Atomic operation | All state changes MUST succeed or none; no partial recovery state |
| Security notification | MUST be sent to verified email after successful recovery |
| Audit | All recovery events MUST be logged |
| New codes | MUST be generated and shown once after every successful recovery |

---

## Related Documents

- [User Registration](user-registration.md) — includes initial recovery code generation
- [Client Registration](client-registration.md) — First Client Bootstrap mechanism used in recovery
- [Client Identity and Secure Communication](client-identity-and-secure-communication.md) — key model re-established during recovery
- [Notification Channel](notification-channel.md) — security notification delivery
- [Use Case: Account Recovery](../use-cases/use-case-account-recovery.md) — end-to-end recovery flow
- [ADR-0009: User Registration Model](../adr/0009-user-registration-model.md) — notes recovery as follow-up implication
- [ADR-0010: Account Recovery Model](../adr/0010-account-recovery-model.md) — decision record for self-service model and recovery codes
