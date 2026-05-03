---
read_when: working on any user-facing UI, copy text, component design, or visual styling for mis-client, mis-website, or any make-it-so surface
---

# Design Language

## Philosophy

The make-it-so visual and verbal identity is inspired by the LCARS interface
system from *Star Trek: The Next Generation*.

The goal is **tasteful and modern, not cosplay**. LCARS provides a distinctive
aesthetic vocabulary — dark backgrounds, warm accent colors, geometric pill
shapes, uppercase condensed type — that reinforces the project's core idea:
human authority over autonomous systems. The captain is in command.

This influence is intentional and playful, but always secondary to clarity and
usability.

---

## Color System

LCARS lives on a **pure black background** ("the bridge is always dark").

All color tokens are defined as CSS custom properties in `mis-website/src/styles.css`.

### Palette Tokens

| Token | Value (oklch) | Approx. | Role |
|---|---|---|---|
| `--lcars-orange` | `oklch(0.78 0.18 65)` | `#F90` warm orange | Primary, CTA, approve actions |
| `--lcars-amber`  | `oklch(0.86 0.17 90)` | `#FFCC00` gold | Display text, hero, emphasis |
| `--lcars-blue`   | `oklch(0.82 0.13 240)` | `#9CF` medium blue | Secondary, navigation, info |
| `--lcars-salmon` | `oklch(0.78 0.16 30)` | `#F96` coral | Deny, alert, destructive |
| `--lcars-peach`  | `oklch(0.86 0.12 70)` | `#FC9` soft peach | Soft accent, supporting actions |
| `--lcars-violet` | `oklch(0.72 0.13 320)` | `#C9C` muted violet | Status labels, meta information |
| `--lcars-red`    | `oklch(0.65 0.22 25)` | deep red | Error, critical warning |
| `--lcars-panel`  | `oklch(0.16 0.02 60)` | near-black | Panel/card backgrounds |

### Semantic Color Assignments

| Semantic Role | Token |
|---|---|
| Approve / Confirm / CTA | `--lcars-orange` |
| Deny / Stop / Destructive | `--lcars-salmon` |
| Informational / Navigation | `--lcars-blue` |
| Display text / Hero | `--lcars-amber` |
| Status / Metadata | `--lcars-violet` |
| Soft accent / Secondary action | `--lcars-peach` |
| Critical error | `--lcars-red` |
| Content surface | `--lcars-panel` |

---

## Typography

### Fonts

| Role | Font | Fallback |
|---|---|---|
| Display (headings, labels, badges) | Antonio | Oswald, system-ui sans-serif |
| Body (readable text) | Inter | system-ui sans-serif |

### Display Type Rules

- Always **uppercase**
- Wide **letter-spacing** (`tracking-widest` / `0.3em–0.4em` for labels)
- Used for: headings, nav pills, status labels, section identifiers
- Black text on colored backgrounds; amber on dark panels

### Body Type Rules

- Normal case
- `text-foreground/85` to `text-foreground/90` on dark panels
- `text-sm` for body copy inside panels

---

## Shape and Layout Patterns

### The LCARS Elbow

The characteristic LCARS corner shape: a pill radius applied to a single
corner of a rectangular block.

CSS classes: `.lcars-elbow-tl`, `.lcars-elbow-bl`, `.lcars-elbow-tr`, `.lcars-elbow-br`

Used in: header top bar, CommandConsole top/bottom bars.

### Pill Shapes

Full pill (all corners rounded): `.lcars-pill` — used for nav links, labels,
action buttons.

### Panel Containers

Content sections use `rounded-2xl` or `rounded-3xl` with `bg-[var(--lcars-panel)]`.
Padding: `p-5 sm:p-7` (panels), `p-6 sm:p-12` (hero sections).

### Vertical Color Rail

The `LcarsPanel` component pattern: a narrow colored vertical bar (`w-3 sm:w-4`,
`rounded-full`) on the left of a content block, with a color-matched pill label
above the content. This is the primary content layout unit.

### Scanline Overlay

A very-low-opacity CRT scanline effect over the full viewport:
`.lcars-scanlines` — `opacity: 0.35`, `mix-blend-mode: overlay`.
Applied at the root shell level, not per-component.

### Separator Lines

Thin `h-px flex-1 bg-foreground/20` horizontal lines used to visually
separate label from status in info strips.

---

## UX Copy and Naming Conventions

make-it-so uses Star Trek TNG vocabulary for user-facing interaction copy.
These names appear in the mis-client app, agent chat prompts, and status messages.

### Core Interaction Labels

| Concept | User-facing copy |
|---|---|
| Approve an action | **"Engage"** |
| Deny an action | **"Belay that"** |
| Approve button label | **"Make it so"** |
| Awaiting user decision | **"Awaiting Captain"** |

### Session Connect Vocabulary

| Concept | User-facing copy |
|---|---|
| Initiate agent connect | **"Open Channel"** |
| Pairing code display label | **"Clearance Code"** |
| mis-client device | **"Tricorder"** |
| Successful connect | **"Channel open. Make it so."** |
| Session continuation prompt | **"Soll ich mit deinem bisherigen Channel weitermachen?"** |
| No session found | **"No channel established. Please re-connect from your Tricorder."** |

### Status and Area Labels

| Context | Label copy |
|---|---|
| Main command area | **"BRIDGE · COMMAND CONSOLE"** |
| Auth required state | **"CMD · BRIDGE · AUTH REQUIRED"** |
| Waiting for approval | **"AWAITING CAPTAIN"** |
| Timestamp display | **"STARDATE [value]"** |
| Early development notice | **"STATUS · EARLY DEVELOPMENT"** |

### Stardate

Timestamps are displayed as TNG-style stardates:

```
STARDATE = (current_year − 2323) × 1000 + (day_of_year / 365) × 1000
```

Implemented in `mis-website/src/components/site/Stardate.tsx`.

---

## Reference Implementation

The mis-website (`make-it-so.app`) is the canonical reference for the visual
system. All tokens, components, and patterns described here are implemented there.

Key files:

| File | Contains |
|---|---|
| `src/styles.css` | All LCARS CSS tokens, typography, shape classes, scanline effect |
| `src/components/site/LcarsPanel.tsx` | Vertical rail + pill label content panel |
| `src/components/site/Header.tsx` | LCARS top bar with elbow shapes and nav pills |
| `src/components/site/CommandConsole.tsx` | Animated hero terminal with LCARS chrome |
| `src/components/site/Stardate.tsx` | TNG stardate computation and display |

---

## Legal Notice Requirement

The make-it-so design language is inspired by Star Trek: The Next Generation.
The project is not affiliated with, endorsed by, or connected to CBS or Paramount.

**Every user-facing application** (mis-client, mis-website, any future surface)
MUST display the following notice at least once in a visible location — typically
in a footer, an About screen, or an Info section:

> Inspired by Star Trek: The Next Generation. Not affiliated with CBS / Paramount.

This notice MUST NOT be omitted even if the LCARS aesthetic is used subtly.

Reference implementation: `mis-website/src/components/site/Footer.tsx`

---

## What This Is Not

- This is not a strict LCARS recreation. Canon accuracy is not the goal.
- UI patterns should serve usability. When LCARS aesthetics conflict with
  clarity, clarity wins.
- These guidelines apply to user-facing surfaces (mis-client, mis-website).
  They do not apply to internal tooling, ADRs, or developer documentation.
