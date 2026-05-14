# Design System

## Purpose

This directory will contain visual identity guidance, UI style guide rules, design token
concepts, interaction state rules, accessibility requirements, platform mapping guidance,
and security-relevant approval UX requirements for make-it-so.

The design system must balance the LCARS-inspired visual identity with clear, accessible,
secure user interaction. For make-it-so, the mis-client is the trusted approval surface —
UI clarity is therefore a security property, not only a design preference.

---

## Status

**Placeholder / to be elaborated.**

This directory captures the topic and initial structure. It does not yet define:

- final color values or a color palette
- typography specifications
- component specifications or design tokens
- native implementation patterns for iOS or Android

It does not replace architecture documents about the mis-client, approval flows, the
notification channel, or client security. It does not contain implementation code.

---

## Contents

| File | Purpose |
|---|---|
| [ui-style-guide-and-approval-ux.md](ui-style-guide-and-approval-ux.md) | Placeholder for visual identity principles, interaction clarity, component states, accessibility, platform mapping, and approval UX safety requirements |

---

## Planned Future Guides

The following files are not yet created. They are listed to capture planned scope only.

| File | Purpose |
|---|---|
| visual-identity.md | LCARS-inspired visual direction, brand rules, and non-interactive decorative patterns |
| design-tokens.md | Future token model for color, spacing, typography, elevation, motion, and state |
| components.md | Future reusable component guidance for buttons, cards, panels, alerts, forms, and approval surfaces |
| approval-ux.md | Future detailed approval/deny, session reconnect, client registration, and high-risk action UX |
| accessibility.md | Future accessibility requirements and test guidance |
| platform-mapping.md | Future mapping for Web/PWA, iOS native, and Android native UI patterns |

Do not create these files until they enter an active release context.

---

## Related Documents

- [../programming-guides/language-and-toolchain-strategy.md](../programming-guides/language-and-toolchain-strategy.md) — language and toolchain candidate direction
- [../../architecture/client-registration.md](../../architecture/client-registration.md) — client lifecycle, enrollment flows, and trusted approval surface
- [../../architecture/client-identity-and-secure-communication.md](../../architecture/client-identity-and-secure-communication.md) — client key model, assurance levels, and secure communication
- [../../architecture/notification-channel.md](../../architecture/notification-channel.md) — secondary notification channel definition and constraints
- [../../architecture/action-model.md](../../architecture/action-model.md) — action request structure and lifecycle states
- [../../architecture/request-lifecycle.md](../../architecture/request-lifecycle.md) — detailed request lifecycle
- [../../governance/release-context.md](../../governance/release-context.md) — release coordination model
- [../../governance/repository-safety-and-canaries.md](../../governance/repository-safety-and-canaries.md) — protected asset classes and repository safety
- [../../AGENTS.MD](../../AGENTS.MD) — operational rules: commits, branches, git safety
- [../../AI_CONTEXT.md](../../AI_CONTEXT.md) — project architecture and domain model
- [../../adr/0003-domain-and-category-markers.md](../../adr/0003-domain-and-category-markers.md) — domain and category marker system
