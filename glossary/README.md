
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

# Runtime Security Concepts

## MCP Server

A server implementing the Model Context Protocol (MCP) that enables AI agents or LLM systems to interact with external tools, APIs, or enterprise systems.

Within the make-it-so architecture, MCP servers are considered **untrusted by default** and must request authorization from make-it-so before accessing protected enterprise resources.

---

## Short-Lived Scoped Session

A temporary authorization context granted after human approval.

The session provides narrowly scoped permissions to an MCP server for a limited duration (TTL). The scope defines which resources and operations are allowed.

This mechanism enables just-in-time enterprise access without granting standing privileges.

---

## Scope

A defined set of permissions, resources, and allowed operations granted to an MCP server during an authorized session.

Scopes restrict what the MCP server can access within enterprise systems.

---

## Session TTL (Time To Live)

The maximum duration for which a short-lived scoped session remains valid before it automatically expires.

This ensures that temporary access permissions are automatically revoked after a defined period.

---

## mis-client

A trusted user device used to approve authorization requests within the make-it-so system.

The mis-client receives approval requests and allows the user to approve or deny them.

---

## Zone 1

A lower-risk enterprise network zone containing internal systems that may be accessed by MCP servers under controlled conditions.

More sensitive systems (for example databases, administrative infrastructure, or production control systems) typically reside in higher security zones.

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
