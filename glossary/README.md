# Glossary

This document defines the core terminology used within the make-it-so project.

Maintaining a shared vocabulary ensures consistent communication between human contributors and AI Coders.

---

# Core Concepts

## Agent

An Agent is an operational entity within the runtime environment of the make-it-so platform.

Agents perform actions that can be:

- approved
- executed
- monitored
- audited

Agents belong to the runtime domain of the system.

---

## AI Coder

An AI Coder is an AI-based development unit responsible for implementing or modifying parts of the codebase.

Examples include:

- AI-Coder-Core
- AI-Coder-Frontend
- AI-Coder-Auth
- AI-Coder-Testing

AI Coders belong to the development domain.

---

# Backlog Model

The development process distinguishes between two backlog levels.

---

## G-BL (Global Backlog)

The Global Backlog is used for:

- ideas
- request refinement
- architectural exploration
- decomposition of complex requests

Items in the Global Backlog are not executable tasks.

---

## S-BL (Sprint Backlog)

The Sprint Backlog contains tasks that have been approved for implementation.

Only tasks inside the Sprint Backlog may trigger development activity.

The transfer from G-BL → S-BL represents formal approval for implementation.

Only the human project owner performs this transition.
