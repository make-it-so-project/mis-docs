---
read_when: working on backend hosting, runtime operations, deployment targets, backup, restore, failover, monitoring, cost model, Cloudflare deployment, VPS fallback, or production readiness
---

# Backend Hosting and Operations Strategy

## Purpose

This document captures the initial hosting and operations strategy for the make-it-so
backend.

It defines the early decision space for where the backend may run, how MVP costs should be
controlled, and which operational concerns must be designed before production use.

---

## Status

**Placeholder / to be elaborated.**

This document captures initial principles and open questions. It is not yet an Architecture
Decision Record. It does not define the final deployment architecture. It does not contain
implementation code or deployment configuration. It does not disclose private infrastructure
details. It should later inform one or more ADRs when hosting decisions become binding.

---

## Scope

This document covers:

- MVP backend hosting candidates
- Cloudflare-first MVP posture
- project-owned VPS as possible backup, emergency, or diagnostic node
- cost model
- deployment model questions
- state and persistence
- backup, restore, and sync
- resilience and failover
- secrets management
- monitoring, logging, and audit
- security boundaries
- relationship to programming guides and language choices
- relationship to release context
- relationship to repository safety
- open questions for later elaboration

---

## Non-Goals

This document does not define:

- implementation code
- deployment automation
- provider-specific configuration files or manifests
- final Cloudflare architecture
- final VPS architecture
- final database or storage product choice
- final queue or messaging choice
- incident response runbook
- private infrastructure details of any kind
- a production readiness claim
- a new ADR

---

## Core Principle

MVP hosting should be cost-conscious, simple, secure enough for early usage, and reversible.
Operational simplicity is a feature, but not a substitute for backup, restore, secrets
management, and auditability.

Required properties:

- Prefer free or low-cost hosting during MVP where feasible.
- Cloudflare-first is the current candidate posture, not a final binding decision.
- Growing beyond free or low-cost tiers is acceptable if it reflects real adoption; that is
  a scaling and growth question, not an MVP blocker.
- A project-owned VPS may be useful as backup, emergency, diagnostic, or secondary
  operations node; the exact role is not yet decided.
- Public documentation must not disclose private hostnames, IP addresses, credentials,
  backup paths, secret names, or operational runbooks.
- Security and restore capability must be designed before handling sensitive production data.

---

## Candidate MVP Hosting Posture

### Cloudflare-First MVP Posture

Cloudflare is the primary candidate for early backend hosting where technically feasible.

Potential benefits include:

- low or no cost at early usage levels
- managed edge platform with global availability
- simple public endpoint hosting without self-managed infrastructure
- lower operational overhead compared to self-hosted services

Suitability depends on backend runtime requirements. These requirements must be clarified
before a final decision:

- long-running processes
- durable state close to the runtime
- database or storage access patterns
- queue or messaging needs
- scheduled jobs or cron-style workers
- WebSocket or persistent connection requirements
- deployment model and environment separation

**This posture is not a final provider decision.** If Cloudflare constraints do not fit the
required backend architecture, alternative hosting is not excluded.

### Project-Owned VPS as Secondary Operations Option

A project-owned VPS may serve one or more of the following roles depending on operational
decisions made later:

- backup target
- emergency or diagnostic node
- warm or cold standby
- secondary runtime option
- sync target for offline or resilience scenarios
- operational tooling host

The exact role of the VPS is not decided yet. It may range from cold backup storage to a
warm standby or an active secondary runtime. See Standby Levels below.

**Private details such as hostnames, IP addresses, SSH configuration, ports, credentials,
and access paths are not documented here.**

### Cost Model

- MVP should avoid unnecessary fixed infrastructure cost.
- Free or low-cost tiers are preferred while usage is low.
- If usage creates meaningful hosting costs, this is a scaling and growth issue to manage,
  not an MVP blocker.
- Cost visibility and basic usage monitoring are required before broader testing begins so
  unexpected cost growth is detectable early.

---

## Deployment Model Questions

The following questions must be answered before a deployment architecture is decided. They
are listed here to define the scope of the decision, not to answer it.

- Is the backend serverless, containerized, or a hybrid?
- Does the backend require long-running workers?
- Does the backend require queues or message buses?
- Does the backend require scheduled or cron-style jobs?
- Does the backend require WebSocket or persistent connections for agent or client
  communication?
- Does the backend require durable state close to the compute runtime?
- Can the Cloudflare product model support the required MVP architecture cleanly?
- Which components, if any, are better placed on a VPS or container host?
- How are environments represented: dev, test, staging, production, or a smaller MVP
  variant?
- How are deployments tied to the active release context?

---

## State and Persistence

Backend hosting cannot be finalized without a clear state strategy. Make-it-so requires
persistent state that includes at a minimum:

- user records with verified email and `user_id`
- registered client records with lifecycle status
- active and historical sessions
- pending and historical approval requests
- audit log entries for security- and approval-relevant events
- agent registrations and status
- recovery metadata
- configuration and policy data

Required design principles:

- Persistent state must not rely on a single untested storage location.
- Audit logs require integrity and retention guarantees beyond operational logs.
- Recovery data and secrets require special handling separate from general application state.
- State must be backed up, restorable, and subject to retention and security policy.

The concrete database or storage technology is deferred.

---

## Backup, Restore, and Sync

- Backup is not useful unless restore is tested. A backup that has never been restored
  should not be treated as a proven safety net.
- If Cloudflare is primary, a secondary backup or sync path to a project-owned VPS or
  another secure target may be considered. The sync design must avoid leaking secrets or
  creating inconsistent state.
- Backups must be encrypted where appropriate.
- Restore tests should become part of operational readiness checks before production use.
- Public documentation must not disclose backup paths, credentials, bucket names, provider
  tokens, or internal restore commands.

### Standby Levels (Conceptual)

| Level | Definition |
|---|---|
| Cold standby | Backup exists; restore requires manual action and time |
| Warm standby | State is periodically synced; activation requires deliberate manual steps |
| Hot standby | Active failover-ready node; automatic or near-automatic activation |

The MVP does not yet decide which standby level is required or feasible.

---

## Resilience and Failover

- Define failure modes that matter for MVP before claiming resilience.
- Relevant failure scenarios include: Cloudflare edge outage, storage or database outage,
  deployment failure or bad release, credential compromise, data corruption, and VPS
  unavailability.
- Each failure scenario requires a distinct response; a single failover plan does not cover
  all cases.
- Failover must be tested before it is claimed.
- Manual recovery is acceptable for MVP if clearly documented in a private runbook, safe to
  execute, and tested.
- Automatic failover is a target architecture goal, not an MVP requirement.
- High availability must not be claimed until monitoring, backup, restore, and failover
  tests exist and have passed.

---

## Secrets Management

- Secrets must not be committed to git.
- Secrets must not be stored in public documentation.
- Secrets must be managed through a provider secret store, GitHub Secrets, or another
  controlled secret manager appropriate to the deployment environment.
- Runtime secrets must be environment-specific; production secrets must not be shared with
  development or test environments.
- Backup and sync processes must not expose secrets or embed them in synced data.
- Rotation and revocation must be possible for every secret and credential.
- Secret names and operational values should not be documented publicly when doing so
  creates an exploitable risk.
- Secret management design is a prerequisite before any real deployment.

---

## Monitoring, Logging, and Audit

Operational monitoring and audit logging are distinct concerns with different requirements.

### Operational Monitoring

- Detects service health, errors, latency, usage, and cost.
- Required before external testing with real users.
- Should surface unexpected cost changes early.
- Should detect service unavailability promptly.

### Audit Logging

- Records security- and approval-relevant events such as session creation, approval
  decisions, client registration, account recovery, and agent access.
- Audit logs may require stronger integrity and retention guarantees than operational logs.
- Audit logs must avoid recording secrets, private keys, recovery codes, approval payload
  secrets, and unnecessary personal data.
- Audit log design must follow existing architecture decisions on what events to record.

### Logging Guardrails

- Logs must not include credentials, tokens, private keys, recovery codes, or approval
  payload content.
- Logs must minimize personal data retention.
- Logging configuration is a security-sensitive concern and belongs to protected asset
  classes.

---

## Security Boundaries

- Public hosting increases exposure and requires explicit API security from day one.
- The backend must enforce TLS/HTTPS for all external communication paths.
- Agent Runtime, mis-client, and admin or operator access paths must remain distinct and
  not intermingled at the API boundary.
- Admin or operator access must not be routed through the same paths as runtime user
  approval flows.
- Backup and sync paths are security-sensitive and must be authenticated and encrypted.
- Provider dashboards, deployment tokens, and infrastructure management credentials are
  privileged assets and must be managed as secrets.
- Infrastructure configuration and deployment credentials belong to protected asset classes
  as defined in the repository safety model.

---

## Relationship to Programming Guides

Hosting constraints may influence language and toolchain choices:

- Cloudflare-oriented code may favor TypeScript where Workers or other Cloudflare
  primitives are used.
- Containerized services or CLI and operational tooling may favor Go.
- Final language decisions are not made here and must remain compatible with the documented
  build, test, and lint command expectations.

See [docs/programming-guides/language-and-toolchain-strategy.md](../docs/programming-guides/language-and-toolchain-strategy.md).

---

## Relationship to Release Context

- Deployment and hosting changes should be associated with an active release context.
- Release builds should identify which environment or hosting target they deploy to.
- Major hosting or operations changes should be called out in release notes for the relevant
  release context.
- Hosting readiness — backup tested, secrets provisioned, monitoring active — is part of
  release readiness.

See [governance/release-context.md](../governance/release-context.md).

---

## Relationship to Repository Safety

- Deployment files, infrastructure configuration, secrets metadata, dependency files, and
  backup or restore documentation belong to protected asset classes.
- Public documentation must avoid leaking operational details.
- Hosting changes require Coder/QAT review and project owner approval before merging.
- Private runbooks and operational details must be stored outside the public repository.

See [governance/repository-safety-and-canaries.md](../governance/repository-safety-and-canaries.md).

---

## MVP Constraints

Tentative MVP operational constraints:

- keep fixed hosting cost as low as practical
- avoid complex multi-region or multi-provider architecture
- avoid requiring enterprise-grade high availability before product-market validation
- prefer simple deployment and rollback over complex orchestration
- require basic backup and restore planning before handling sensitive production data
- require secrets management before any real deployment
- require basic monitoring before external testing
- require explicit decision and documentation before using the VPS for production runtime
- do not claim high availability until tested failover and restore exist

---

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Cloudflare candidate does not support required backend behavior | Keep hosting decision provisional; validate runtime requirements before writing an ADR |
| MVP cost grows unexpectedly | Usage monitoring, cost visibility, simple architecture; treat growth as a scaling issue |
| Backup exists but restore fails | Require restore tests before relying on backups for production readiness |
| Sync to VPS leaks sensitive data | Encrypt backups; limit replicated data; exclude secrets; document runbook privately |
| VPS becomes unmanaged production dependency | Define explicit role; require monitoring, patching, and security review before production use |
| Public docs expose operational details | Document principles only; keep hostnames, IPs, credentials, paths, and runbooks private |
| Operational logs leak secrets or personal data | Apply logging guardrails; redact sensitive fields; minimize data retention |
| High availability claimed prematurely | Do not claim HA until tested failover and restore tests exist and have passed |

---

## Open Questions

The following questions are deferred to later elaboration:

- Which Cloudflare product model is suitable for the MVP backend: Workers, Pages, Durable
  Objects, Queues, a combination, or a containerized alternative?
- Does the MVP backend require containers, serverless functions, durable objects, queues,
  cron-style workers, WebSockets, or long-running workers?
- Where is persistent state stored and on which storage product?
- What is the minimum backup strategy before private beta?
- What is the minimum restore test required to claim backup readiness?
- What is the role of the project-owned VPS: backup target, warm standby, emergency node,
  diagnostics node, or something else?
- Which data may be synced to the VPS and which data must not?
- How often should state be synced?
- How are secrets excluded from backup and sync?
- How are deployment credentials stored and rotated?
- What monitoring is required at MVP before external users are invited?
- What audit log events are required before real approval flows are active?
- What operational events should appear in release notes?
- Which hosting and operations decisions require formal ADRs?
- What is the first environment model: dev/test/staging/prod or a smaller MVP variant?

---

## Related Documents

- [architecture/README.md](README.md) — architecture document index
- [architecture/system-context.md](system-context.md) — system purpose, design goals, and non-goals
- [architecture/component-diagram.md](component-diagram.md) — main components and relationships
- [architecture/control-plane.md](control-plane.md) — control flow and core architectural principle
- [architecture/agent-interface-security.md](agent-interface-security.md) — Agent Runtime trust model and session security
- [architecture/client-identity-and-secure-communication.md](client-identity-and-secure-communication.md) — client key model and secure communication
- [docs/programming-guides/language-and-toolchain-strategy.md](../docs/programming-guides/language-and-toolchain-strategy.md) — language and toolchain candidate direction
- [governance/release-context.md](../governance/release-context.md) — release coordination model
- [governance/repository-safety-and-canaries.md](../governance/repository-safety-and-canaries.md) — protected asset classes and repository safety
- [governance/README.md](../governance/README.md) — development governance model
- [AGENTS.MD](../AGENTS.MD) — operational rules: commits, branches, git safety
- [AI_CONTEXT.md](../AI_CONTEXT.md) — project architecture and domain model
- [adr/0003-domain-and-category-markers.md](../adr/0003-domain-and-category-markers.md) — domain and category marker system
