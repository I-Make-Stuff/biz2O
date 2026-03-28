# Reference Implementation (Platform Architecture)

This document describes a reference implementation for the Constitutional
Governance Platform.

Goals:
- containerized deployment (Docker-first, Kubernetes-ready)
- secure by default (least privilege, auditability, secrets hygiene)
- low latency for task + validation + ledger writes
- multi-tenant (many businesses/orgs) with isolated governance roots
- multiple governance methods (single founder, board, committee stewards)
- optional blockchain anchoring without putting rights on-chain

This is language-agnostic. It defines services, interfaces, and deployment patterns.

---

## 1. Reference Architecture Overview

Core design:
- Off-chain is canonical for legal meaning and authority decisions
- On-chain (optional) anchors hashes and provides tamper-evidence
- All economic state transitions are append-only ledgers
- Multi-tenant isolation is enforced at the data and auth layers

### 1.1 System Components

- API Gateway (public edge)
- Identity & Access (OIDC/AuthN/AuthZ)
- Governance Service (authority, roles, proposals)
- Policy Service (versioned policies, activation)
- Task Service (tasks, submissions, validation workflows)
- Ledger Service (rewards + credits append-only ledgers)
- Distribution Service (waterfall calculations + snapshots)
- Audit Service (event log + hashing + anchoring)
- Content Store (submissions/artifacts)
- Optional Chain Anchor Service (public chain anchoring)
- Admin Console (operator UI)
- Org Console (business UI)

---

## 2. Multi-Tenant Model (Organizations)

### 2.1 Organization as First-Class Root

Every record must include `org_id`.

Each org has:
- its own constitutional document hash
- its own authority graph
- its own policies and pools
- its own audit stream

**Hard rule:** No cross-org reads/writes without explicit platform-admin scope.

### 2.2 Recommended Tenant Isolation Strategy

Default (recommended):
- single shared cluster
- shared services
- strict row-level isolation (org_id + auth)
- per-tenant encryption keys

High-security option:
- dedicated namespace per org (Kubernetes)
- separate databases per org
- separate object store buckets

---

## 3. Service Responsibilities & Interfaces

### 3.1 API Gateway
Responsibilities:
- request routing
- rate limiting
- WAF controls
- request_id injection
- auth token validation (delegated to IdP)
- structured logging

Suggested interfaces:
- REST/JSON for broad compatibility
- gRPC between internal services for low latency

---

### 3.2 Identity & Access Service (Auth)
Responsibilities:
- OIDC integration (Google/Microsoft/Okta or self-hosted)
- service account issuance (for validators, automation)
- RBAC + ABAC enforcement primitives
- session and device policies

Key requirements:
- MFA support for privileged roles
- separation of duties (platform admin vs org admin)
- signed authorization context forwarded to services

---

### 3.3 Governance Service
Responsibilities:
- roles and assignments (Managing Authority, Stewards, Operational)
- delegation records (scope, duration, revocation)
- governance method selection per org:
  - single managing authority
  - board model
  - steward committee model
- amendment proposal workflow:
  - propose → review → approve → publish

Outputs:
- authoritative “who can do what” decisions used by other services

---

### 3.4 Policy Service
Responsibilities:
- versioned policies (reward mapping, credit rules, validation rules)
- activation & retirement with effective dates
- policy hashing for audit anchoring
- compatibility checks (policy invariants)

Key feature:
- policies are machine-readable JSON plus human-readable markdown.

---

### 3.5 Task Service
Responsibilities:
- tasks, submissions, and validation workflows
- supports “mining-style” competition (multi submissions)
- integrates with validation engines:
  - manual validator UI
  - algorithmic validator (deterministic checks)
  - AI validator (advisory scoring)

Hard rule:
- AI cannot finalize issuance; it can only attest.

---

### 3.6 Ledger Service (Rewards + Credits)
Responsibilities:
- append-only ledger entries for:
  - Reward Tokens
  - Allocation Credits
- balance materialization (fast reads)
- ledger invariants enforcement:
  - non-transferability
  - authority signatures required
  - no silent edits (append-only)
- supports revocation/expiration as new entries

Performance pattern:
- write: append ledger entry
- read: materialized balance table updated asynchronously or transactionally

---

### 3.7 Distribution Service
Responsibilities:
- distribution event creation (draft)
- waterfall calculation based on:
  - directive percentages snapshot
  - pool credit balances snapshot
- generates DistributionLineItems
- requires Managing Authority approval to execute
- stores payout refs (off-chain rails)

---

### 3.8 Audit Service
Responsibilities:
- centralized immutable audit log
- per-org audit streams
- hashing pipeline:
  - event hashes
  - Merkle trees per batch
  - anchor-ready merkle roots
- export tooling (regulatory audits)

---

### 3.9 Content Store Service
Responsibilities:
- submission file storage (object store)
- content hashing
- signed URLs for controlled access
- malware scanning hooks

Never store sensitive identity docs here unless in a restricted bucket.

---

### 3.10 Chain Anchor Service (Optional)
Responsibilities:
- takes Merkle roots from Audit Service
- anchors to:
  - public chain (hash-only)
  - or permissioned chain (internal audit chain)
- records tx_hash references

Hard rule:
- chain anchoring stores proofs, not rights.

---

## 4. Data Storage & Consistency

### 4.1 Recommended Datastores (Default)
- Postgres (primary relational store)
- Redis (caching, rate limiting, job dedup)
- Object store (S3-compatible) for submissions
- Optional: OpenSearch/ELK for audit search

### 4.2 Consistency Model
- Strong consistency for:
  - ledger writes
  - role assignments
  - policy activation
- Eventual consistency acceptable for:
  - search indices
  - analytics dashboards
  - audit anchor batching

### 4.3 Event Bus
Use an event bus for internal fan-out:
- NATS, Kafka, or RabbitMQ (choose one)
Events:
- ledger_entry_created
- policy_activated
- role_changed
- task_validated
- distribution_approved

---

## 5. Security Model (Zero Trust-ish, Practical)

### 5.1 Identity & AuthZ
- OIDC for users
- mTLS for service-to-service
- JWT with short TTL for requests
- service accounts for automation (scoped)

### 5.2 Authorization
Enforce at multiple layers:
1. API Gateway (coarse)
2. Service layer (fine-grained)
3. Database RLS (org_id isolation)

### 5.3 Secrets
- use a secrets manager (Vault, AWS Secrets Manager, etc.)
- never bake secrets into images
- rotate keys

### 5.4 Auditability
- every sensitive write generates an AuditEvent
- include: actor_id, role, org_id, rationale, request_id
- store immutable logs + hash anchors

### 5.5 Supply Chain
- signed container images
- SBOM generation
- dependency scanning

---

## 6. Low Latency Patterns

### 6.1 Hot Path (Task → Validate → Reward)
Keep the hot path minimal:
1. submission stored (object store)
2. validation recorded
3. reward ledger appended
4. balance materialized

Optimize with:
- gRPC internal calls
- Redis cache for role checks
- DB transactions only where necessary
- asynchronous post-processing (search index, analytics)

### 6.2 Materialized Balances
For fast UI reads:
- maintain `reward_balance` and `credit_balance` tables per org/person/pool
- updated by:
  - synchronous trigger (simpler, slower)
  - or async consumer (faster, more moving parts)

Default:
- synchronous for MVP
- async when scaling

---

## 7. Governance Methods (Multi-Mode Support)

Each org selects a governance mode at onboarding:

### 7.1 Mode: Founder-Managed
- single Managing Authority
- stewards optional
- simplest and fastest

### 7.2 Mode: Board-Managed
- board approvals required for:
  - distributions
  - constitutional amendments
  - policy activation
- stewards as committees

### 7.3 Mode: Steward Committee
- each directive pool has steward committee
- Managing Authority retains veto + compliance duty
- strong separation of duties

Implementation:
- Governance Service exposes a uniform permission check:
  `can(actor, action, scope) -> allow/deny + rationale`

---

## 8. Containerization & Deployment

### 8.1 Repo Layout (Suggested)
