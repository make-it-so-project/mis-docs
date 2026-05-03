---
read_when: adding or evaluating any user-facing notification, alert, or out-of-band messaging integration
---

# Secondary Notification Channel

## Purpose

A Secondary Notification Channel is an optional, out-of-band communication
path that informs users about events occurring in the make-it-so platform.

It exists solely to improve user awareness. It has no role in the authorization
or approval process.

---

## Definition

A **Secondary Notification Channel** is any external channel through which
make-it-so may deliver event notifications to a user.

Examples of channels:

- push notifications (mobile)
- messaging platforms (e.g., Slack, Teams, Signal)
- email
- SMS

---

## Example Events

The following events may be communicated via a secondary channel:

**Approval flow events:**
- a pending approval is awaiting the user's decision
- an agent has completed a task
- a failure or escalation has occurred
- a session or credential has expired

**Session events:**
- a session continuation has been established (existing identity reused in new chat)
- a session override has occurred (client binding replaced in active session)

**Client lifecycle security events:**
- a new client registration is pending confirmation
- a new client has been activated
- a client registration was denied
- a client registration expired without confirmation
- a client has been revoked

Security notifications for client lifecycle events MAY be sent to all
active registered clients of the user where appropriate. They are
inform-only and MUST NOT contain credentials or approval decisions.

---

## Explicit Constraints

### MUST NOT

A secondary notification channel:

- MUST NOT perform or accept approval decisions
- MUST NOT transmit approval outcomes back to the control plane
- MUST NOT carry credentials of any kind (OTPs, tokens, session keys)
- MUST NOT act as a trusted surface for any authorization action
- MUST NOT be used as a fallback approval mechanism

### MUST

A secondary notification channel:

- MUST only inform the user or redirect them to the trusted approval surface
- MUST direct the user to the mis-client for any approval action
- MUST be treated as untrusted by the control plane

---

## Notification Flow

The following flow describes how a pending approval reaches the user:

```
mis (control plane)
  │
  ▼
Secondary Notification Channel
  │   (inform only – no approval data)
  ▼
User
  │
  ▼
mis-client (trusted approval surface)
  │
  ▼
Approval Decision → mis (control plane)
```

The notification channel touches only the left side of this flow.
The approval decision travels exclusively through the mis-client.

---

## Security Boundary

The notification channel is explicitly outside the trust boundary of the
make-it-so authorization model.

The control plane MUST NOT rely on the notification channel for:

- delivery guarantees that affect authorization timing
- confirmation that the user received the notification
- any input that influences a policy decision

---

## Related Documents

- [Control Plane](control-plane.md) — approval and policy enforcement
- [Client Registration](client-registration.md) — client lifecycle events that trigger security notifications
- [ADR-0004](../adr/0004-secondary-notification-channel.md) — decision record for this concept
