---
read_when: working on account recovery, loss of all clients, passkey loss, or re-establishing access for an existing user
---

# Account Recovery

## Status

**Placeholder — not yet specified.**

This document defines the scope, constraints, and open questions for
Account Recovery. The concrete flow and authentication mechanism will
be defined in a future dedicated ADR.

---

## Purpose

Account Recovery enables an existing mis-user to regain access to their
account after losing the ability to authenticate through normal means.

This is distinct from:

- **User Registration** — creating a new account (see [user-registration.md](user-registration.md))
- **First Client Bootstrap (Initial)** — enrolling the first client during new account creation
- **Additional Client Registration** — adding a new client when at least one active client exists (see [client-registration.md](client-registration.md))

Account Recovery applies specifically to the case where a registered
user can no longer authenticate because all trusted paths are unavailable.

---

## Recovery Triggers

Recovery may be required in three situations:

**1. All registered clients lost**
The user's registered devices have been lost, stolen, wiped, or revoked,
leaving no ACTIVE mis-client available. Without an active client, the
user cannot receive approval requests or confirm new client registrations.

**2. Passkey lost**
The user's passkey has been destroyed (e.g., browser storage cleared,
device replaced, PWA reinstalled). Without the passkey, the user cannot
authenticate to the mis-backend, even if a client record technically exists.

**3. Both (most common)**
In practice, loss of a device typically means loss of both the registered
client identity (WebCrypto key) and the passkey. These two triggers
usually occur together.

---

## Why This Is a Separate, High-Security Flow

Recovery is not a simplified re-registration. It must satisfy a stronger
set of requirements than initial User Registration, because:

- The user cannot prove identity through their registered passkey
  (it is lost) or through an existing trusted client (there are none)
- An attacker who knows a user's email address could attempt to trigger
  recovery and gain unauthorized access
- Recovery therefore represents the most dangerous identity transition
  in the system — it must not become an account takeover vector

The recovery mechanism must establish proof of identity through a channel
that is independent of the lost credentials, without creating a path
that attackers can exploit more easily than the protected account itself.

---

## Hard Constraints (Must be satisfied by any future design)

Any recovery flow defined in the future ADR MUST satisfy the following:

| Constraint | Rationale |
|---|---|
| Must not rely on the lost passkey or lost clients | They are unavailable by definition |
| Must prove identity independently | Email alone is insufficient — email accounts can be compromised |
| Must not be easier to exploit than the account itself | Recovery must not lower the effective security of the system |
| Must be logged in full | All recovery events are high-security audit events |
| Must notify the user | Via all available channels at the time of recovery |
| Must result in First Client Bootstrap | Recovery ends with the user re-enrolling a new first client through the standard bootstrap mechanism |
| Must not silently re-activate lost clients | Recovered access starts fresh; old client records remain REVOKED |

---

## What Recovery Is Not

- Not a "forgot password" flow — there is no password in this system
- Not a way to bypass the First Client Bootstrap security model
- Not a support shortcut — any recovery path involving human support
  must itself be designed to prevent social engineering attacks
- Not in scope for the current MVP phase

---

## Open Questions (To be resolved in the future ADR)

The following design decisions are intentionally deferred:

1. **Identity proof mechanism** — What out-of-band proof of identity is
   acceptable? Options include: verified email + time-delay + additional
   verification step; government ID verification; support-assisted flow;
   pre-registered recovery codes generated at registration time.

2. **Cool-down period** — Should there be a mandatory waiting period
   after a recovery request before the new client is activated?
   This limits the window for attackers but increases user friction.

3. **Notification strategy** — If all clients are gone and email is the
   only remaining channel, what security notifications can be sent and
   when? What if the email account is also compromised?

4. **Self-service vs. support-assisted** — Can recovery be fully
   self-service, or does it require human support involvement for
   additional verification?

5. **Impact on pending requests** — What happens to action requests
   that are pending approval when a user enters recovery? Are they
   cancelled, held, or expired?

6. **Recovery codes** — Should users be offered pre-generated recovery
   codes at registration time (similar to 2FA backup codes) as a
   recovery path? These must be stored securely by the user.

---

## Current Behavior (Until Recovery is Defined)

Until Account Recovery is specified and implemented:

- A user who loses all registered clients and their passkey cannot
  authenticate or receive approval requests.
- Agents receive rejection responses for action requests that cannot
  be routed to an active client.
- The user must contact support; no automated recovery path exists.

This is the correct conservative default — providing no recovery path
is safer than providing an insecure one.

---

## Related Documents

- [User Registration](user-registration.md) — combined account creation and first client bootstrap
- [Client Registration](client-registration.md) — First Client Bootstrap mechanism used at the end of recovery
- [Client Identity and Secure Communication](client-identity-and-secure-communication.md) — key model that recovery must re-establish
- [ADR-0008: Client Registration Confirmation Model](../adr/0008-client-registration-confirmation-model.md) — defines the recovery boundary for client registration
- [ADR-0009: User Registration Model](../adr/0009-user-registration-model.md) — notes recovery as a follow-up implication
