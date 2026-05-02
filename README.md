# make-it-so – Documentation

This repository contains the architectural, governance, and conceptual documentation for the make-it-so platform.

The documentation in this repository acts as the single source of truth for the development and conceptual design of the platform.

It defines:

- core terminology
- development governance
- architectural concepts
- architecture decisions

The goal is to maintain a shared understanding between human contributors and AI Coders.

---

# Organization Repositories

The make-it-so project is split across three repositories.

| Repository | Purpose |
|---|---|
| [mis-docs](https://github.com/make-it-so-project/mis-docs) | Architecture, decisions, governance, and terminology — the single source of truth (this repo) |
| [mis-showcase](https://github.com/make-it-so-project/mis-showcase) | Early-stage proof-of-concept prototype; demonstrates the core idea but is not a reference implementation |
| [mis-website](https://github.com/make-it-so-project/mis-website) | Public-facing marketing site — landing page, about, how-it-works; deployed on Cloudflare Workers via Lovable |

When in doubt about project scope or terminology, start here in mis-docs.

---

# Repository Structure

## Glossary

Defines the terminology used across the project.

Directory:
glossary/

---

## Governance

Defines the development governance model, including backlog structure and approval rules.

Directory:
governance/

---

## Architecture

Contains high-level architecture descriptions and conceptual system structure.

Directory:
architecture/

---

## Architecture Decision Records (ADR)

Documents important architectural decisions.

Directory:
adr/

---

# Terminology

Two domains must be clearly separated.

## Runtime Domain

Contains operational Agents that exist inside the make-it-so system.

## Development Domain

Contains AI Coders that work on the codebase.
