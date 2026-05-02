---
read_when: adding or evaluating notification integrations; any feature that routes events to external channels
---

# ADR-0004: Introduce secondary notification channels without approval capability

## Status

Accepted

## Date

2026-04-22

## Context

Users interacting with make-it-so may not always be actively watching the
mis-client when an approval is required. To improve responsiveness, the
platform needs a way to alert users about pending approvals and other events
through external channels such as mobile push notifications or messaging platforms.

At the same time, make-it-so has a strict authorization model: approvals MUST
only occur through the trusted mis-client. Allowing external channels to
participate in the approval flow would weaken this boundary and introduce
uncontrolled attack surfaces.

## Decision Drivers

- usability: users need timely awareness of pending actions
- security: approval authority must remain exclusively in the mis-client
- trust boundary: external channels must not become authorization surfaces
- auditability: approval paths must remain traceable and deterministic

## Considered Options

1. Allow external channels to accept approval decisions (e.g., reply "yes" in chat)
2. Allow external channels for notification only, redirect to mis-client for approval
3. No external channels; users must monitor mis-client directly

## Option Analysis

### Option 1: Approval via external channels

#### Advantages
- convenient; lower friction for the user

#### Disadvantages
- external channels are untrusted and cannot be adequately secured
- approval via chat or SMS is vulnerable to interception, spoofing, and replay
- breaks the single-trusted-surface model

#### Assessment
Unacceptable. Security cost outweighs usability gain.

---

### Option 2: Notification only, approval in mis-client (selected)

#### Advantages
- preserves the trust boundary of the mis-client
- improves user awareness without changing the authorization model
- channel compromise does not affect approval integrity

#### Disadvantages
- adds an integration point that must be kept strictly limited
- users must switch context to the mis-client to act

#### Assessment
Best balance of usability and security. Acceptable friction.

---

### Option 3: No external channels

#### Advantages
- smallest attack surface

#### Disadvantages
- users may miss time-sensitive approvals
- poor operational usability

#### Assessment
Too restrictive for practical use.

## Decision

Introduce secondary notification channels as an explicit concept in the
make-it-so architecture.

### Key rules

- Secondary channels MAY notify users about events (pending approvals, task
  completion, failures, escalations).
- Secondary channels MUST NOT transmit, accept, or influence approval decisions.
- Secondary channels MUST NOT carry credentials (OTPs, tokens, session keys).
- Approval MUST only happen in the trusted mis-client.
- The control plane MUST treat all notification channels as untrusted.

### Flow

```
mis (control plane) → notification channel → user → mis-client → approval
```

The notification channel is read-only from the control plane's perspective.

## Consequences

### Positive Consequences
- users receive timely awareness of events without polling mis-client
- security boundary of the approval model remains intact
- explicit constraint prevents future scope creep into approval flows

### Negative Consequences
- users must context-switch to mis-client to act on notifications
- notification delivery is not guaranteed; the platform cannot rely on it

### Follow-up Implications
- notification channel integrations must be reviewed against the MUST NOT
  constraints before being enabled
- the architecture document for notification channels defines the integration
  contract for any future channel implementation

## Rationale Summary

Separating notification from authorization preserves the integrity of the
mis-client as the sole trusted approval surface while meaningfully improving
user awareness. The usability cost of redirecting to mis-client is acceptable
given the security guarantee it provides.
