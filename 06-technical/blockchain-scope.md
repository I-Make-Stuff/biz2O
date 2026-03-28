# Blockchain Scope

This document defines what the blockchain is allowed to do in this system,
what must remain off-chain, and how to support multiple “block types” (transaction
types) without turning the system into an illegal or unmanageable token offering.

This architecture is intentionally multi-capability:
- task validation + reward issuance
- contribution obligations (“holders must provide X”)
- profit-linked distributions (only in compliant modes)
- extensible transaction types (“block types”)

---

## 1. Core Principle

The blockchain is a **tamper-evident record and enforcement engine**.

It is NOT:
- the source of legal authority
- a replacement for contracts
- a way to bypass securities, tax, or labor law

Legal documents define meaning.
Blockchain may enforce and prove behavior.

---

## 2. Three Compliance Modes (Pick Explicitly)

The features you described (obligation + profit rewards) can be implemented,
but only under specific compliance modes.

### Mode A — Internal / Non-Transferable (Default)
Use when you want:
- task/reward accounting
- obligation tracking
- reputation/credits
- internal distributions via governance

Requirements:
- non-transferable units
- permissioned access (or strict allowlist)
- no “profit for holders” promises
- no public trading

### Mode B — Permissioned Security Token (Regulated)
Use when you want:
- token holders receive profit-linked payments
- token represents equity-like rights or revenue share

Requirements:
- treat tokens as securities
- KYC/AML
- transfer restrictions
- exemptions/registration (Reg D, Reg CF, Reg A+ etc.)
- investor disclosures
- securities counsel + compliant platform practices

### Mode C — Public Utility Token (High Risk, Narrow)
Use when:
- token is purely consumptive (access/usage)
- no profit expectation
- no dividends, revenue share, or buyback signaling

This mode is incompatible with “profit for holders.”

**Important:** If you want “give reward to holders based on business profit,”
you are effectively in Mode B (security token) in the U.S. and most jurisdictions.

---

## 3. System Boundaries (On-chain vs Off-chain)

### 3.1 What Must Stay Off-chain
- constitutional authority decisions (amendments, steward appointment/removal)
- legal identity verification documents
- employment/contractor classification logic
- discretionary approvals (final reward issuance, final distribution approval)
- sensitive personal data

### 3.2 What Can Go On-chain Safely (Mode A)
- hashed audit logs (append-only)
- task creation references + policy version pointers
- submission hashes + timestamps
- validation records (AI/algo/human) as attestations
- reward issuance events (non-transferable units)
- allocation credit ledger events (non-transferable)
- distribution event commitments (hashes, snapshots)

### 3.3 What Can Go On-chain in Mode B (Regulated)
All of the above, plus:
- restricted transfer security token ledger (whitelisted transfers)
- dividend/distribution calculations and payouts (still auditable off-chain)
- cap table mirror for tokenized equity (transfer restricted)

---

## 4. “Block Types” = Transaction Types + Modules

When you say “many types of blocks,” implement this as:
- a single chain (or permissioned chain)
- multiple **transaction types**
- governed by versioned policies

### 4.1 Canonical Transaction Types (V1)

**Audit / Anchoring**
- AUDIT_BATCH_ANCHOR
- POLICY_VERSION_ANCHOR
- DISTRIBUTION_EVENT_ANCHOR

**Task System**
- TASK_CREATED
- SUBMISSION_COMMITTED
- VALIDATION_ATTESTED
- WINNER_DECLARED (requires authority signature)

**Reward Tokens (non-transferable)**
- REWARD_MINTED
- REWARD_ADJUSTED
- REWARD_REDEEMED
- REWARD_EXPIRED
- REWARD_REVOKED

**Allocation Credits (non-transferable)**
- CREDIT_MINTED
- CREDIT_ADJUSTED
- CREDIT_EXPIRED
- CREDIT_REVOKED

**Obligations (“holders must provide X”)**
- OBLIGATION_CREATED
- OBLIGATION_ACCEPTED
- OBLIGATION_PROGRESS
- OBLIGATION_VERIFIED
- OBLIGATION_BREACHED
- OBLIGATION_RELEASED

**Distributions**
- DISTRIBUTION_DRAFTED (commit snapshot)
- DISTRIBUTION_APPROVED (authority signature)
- DISTRIBUTION_EXECUTED (payout refs)

### 4.2 Module System
Each transaction type belongs to a module:

- module.audit
- module.tasks
- module.rewards
- module.credits
- module.obligations
- module.distributions
- module.security_token (Mode B only)

Modules have:
- strict schema
- policy version reference
- authority signature rules
- audit event mapping

---

## 5. Obligations for Holders (Your “Manufacture Products” Example)

### 5.1 What “Holders” Means
Define “holder” precisely. Options:

- **Role Holder**: someone assigned a role (manufacturer, operator)
- **Obligation Holder**: someone who accepts a specific obligation contract
- **Token Holder**: someone holding a token balance (dangerous if transferable)

**Recommended (Mode A):**
Use Role Holder / Obligation Holder, not Token Holder.

Reason:
Token-based obligations become coercive and resemble investment contracts or
employment misclassification if not handled carefully.

### 5.2 Obligation Object (On-chain Reference + Off-chain Contract)
The obligation must point to an off-chain agreement defining:
- scope of work (deliverables)
- quality standards
- deadlines
- compensation terms (if any)
- dispute resolution
- liability and warranties
- termination terms

On-chain stores:
- obligation_id
- hash of the agreement
- acceptance signature
- milestone attestations
- verification signature
- breach events

### 5.3 Enforcement Types
Enforcement should be bounded:

Allowed:
- reputation effects (reduce eligibility for tasks)
- credit issuance suspension
- reward redemption limitations
- role removal
- internal penalties defined in contract

Avoid or handle with counsel:
- automatic financial penalties without due process
- coercive mechanisms tied to wages
- forced performance

---

## 6. Profit-Based Rewards to Holders (High-Risk Feature)

### 6.1 Default Rule
In Mode A (internal/non-transferable), do NOT implement:
- “profit rewards to holders” as a token right

You may implement:
- discretionary bonuses
- pool distributions based on allocation credits
- performance-based payouts with contracts

### 6.2 If You Want Profit Rewards to Holders Anyway
That is Mode B (security token). You must design:

- token as a security
- whitelist + KYC
- restricted transfers
- disclosure framework
- dividend policy and board approvals
- tax reporting plan

On-chain can compute and record dividend entitlements, but legal approval and
compliance gates are mandatory.

### 6.3 Language and UX Guardrails
Never describe it as:
- “guaranteed profit”
- “passive income”
- “dividend just for holding”

Use compliant language:
- “distributions, if any, as approved by the company”
- “subject to legal restrictions and board approval”
- “no assurance of distributions”

---

## 7. Multi-Chain or Single Chain?

### 7.1 Single Chain (Recommended)
Use one permissioned chain and keep:
- modules segmented
- transaction schemas strict
- policy versions explicit

Pros:
- simpler verification
- easier anchoring and audits
- fewer bridges (bridges are attack surfaces)

### 7.2 Multiple Chains
Only use if you have a clear separation:
- internal chain for audit/task/credits
- regulated chain for security tokens (Mode B)
- public anchoring chain for hash proofs only

---

## 8. On-Chain Data Minimization

Store only:
- ids
- hashes
- timestamps
- signatures
- policy version references
- non-sensitive numeric balances

Never store:
- names, addresses, SSNs, bank details
- full submissions
- private contracts

---

## 9. Signature & Authority Requirements

Every sensitive transaction requires signatures:

- Managing Authority signature for:
  - WINNER_DECLARED
  - CREDIT_REVOKED (if not policy automatic)
  - DISTRIBUTION_APPROVED
  - POLICY_ACTIVATED (or anchor)

- Steward signature for:
  - CREDIT_MINTED / ADJUSTED within pool scope
  - REWARD_REDEEMED into credits within pool rules

- Operational Authority signature for:
  - VALIDATION_ATTESTED (human)
  - OBLIGATION_VERIFIED (if operational)

AI outputs never count as final signatures.

---

## 10. Extensibility Rules (“Add New Block Types”)

To add a new transaction type:
1. Define schema (fields, required refs)
2. Define authority signature requirements
3. Define invariants (what it cannot do)
4. Link to a Policy version
5. Add audit mapping (AuditEvent generation)
6. Add downgrade/rollback plan (deprecate safely)

New types must not:
- create transferability for credits/rewards
- create token-based authority
- promise profit or appreciation without Mode B compliance
- bypass final human approval gates

---

## 11. Recommended Scope for V1

Implement V1 as Mode A:

On-chain:
- audit anchors
- task lifecycle commitments
- reward/credit ledgers (non-transferable)
- obligation attestations
- distribution snapshots and approvals

Off-chain:
- identity and access control
- legal agreements
- payout rails (banking)
- governance decisions

Reserve Mode B for later, explicitly.

---

## 12. Summary

You can support “many block types” by implementing:
- a modular transaction system
- strict schemas and authority signatures
- policy-version referencing
- auditable anchors

But “profit rewards to token holders” places you in a regulated securities
framework unless you remove profit expectation and transferability.

The system remains coherent when:
- blockchain proves and enforces workflow
- humans hold bounded authority
- legal documents define rights
