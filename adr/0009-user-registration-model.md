---
read_when: working on user registration, onboarding flows, authentication design, or first client bootstrap
---

# ADR-0009: User Registration Model

## Status

Accepted

## Date

2026-05-04

## Context

User Registration was intentionally left as a placeholder in the initial
architecture. All earlier documents assumed a mis-user already existed
before Client Registration or Session Connect was performed.

As the project reaches the stage where User Registration must be defined,
three interconnected decisions must be made:

1. **Authentication mechanism** — what credential does a user create and
   use to authenticate to the mis-backend?
2. **Registration flow** — how does email verification relate to the
   passkey bootstrap, and when does the mis_user record come into existence?
3. **Onboarding model** — who initiates registration, and under what conditions?

These decisions interact with the existing architecture: ADR-0007 mandates
WebAuthn/passkey for step-up authentication, and ADR-0008 mandates strong
bootstrap authentication for first client enrollment. The User Registration
model must align with both.

Additionally, the project owner has decided that User Registration and
First Client Bootstrap are performed as one combined atomic flow. A user
without a registered client has no usable approval surface; requiring a
separate registration step would create a meaningless intermediate state.

---

## Decision Drivers

- make-it-so governs AI agent approvals — every user interaction is
  security-relevant; a password-only path is insufficient
- ADR-0007 already mandates WebAuthn/passkey for step-up; using passkey
  as the login credential as well yields one mechanism, not two
- TOTP (shared secret) is phishable; passkeys are phishing-resistant
  by design and bound to the origin
- Email must be verified before the mis_user record exists, so that the
  notification address is trustworthy from the moment of account creation
- Avoiding partial user states (no pending_verification in mis_user)
  reduces implementation complexity and eliminates zombie records
- MVP targets private individuals (self-service); enterprise onboarding
  is a future deployment concern
- The target audience for the MVP (tech-forward early adopters) has
  passkey-capable devices in practice

---

## Considered Options

### Authentication Mechanism

#### Option A1: Email + Password + TOTP

Standard two-factor approach: password as primary credential, TOTP
(authenticator app) as second factor.

##### Advantages
- Familiar to most users

##### Disadvantages
- Both password and TOTP are phishable via real-time phishing attacks
- ADR-0007 still requires WebAuthn step-up for bootstrap and high-risk
  approvals → user would need to set up two separate auth mechanisms
- Passwords introduce credential storage and breach risks

##### Assessment
Insufficient for a security-critical approval surface. Phishing risk
is unacceptable. Requires two auth mechanisms where one suffices.

---

#### Option A2: Passkey only (selected)

Passkey (WebAuthn credential) is the sole authentication mechanism.
No password is issued or accepted.

##### Advantages
- Phishing-resistant by design: passkeys are origin-bound and cannot
  be replayed on a different domain
- Hardware-backed on modern devices (Face ID, Touch ID, Windows Hello)
- ADR-0007 already mandates WebAuthn for step-up; passkey login uses
  the same mechanism — one credential for both purposes
- No password to store, manage, or breach
- Simpler implementation: single auth path

##### Disadvantages
- Requires a passkey-capable device
- Less familiar to users with no prior passkey experience

##### Assessment
Best fit for make-it-so. Phishing resistance is critical for an
approval surface governing AI agent actions. One mechanism for both
login and step-up is simpler and more consistent.

---

#### Option A3: Both (password+TOTP and passkey)

Support both paths. User chooses their preferred mechanism.

##### Advantages
- Maximum compatibility

##### Disadvantages
- Security floor drops to the weakest path (password+TOTP is phishable)
- Two auth paths means two attack surfaces
- An attacker targets the weaker path; passkey offers no defense if
  the account can also be accessed via password+TOTP

##### Assessment
Unacceptable. Offering a phishable fallback negates the security
properties of the passkey path.

---

### Registration Flow

#### Option B1: Passkey first, email verification async

User creates passkey immediately; email verification is a subsequent
optional or time-limited step.

##### Advantages
- Fastest time-to-active

##### Disadvantages
- mis_user exists with an unverified email address
- Security notifications after first client enrollment (required by
  ADR-0008) go to an unverified address
- Introduces partial state: active account, unverified notification channel
- Requires handling of accounts that never verify (suspension logic)

##### Assessment
Unacceptable. Introduces exactly the partial state complexity this
architecture aims to avoid.

---

#### Option B2: Email verification first, passkey second

User verifies email via magic link; passkey creation happens on a
separate subsequent page.

##### Advantages
- Email verified before any account exists
- Passkey created in a verified, trusted context
- No partial state in mis_user

##### Disadvantages
- User must switch contexts (email client → browser) then navigate
  to a second screen — two perceptible steps

##### Assessment
Correct security model. Minor UX friction.

---

#### Option B3: Email link leads directly into passkey creation (selected)

User clicks the magic link in their email and lands directly on the
passkey creation screen — no intermediate confirmation page. The flow
is one continuous user journey even though two interactions occur.

##### Advantages
- All advantages of Option B2
- The transition from email link to passkey creation is seamless;
  the user experiences it as a single process
- Aligns naturally with the combined User Registration + First Client
  Bootstrap flow: the email link is the bridge between both

##### Disadvantages
- Requires careful UX design of the landing page to make the passkey
  creation step clear and immediate

##### Assessment
Best balance of security and user experience. Architecturally
equivalent to Option B2 with superior UX framing.

---

### Onboarding Model

#### Option C1: Admin-provisioned only

A platform administrator creates user accounts manually.

##### Advantages
- Full control over who can register

##### Disadvantages
- Not suitable for MVP targeting private individuals
- Requires operational infrastructure from day one

##### Assessment
Not appropriate for the MVP phase.

---

#### Option C2: Self-service (selected for MVP)

Users register themselves without administrator involvement.

##### Advantages
- Enables private individuals to onboard independently
- Low operational overhead for the MVP

##### Disadvantages
- Requires bot/spam protection (rate limiting, token-based flow)

##### Assessment
Correct for MVP. The token-based email flow (magic link) provides
inherent spam protection without CAPTCHA.

---

#### Option C3: Self-service with domain restriction (selected for enterprise)

Self-service registration is enabled but restricted to email addresses
matching a configured domain allowlist.

##### Advantages
- Enables enterprise self-hosted deployments without admin provisioning
- Trust is delegated to the company's email domain security
- No per-user admin action required

##### Disadvantages
- Requires configuration at deployment time

##### Assessment
Correct for enterprise self-hosted instances. The mis-backend exposes
a configuration parameter (e.g., `allowed_email_domains`) that restricts
self-service registration to matching domains. This is a deployment-time
setting, not a core platform feature, and is deferred to post-MVP.

---

## Decision

### Authentication

Passkey-only. No password is issued or accepted. The passkey created
during User Registration serves as both the login credential and the
WebAuthn step-up mechanism required by ADR-0007 and ADR-0008.

### Registration Flow

Option B3: The magic link sent to the user's email address opens directly
onto the passkey creation screen. Email verification and passkey creation
form one continuous user journey.

The mis_user record MUST NOT be created until the full registration flow
is successfully completed. A separate short-lived registration token
(with TTL) tracks the in-progress flow. No partial mis_user state exists.

### Combined Flow

User Registration and First Client Bootstrap are one atomic operation.
On successful completion:

- the mis_user record is created with status `active`
- the first mis_client record is created with status `active`
- exactly 2 recovery codes are generated
- only recovery code hashes are stored by the mis-backend
- plain-text recovery codes are displayed once and are not retained
- the user must acknowledge storing the recovery codes before proceeding
- a security notification is sent to the verified email address

### Onboarding Model

MVP: self-service for private individuals.

Future enterprise: self-hosted backend instances MAY restrict self-service
registration to specific email domains via a configuration parameter.
This is a deployment-time setting deferred to post-MVP.

---

## Consequences

### Positive Consequences

- Phishing-resistant authentication from the first interaction
- No partial user states; mis_user always represents a fully registered,
  active account
- Email is verified before it is used as a notification address
- One authentication mechanism (passkey) covers both login and step-up
- User Registration and First Client Bootstrap are aligned: account
  creation always produces a usable approval surface

### Negative Consequences

- Users without passkey-capable devices cannot register
- Passkey UX varies across platforms and browsers; edge cases exist
- No password fallback means no recovery via "forgot password" flow;
  recovery is addressed in ADR-0010

### Follow-up Implications

- Account recovery (regaining access when all clients are lost, or
  passkey is lost) is defined in ADR-0010
- Enterprise domain restriction configuration must be designed when
  self-hosted enterprise deployments are scoped
- The Policy Engine may later distinguish login assurance level from
  step-up assurance level for specific action types

---

## Rationale Summary

Passkey-only authentication eliminates the phishing risk present in
password+TOTP flows, aligns with WebAuthn mandates already established
in ADR-0007 and ADR-0008, and provides one mechanism where two would
otherwise be required. The combined registration and first client
bootstrap flow ensures that account creation always produces a usable
approval surface, with no intermediate state that requires separate
handling. Email verification is the entry gate to passkey creation,
ensuring the notification address is trustworthy before the account exists.
