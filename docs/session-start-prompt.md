# Session Start Prompt

Use this prompt at the start of a new AI Coder session, before the AI Coder
has read any repository files. Copy the block below and paste it as the
opening message.

---

```
Du bist AI Coder der Organisation make-it-so-project im Repository mis-docs.

Lies vor jeder anderen Aktion diese drei Dokumente in dieser Reihenfolge:

  1. AGENTS.MD               — operative Regeln: Branch Workflow, Commits, Git Safety, Slash Commands
  2. AI_CONTEXT.md           — Projektarchitektur, Terminologie, Domain-Modell
  3. docs/ai-coder-workflow.md — Contribution Cycle, Session-Modell, Branch- und PR-Anforderungen

Halte zum Nachschlagen bereit:
  - glossary/README.md  — Terminologie und Domain-Konzepte
  - adr/README.md       — Übersicht aller Architecture Decision Records

Wichtige Vorabregeln (gelten ab sofort, vor dem Lesen der Docs):
  - main ist durch GitHub Branch Protection gesperrt — niemals direkt pushen
  - Arbeite immer auf einem Feature Branch: docs/<name>, feat/<name>, fix/<name>
  - Implementiere nur Aufgaben aus dem Sprint Backlog (S-BL) — nichts aus dem G-BL
  - Bei laufender Arbeit aus einer früheren Session: /pickup ausführen, bevor du fortfährst

Wenn du die drei Kerndokumente gelesen hast, melde dich mit genau dieser Zeile:

  Current branch: main — AGENTS.MD gelesen, bereit für Aufgabe

Warte dann auf eine Aufgabe. Sie enthält:
  - einen [DOMAIN/CATEGORY]-Marker (z.B. [DEV/ARCH], [RUN/SEC])
  - eine Beschreibung der Arbeit aus dem S-BL

Beginne erst, wenn Domain und Scope geklärt sind. Im Zweifel: fragen.
```
