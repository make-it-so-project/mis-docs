# GitHub Labels

This file defines the label scheme for the make-it-so-project/mis-docs repository.

Labels mirror the `[DOMAIN/CATEGORY]` marker system defined in ADR-0003.
Apply them to issues and PRs alongside the marker in the title.

---

## Domain Labels

| Label | Color | Description |
|---|---|---|
| `DEV` | `#0075ca` | Development Domain – relates to building the platform |
| `RUN` | `#e4e669` | Runtime Domain – relates to operational system behavior |

---

## Category Labels

| Label | Color | Description |
|---|---|---|
| `ARCH` | `#d93f0b` | Architecture |
| `FUNC` | `#0e8a16` | Functional Behavior |
| `NFR` | `#1d76db` | Non-Functional Requirements |
| `OPS` | `#5319e7` | Operations / Tooling / CI |
| `SEC` | `#b60205` | Security |
| `ADR` | `#f9d0c4` | Architecture Decision Record |

---

## Usage

Labels are applied in combination: one domain label + one category label.

Examples:

| Title marker | Labels to apply |
|---|---|
| `[DEV/ADR]` | `DEV` + `ADR` |
| `[RUN/SEC]` | `RUN` + `SEC` |
| `[DEV/OPS]` | `DEV` + `OPS` |

---

## Setup

Labels can be created via the GitHub UI (Settings → Labels) or with the GitHub CLI:

```bash
gh label create DEV --color 0075ca --description "Development Domain" --repo make-it-so-project/mis-docs
gh label create RUN --color e4e669 --description "Runtime Domain" --repo make-it-so-project/mis-docs
gh label create ARCH --color d93f0b --description "Architecture" --repo make-it-so-project/mis-docs
gh label create FUNC --color 0e8a16 --description "Functional Behavior" --repo make-it-so-project/mis-docs
gh label create NFR --color 1d76db --description "Non-Functional Requirements" --repo make-it-so-project/mis-docs
gh label create OPS --color 5319e7 --description "Operations / Tooling / CI" --repo make-it-so-project/mis-docs
gh label create SEC --color b60205 --description "Security" --repo make-it-so-project/mis-docs
gh label create ADR --color f9d0c4 --description "Architecture Decision Record" --repo make-it-so-project/mis-docs
```
