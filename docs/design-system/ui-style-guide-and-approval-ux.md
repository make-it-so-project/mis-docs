---
read_when: working on UI style guide, LCARS-inspired visual identity, design tokens, interaction states, approval UX, trusted approval surface, accessibility, Web/PWA UI, native client UI, or security-relevant user decisions
---

# UI Style Guide and Approval UX

## Purpose

This document captures the initial design system and approval UX principles for
make-it-so.

The UI must make approval decisions understandable, deliberate, accessible, and safe.
The visual identity may be LCARS-inspired, but interaction clarity and security always
take precedence.

---

## Status

**Placeholder / to be elaborated.**

This document captures initial principles and open questions. It is not a final UI
style guide. It does not define:

- final color values or a complete color palette
- final typography specifications
- final component specifications or design tokens
- platform-specific native implementation APIs

It does not add implementation code. It should later inform concrete design token and
component guides.

---

## Scope

This document covers:

- LCARS-inspired visual identity principles
- distinction between decorative and interactive UI
- UI element state model
- approval UX safety requirements
- deny UX
- status and risk communication
- notification vs approval UI distinction
- accessibility requirements
- Web/PWA, iOS, and Android platform mapping
- relationship to the trusted approval surface
- relationship to programming guides
- relationship to release context
- relationship to repository safety
- open questions for later elaboration

---

## Non-Goals

This document does not define:

- implementation code
- a final color palette
- final typography
- a final component library
- a design token file
- platform-specific native implementation decisions
- a dependency on Figma or any specific design tool
- replacements for client security architecture documents
- changes to notification channel rules or constraints
- a new ADR

---

## Core Principle

Make-it-so approval UI is a security boundary. The user must be able to clearly
distinguish information, decoration, status, navigation, and action controls.

Required properties:

- Decorative LCARS-inspired elements must never be confused with buttons or approval
  controls.
- Clickable and tappable controls must look and behave consistently.
- Primary, secondary, destructive, and security-critical actions must be visually
  distinct from each other.
- Approve and Deny must be clearly separated in layout, label, and visual weight.
- Disabled, pending, selected, focused, hover, pressed, expired, approved, denied,
  warning, and error states must be explicit and unambiguous.
- Color alone must not be the only signal for meaning. Icons, labels, text, and
  patterns must reinforce state and risk.
- Accessibility and platform conventions are not optional.
- Security-relevant UI must avoid ambiguity, hidden state, and any path to accidental
  approval.

---

## LCARS-Inspired Visual Identity

The make-it-so visual direction is LCARS-inspired. This means the UI may use bold
panels, rounded blocks, strong color accents, segmented layouts, and a futuristic
interface rhythm.

This visual direction must be adapted to make-it-so's product needs, not copied
mechanically from any franchise asset.

Required constraints:

- The LCARS-inspired language must not become a confusing imitation of interactive
  controls.
- Decorative panels must be visually distinguishable from actionable controls.
- Informational labels must be distinguishable from buttons.
- Functional hierarchy must be stronger than decoration. When a user is uncertain
  whether a colored shape is a button or a border, decoration has won over function.
- The style is a visual identity direction. It does not justify reducing interaction
  clarity, accessibility, or approval safety.

Do not include copyrighted imagery, franchise-specific assets, or exact replica UI
claims. "LCARS-inspired" is a direction, not a license claim.

---

## Interaction Clarity

The following categories must be visually distinguishable at a glance:

| Category | Definition |
|---|---|
| Decorative surface | Non-interactive panel, border, accent, or layout block |
| Static text | Informational label, description, or title with no action |
| Status indicator | A badge, chip, or icon that communicates current state |
| Navigation link | An element that navigates within the application |
| Clickable/tappable action | An interactive element that performs an operation |
| Primary action | The main recommended operation for a view |
| Secondary action | A less prominent or alternative operation |
| Destructive action | An operation that is irreversible or removes data |
| Security-critical approval action | An approval or denial in the trusted approval flow |
| Disabled/non-actionable element | A control that exists but cannot be activated |

**Required rule:** A user should not need to guess whether a colored shape is
clickable. If visual style makes this ambiguous, the style must be corrected.

---

## UI Element State Model

The following states must be explicitly represented for interactive elements.
Final visual values are not defined here.

| State | Meaning | Requirement |
|---|---|---|
| default | Element is available and operable | Clearly interactive if actionable |
| hover | Pointer is positioned over the element | Visible hover feedback on pointer platforms |
| focused | Element has keyboard or assistive focus | Strong focus ring or equivalent visible indicator |
| pressed / active | User is activating the element | Immediate visual feedback during activation |
| selected | Item or option is currently chosen | Distinct from hover and focus states |
| disabled | Action is currently unavailable | Clearly non-actionable; accessible explanation where needed |
| pending | Action submitted, result not yet returned | Prevent duplicate action; show progress indicator |
| approved | Approval flow completed successfully | Clear, final, non-actionable state |
| denied | Denial completed | Clear, final, distinct from error states |
| expired | Request can no longer be acted on | No Approve button visible or active |
| warning | User attention required before proceeding | Not color-only; must include label or icon |
| error | Action failed or state is invalid | Explain recovery path; do not silently swallow failures |

---

## Approval UX Safety Requirements

The approval screen is the core security-sensitive surface. It must clearly communicate:

- who or what is requesting approval
- `agent_id` or a human-readable Agent Runtime identity where available
- `session_id` or a session summary where appropriate
- action summary in plain language
- requested scope
- risk level or sensitivity where available
- target system or resource category
- expiration or TTL of the request
- consequences of approval
- consequences of denial
- whether user step-up authentication is required
- whether the request is read-only or changes external state
- whether the request is an action approval, session reconnect, client registration,
  or account recovery-related flow

Required rules:

- Approve and Deny must be visually and spatially distinct. They must not share a
  button group layout that risks accidental activation.
- High-risk approvals must require deliberate interaction, not an accidental tap or
  click.
- Critical actions must support step-up authentication where policy requires it.
- Expired requests must not show an active Approve button.
- A pending approval submission must prevent duplicate approval attempts.
- The UI must not display agent-supplied content as trusted platform content. Agent
  summaries must be clearly scoped as agent-provided.
- Approval UI must align with Approval Payload Binding requirements and the
  signed, challenge-bound client request model defined in the client identity and
  secure communication architecture.

---

## Deny UX

Deny is a first-class safe outcome, not a failure state.

Required rules:

- Deny must be easy to locate on the approval screen.
- Deny must not be visually hidden, de-emphasized, or secondary in style when the
  user is uncertain about the request.
- Denial result must be clear and final.
- For high-risk or ambiguous requests, the UI may visually encourage denial when
  context is insufficient for a confident approval.
- Optional denial reason may be supported in a future guide but is not required here.

---

## Notification vs Approval UI

Secondary notification channels are inform-only surfaces.

Required rules:

- Notification UI may inform the user that an approval request is pending.
- Notification UI must redirect the user to the mis-client for any approval action.
- Notification UI must not collect, transmit, or imply approval or denial decisions.
- Approval controls exist only on the trusted mis-client approval surface.
- Visual style must clearly distinguish a notification preview from an approval
  request. A notification that looks like an approval screen is a design defect.

See [../../architecture/notification-channel.md](../../architecture/notification-channel.md).

---

## Platform Mapping

Platform principles are defined here without final implementation decisions.

### Web/PWA

- The WebApp/PWA is the MVP candidate client form.
- Must support accessible keyboard and pointer interaction.
- Must clearly expose focus states for keyboard navigation.
- Must account for browser constraints and the `web_pwa` / `basic` assurance level
  as defined in ADR-0006.
- Browser storage constraints affect client identity persistence; UI must handle
  key loss gracefully.

### iOS Native

- When implemented, must use native iOS navigation, sheets, alerts, passkey and
  biometric patterns, and accessibility conventions.
- LCARS-inspired identity must adapt to iOS interaction patterns, not replace native
  safety affordances.
- iOS native clients carry `assurance_level = standard` or `high` as appropriate.

### Android Native

- When implemented, must use native Android navigation, dialogs, biometric and
  passkey patterns, back behavior, and accessibility conventions.
- LCARS-inspired identity must adapt to Android interaction patterns, not replace
  native safety affordances.
- Android native clients carry `assurance_level = standard` or `high` as appropriate.

**Required rule:** The same product identity may span platforms, but interaction
patterns must respect platform conventions. Overriding platform conventions to
enforce visual consistency is not acceptable when doing so reduces safety or
accessibility.

---

## Accessibility Requirements

The following requirements apply across all platforms. Exact WCAG conformance level
is an open question.

- Sufficient contrast between text and background to meet meaningful readability
  thresholds.
- Visible focus indicators for all interactive elements on keyboard-accessible
  platforms.
- Full keyboard operability for Web/PWA interactive surfaces.
- Screen reader labels for all controls, status badges, and state indicators.
- Touch target size appropriate for mobile use without requiring pixel-precise
  interaction.
- No state or risk communication that depends on color alone. Icons, labels, and
  text must reinforce meaning.
- Reduced motion option where animation or transition is used.
- Clear and actionable error messages with recovery path.
- Understandable action labels — no ambiguous verbs on approval controls.
- Support for zoom and text scaling where practical.
- Accessible confirmation and step-up authentication flows.

---

## Design Tokens and Components

Future design system work should define tokens for:

- color — including interactive, semantic, status, and risk state variants
- typography — size, weight, and line-height scale
- spacing — padding, margin, and gap scale
- radius — border radius scale
- border — stroke weight and style
- focus — focus ring style and offset
- motion — timing and easing scale
- elevation or depth if used
- risk and status state tokens
- platform adaptation tokens where needed

Future component guide should define at least:

- Button
- Link
- Approval Card
- Request Details Panel
- Status Badge
- Alert / Warning
- Error Message
- Confirmation Dialog
- Step-up Prompt
- Session Banner
- Notification Preview
- Client Registration Panel

Final token names and values are not defined here.

---

## Relationship to Trusted Approval Surface

- The mis-client is the trusted approval surface. Its UI requirements are
  security-relevant, not only design preferences.
- Secondary notification channels are not trusted approval surfaces.
- Approval UX must account for assurance level differences between `web_pwa`
  (basic) and native clients (standard, high) as defined in ADR-0006.
- The UI must make it clear when user step-up authentication is required before
  a decision can be recorded.
- The UI must not imply that holding an active client session is equivalent to
  user verification at the moment of approval.

See [../../architecture/client-identity-and-secure-communication.md](../../architecture/client-identity-and-secure-communication.md)
and [../../architecture/client-registration.md](../../architecture/client-registration.md).

---

## Relationship to Programming Guides

- UI implementation must follow the programming guides applicable to its platform.
- Web/PWA UI implementation connects to TypeScript guidance.
- Native iOS UI implementation connects to Swift guidance when introduced.
- Android native UI implementation connects to Android/Kotlin guidance if that
  platform is selected.
- Components and tokens must be documented so AI Coders can implement them
  consistently without ambiguity.
- UI work must produce testable behavior, not only screenshots.

See [../programming-guides/language-and-toolchain-strategy.md](../programming-guides/language-and-toolchain-strategy.md).

---

## Relationship to Release Context

- Major UI style guide changes should be associated with an active release context.
- Release notes should call out user-visible UI changes and approval UX changes.
- Approval UX changes are safety-relevant and require explicit review.
- UI changes that affect trusted approval behavior must be tested and reviewed before
  they enter a release build.

See [../../governance/release-context.md](../../governance/release-context.md).

---

## Relationship to Repository Safety

- Design system files, UI security requirements, approval UX documentation, and future
  token and component definitions are protected asset classes when they affect approval
  behavior or interaction clarity on the trusted approval surface.
- Changes to approval UX safety requirements must be called out explicitly in PR
  bodies.
- QAT review must verify whether UI changes weaken approval clarity, accessibility, or
  the trusted approval surface model.

See [../../governance/repository-safety-and-canaries.md](../../governance/repository-safety-and-canaries.md).

---

## MVP Guidance

Tentative MVP posture:

- Use WebApp/PWA as the initial client form per ADR-0006.
- Prioritize approval clarity over visual flourish.
- Establish minimal but explicit component states before UI implementation begins.
- Do not implement native iOS or Android clients until they enter an active release
  context with a confirmed platform decision.
- Do not require a complete design token infrastructure before MVP, but avoid
  hardcoded ambiguity in component state and interaction affordance.
- Do not let LCARS-inspired decoration obscure actions, risk signals, or state.

---

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Decorative elements look like buttons | Define explicit interactive affordance rules; separate decorative and actionable surfaces in design and implementation |
| User approves accidentally | Distinct approve/deny layout; high-risk deliberate interaction requirement; pending and disabled state enforcement |
| Color alone carries meaning | Require icons, labels, text, and patterns to reinforce all state and risk signals |
| Approval payload is misunderstood | Clear action summary, scope, target, TTL, consequences, and risk cues on every approval screen |
| Native platform conventions ignored | Platform mapping guidance; require native interaction patterns for iOS and Android clients |
| Secondary notification channel becomes approval-like | Notification UI is inform-only and must redirect to mis-client; visual style must distinguish notification from approval |
| AI Coder implements inconsistent UI | Future token and component guide; standard state model; QAT review; accessibility checks as acceptance criteria |
| UI weakens security posture | Treat approval UX requirements as a protected asset class; require explicit review for changes |

---

## Open Questions

- What is the final color palette?
- What are the contrast requirements and target WCAG conformance level?
- What design token naming convention will be used?
- What is the minimum MVP component set?
- Which UI states are mandatory before MVP release?
- How should risk level be represented visually and textually?
- How should high-risk approvals require deliberate interaction — separate tap target,
  confirmation step, or step-up?
- How should Approval Payload Binding be surfaced to the user without exposing internal
  technical complexity?
- What exact content must be shown for action approval, session reconnect, client
  registration, and account recovery flows?
- How are long or complex action detail payloads summarized clearly?
- What UI test strategy is required for AI Coder implementation of approval surfaces?
- What accessibility checks are mandatory before a release?
- When do iOS and Android native style sub-guides become active?
- How is the design system shared or coordinated across Web/PWA and native clients?
- Which UI and design decisions require formal ADRs?

---

## Related Documents

- [README.md](README.md) — design system directory index
- [../programming-guides/language-and-toolchain-strategy.md](../programming-guides/language-and-toolchain-strategy.md) — language and toolchain candidate direction
- [../../architecture/client-registration.md](../../architecture/client-registration.md) — client lifecycle and trusted approval surface enrollment
- [../../architecture/client-identity-and-secure-communication.md](../../architecture/client-identity-and-secure-communication.md) — client key model, assurance levels, and secure communication
- [../../architecture/notification-channel.md](../../architecture/notification-channel.md) — secondary notification channel constraints
- [../../architecture/session-connect.md](../../architecture/session-connect.md) — session establishment and agent-user binding
- [../../architecture/action-model.md](../../architecture/action-model.md) — action request structure and lifecycle
- [../../architecture/request-lifecycle.md](../../architecture/request-lifecycle.md) — detailed request lifecycle
- [../../architecture/control-plane.md](../../architecture/control-plane.md) — control flow and approval routing
- [../../governance/release-context.md](../../governance/release-context.md) — release coordination model
- [../../governance/repository-safety-and-canaries.md](../../governance/repository-safety-and-canaries.md) — protected asset classes and repository safety
- [../../governance/README.md](../../governance/README.md) — development governance model
- [../../AGENTS.MD](../../AGENTS.MD) — operational rules: commits, branches, git safety
- [../../AI_CONTEXT.md](../../AI_CONTEXT.md) — project architecture and domain model
- [../../adr/0003-domain-and-category-markers.md](../../adr/0003-domain-and-category-markers.md) — domain and category marker system
