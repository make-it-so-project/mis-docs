---
read_when: working on account recovery, recovery codes, or re-establishing access for an existing user
---

# ADR-0010: Account Recovery Model

## Status

Accepted

## Date

2026-05-04

## Context

Account Recovery is the process by which an existing mis-user regains
access to their account after losing the ability to authenticate through
normal means — specifically when the user's passkey is unavailable and
no usable registered client is available to confirm a new client registration.

This situation arises when a user loses or replaces their device,
clears browser storage, or has all clients revoked. The mis-backend may
still contain previous mis_client records with status ACTIVE, but if the
user no longer controls those clients (key material lost or inaccessible),
recovery must revoke them and re-establish a trusted approval surface.

Without a recovery path, a locked-out user permanently loses access.
With an insecure recovery path, an attacker who gains access to the
user's email can take over their account.

The recovery mechanism must strike the correct balance: genuinely
self-service for the legitimate user, genuinely resistant to
email-compromise-based takeover.

Three design decisions must be made:

1. **Self-service vs. support-assisted** — who executes the recovery flow?
2. **Identity proof mechanism** — what credential proves the user is legitimate?
3. **Recovery code parameters** — quantity, format, and lifecycle

---

## Decision Drivers

- Recovery must be self-service to scale beyond a small user base
- Email alone is insufficient as the sole recovery factor — email accounts
  can be compromised, and make-it-so governs AI agent approvals
- The recovery mechanism must not be easier to exploit than a standard
  account takeover
- Recovery codes are a well-established pattern for this class of problem
- The recovery flow must end with First Client Bootstrap (per client-registration.md)
  to re-establish a trusted approval surface
- Friction must remain manageable for legitimate users
- Code management complexity must be kept low for the MVP

---

## Considered Options

### Self-service vs. support-assisted

#### Option 1: Support-assisted only

A human support agent verifies identity and manually unlocks the account.

##### Advantages
- No automated path for attackers to exploit
- High assurance of identity verification

##### Disadvantages
- Does not scale; requires operational overhead from day one
- Poor user experience; potentially days of wait time

##### Assessment
Unacceptable for the target self-service model. Support remains a last
resort for edge cases only.

---

#### Option 2: Self-service (selected)

The user completes a fully automated recovery flow without support involvement.

##### Advantages
- Scales with the user base
- Immediate resolution for legitimate users

##### Disadvantages
- Requires a secure second factor independent of the lost credentials

##### Assessment
Correct for a self-service platform. Security depends entirely on the
strength of the second factor — see identity proof options below.

---

### Identity Proof Mechanism

#### Option A: Email + time delay only

User proves email ownership via magic link. A mandatory waiting period
(e.g., 24–72 hours) limits attacker advantage.

##### Advantages
- No additional credentials for the user to manage

##### Disadvantages
- Email compromise is sufficient for an attacker to trigger recovery
- The time delay is the only defense — legitimate users also wait
- Weak for a system governing AI agent approvals

##### Assessment
Insufficient. The make-it-so threat model requires more than email alone.

---

#### Option B: Email + recovery codes (selected)

User proves email ownership via magic link AND presents a recovery code
generated at registration time.

##### Advantages
- Two independent factors: email access AND possession of the recovery code
- An attacker who compromises email alone cannot complete recovery
- Well-established pattern (comparable to 2FA backup codes)
- Immediate recovery for legitimate users; no waiting period needed

##### Disadvantages
- User must store recovery codes securely at registration
- If codes are also lost, self-service recovery is not possible

##### Assessment
Best fit for the make-it-so security model. The recovery code provides
the second factor that email alone cannot.

---

### Recovery Code Parameters

#### Option C1: Many codes (8–16)

Standard for 2FA backup codes used as a frequent daily fallback.

##### Assessment
Unnecessary for account recovery, which is a rare event. After each
recovery, a fresh set is generated. The quantity adds complexity without
meaningful benefit.

---

#### Option C2: Few codes — 2 (selected)

Two single-use codes per account.

##### Advantages
- Minimal for the user to manage
- Sufficient given that recovery is rare and codes are regenerated after use
- Tech-savvy users (MVP audience) typically store both codes in a
  password manager; quantity beyond 2 provides no additional safety

##### Disadvantages
- If both codes are stored in a single location and that location is lost,
  self-service recovery is unavailable — support path applies

##### Assessment
Correct for the MVP target audience. The low count reflects that recovery
is a one-time event after which codes are regenerated.

---

## Decision

### Recovery Model

**Self-service.** No support involvement required for standard recovery.
Support remains available as a last resort for users who have exhausted
all self-service options.

### Identity Proof

**Email magic link + recovery code (Option B).**

Both factors are required:

1. The user requests recovery by providing their email address.
2. A time-limited, single-use recovery initiation token is sent to the
   registered email address.
3. Clicking the link opens a recovery completion page where the user
   enters one of their recovery codes.
4. After successful validation of both factors, the mis-backend revokes
   all previous mis_client records for the user and the user completes
   First Client Bootstrap (new passkey + new WebCrypto client key).

No additional time delay is applied. The recovery code is the second factor.

### Recovery Code Parameters

| Parameter | Value |
|---|---|
| Count per account | 2 |
| Format | `XXXX-XXXX-XXXX` (alphanumeric, hyphen-separated) |
| Storage | Hashed in mis-backend; plain text never stored after issuance |
| Lifecycle | Single-use per code |
| Generation | At User Registration; regenerated after successful recovery |
| Regeneration via active client | Supported; requires step-up authentication |
| Display | Shown once at registration; user must acknowledge before proceeding |
| Exhaustion | If both codes are used or lost: support path only |

### Post-Recovery State

After successful recovery:

- The user's existing passkey credential is invalidated
- All previous mis_client records are revoked by the recovery flow
- A new mis_client is created as ACTIVE (via First Client Bootstrap), and is the only ACTIVE registered client after recovery
- A new set of 2 recovery codes is generated and shown once
- A security notification is sent to the verified email address

The mis_user record itself (user_id, email, display_name) is unchanged.

---

## Consequences

### Positive Consequences

- Self-service recovery available for legitimate users without support
- Email compromise alone is not sufficient for account takeover
- Recovery code model is familiar and well-understood
- Minimal code count keeps management simple for the MVP audience
- Recovery revokes all previous mis_client records, ensuring no lost or stale client remains an ACTIVE trusted approval surface after recovery — recovery is a trust-reset event for the user's client set

### Negative Consequences

- Users who lose both recovery codes cannot self-recover
- Recovery code storage is the user's responsibility
- Users who store both codes in a single location lose all redundancy

### Follow-up Implications

- Recovery code generation is part of the User Registration flow and must remain aligned with `architecture/user-registration.md`.
- The registration UX must make clear that recovery codes are shown once, must be stored safely, and cannot be retrieved after initial display.
- A "regenerate recovery codes" flow must be defined for active clients
  (requires step-up authentication)
- If recovery codes are ever pre-generated before registration completes,
  they must be generated server-side and transmitted securely
- Support-assisted recovery for users who lose all recovery codes is
  intentionally not specified here and requires a separate process definition

---

## Rationale Summary

Recovery codes provide a second factor that is independent of the lost
passkey and clients, making email compromise alone insufficient for
account takeover. Two codes are sufficient for a rare, one-time event
after which codes are regenerated. The simplicity of this model matches
the MVP target audience while maintaining the security bar appropriate
for a system governing AI agent approvals.
