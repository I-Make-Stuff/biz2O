# Data Model

This document defines the canonical data model for the organization’s
task, validation, reward, allocation, and audit systems.

The model is designed to:
- Be implementable in a relational DB or document store
- Support optional on-chain anchoring of audit hashes
- Preserve authority separation (governance ≠ tokens)
- Avoid creating transferable or security-like instruments

All entities below are internal records unless explicitly stated otherwise.

---

## 1. Design Constraints

1. Authority is role-based, not token-based.
2. Allocation Credits and Reward Tokens are non-transferable.
3. Equity Units are tracked but transfer-restricted and legally defined elsewhere.
4. Every state transition must be auditable (who, what, when, why).
5. Technical systems enforce arithmetic and workflow; legal documents govern meaning.

---

## 2. Entity Overview

Core entities:

- Person
- RoleAssignment
- DirectivePool
- Task
- Submission
- Validation
- RewardLedgerEntry
- AllocationCreditLedgerEntry
- DistributionEvent
- DistributionLineItem
- Policy
- AuditEvent

Optional entities:

- EquityLedgerEntry (mirror only)
- OnChainAnchor (hash anchoring)

---

## 3. Enumerations (Canonical)

### 3.1 DirectivePoolType
- FINANCIAL_ASSISTANCE
- LABOR_OPERATIONS
- FOUNDER
- PLANNING_STRATEGY

### 3.2 AuthorityRole
- CONSTITUTIONAL_AUTHORITY
- MANAGING_AUTHORITY
- STEWARD_FINANCE
- STEWARD_OPERATIONS
- STEWARD_PLANNING
- OPERATIONAL_AUTHORITY
- VALIDATOR (human or service account)

### 3.3 TaskValidationMode
- MANUAL
- ALGORITHMIC
- AI_ASSISTED
- HYBRID

### 3.4 TaskWinnerMode
- FIRST_VALID
- BEST_SCORE
- MULTI_WINNER
- STEWARD_SELECT

### 3.5 LedgerEntryType
- MINT
- ADJUST
- REVOKE
- EXPIRE
- REDEEM
- BURN

### 3.6 DistributionStatus
- DRAFT
- APPROVED
- EXECUTED
- VOIDED

---

## 4. Entity Definitions (Fields)

Field notation:
- `id`: unique identifier (UUID recommended)
- `created_at`, `updated_at`: timestamps
- `created_by`: Person or ServiceAccount reference
- `meta`: JSON for extensions (no business logic in meta)

### 4.1 Person
Represents an internal participant identity (employee, contractor, contributor).

Fields:
- id
- display_name
- legal_name (optional, restricted access)
- email (optional)
- status: ACTIVE | INACTIVE | SUSPENDED
- join_date
- exit_date (nullable)
- kyc_status (optional): NOT_REQUIRED | PENDING | VERIFIED | REJECTED
- meta

Notes:
- Do not store sensitive data unless required. Keep separate restricted tables if needed.

---

### 4.2 RoleAssignment
Maps a Person to authority roles over time.

Fields:
- id
- person_id
- role: AuthorityRole
- scope_pool_id (nullable)  # for steward roles, ties to a DirectivePool
- start_at
- end_at (nullable)
- granted_by (person_id)
- grant_reason
- revocable: boolean (default true)
- meta

Constraints:
- A steward role must have `scope_pool_id`.
- Founder pool steward role is prohibited (non-delegable).

---

### 4.3 DirectivePool
Represents a constitutional directive pool and its internal allocation state.

Fields:
- id
- type: DirectivePoolType
- name
- directive_percent: decimal(5,2)  # 30.00, 50.00, etc. (constitutional target)
- is_constitutional_locked: boolean
- steward_role_required: AuthorityRole (nullable)  # e.g., STEWARD_OPERATIONS
- credit_policy_id (nullable)
- reward_policy_id (nullable)
- meta

Constraints:
- Sum of directive_percent across pools must equal 100.00 for active constitutional pools.
- If `is_constitutional_locked` true, only Constitutional Authority can modify directive_percent.

---

### 4.4 Task
Represents an opportunity for contribution.

Fields:
- id
- title
- description
- pool_id (DirectivePool.id)
- created_by (person_id)
- status: OPEN | CLOSED | ARCHIVED
- validation_mode: TaskValidationMode
- winner_mode: TaskWinnerMode
- submission_window_start
- submission_window_end
- max_rewards: integer  # ceiling in Reward Tokens
- reward_floor: integer # may be 0
- reward_policy_id (nullable)
- allocation_credit_eligibility: boolean
- allocation_credit_conversion_rule_id (nullable)
- tags: array<string>
- meta

Notes:
- `max_rewards` defines the max Reward Tokens that can be issued for this task.
- No guaranteed payouts; “0” is always permitted.

---

### 4.5 Submission
A participant’s submission to a Task.

Fields:
- id
- task_id
- submitter_id (person_id)
- submitted_at
- content_ref  # pointer to storage (file path, URL, blob ID)
- content_hash (optional)
- notes (optional)
- status: RECEIVED | UNDER_REVIEW | ACCEPTED | REJECTED | WITHDRAWN
- meta

Constraints:
- Submissions may be many-to-one per task per person (unless policy forbids).

---

### 4.6 Validation
Records validation events for a submission.

Fields:
- id
- submission_id
- task_id
- validator_type: HUMAN | ALGO | AI
- validator_id (person_id or service_account_id)
- validated_at
- result: PASS | FAIL | SCORE
- score_value (nullable decimal)
- rationale
- evidence_ref (optional)
- meta

Rules:
- If AI validation is used, must be accompanied by a HUMAN confirmation record
  before rewards can be issued (enforced at workflow level).

---

### 4.7 RewardLedgerEntry
Immutable ledger of Reward Token changes.

Fields:
- id
- person_id
- pool_id
- related_task_id (nullable)
- related_submission_id (nullable)
- entry_type: LedgerEntryType  # MINT/ADJUST/REVOKE/EXPIRE/REDEEM/BURN
- amount: integer  # positive or negative allowed; enforce via entry_type rules
- reason
- authorized_by (person_id)
- authority_role_used: AuthorityRole
- effective_at
- meta

Constraints:
- Reward Tokens are non-transferable: no `from_person_id` / `to_person_id`.
- Any negative amount must reference a prior issuance event or policy basis.

Derived:
- Reward balance per person per pool = sum(amount) over RewardLedgerEntry

---

### 4.8 AllocationCreditLedgerEntry
Immutable ledger of Allocation Credit changes.

Fields:
- id
- person_id
- pool_id
- entry_type: LedgerEntryType  # MINT/ADJUST/REVOKE/EXPIRE/REDEEM/BURN
- amount: integer
- reason
- authorized_by (person_id)
- authority_role_used: AuthorityRole
- evidence_ref (optional)  # contribution record, contract, milestone proof
- effective_at
- meta

Constraints:
- Credits are non-transferable.
- Only steward for that pool or Managing Authority may authorize (enforce in service layer).

Derived:
- Credit balance per person per pool = sum(amount) over AllocationCreditLedgerEntry

---

### 4.9 DistributionEvent
A discrete distribution calculation and execution record.

Fields:
- id
- period_start (nullable)
- period_end (nullable)
- distributable_value_amount: decimal  # currency amount or unit value
- distributable_value_currency: string # USD, etc.
- status: DistributionStatus
- determined_by (person_id)  # Managing Authority
- approved_by (person_id)    # Managing Authority or defined approver
- approved_at (nullable)
- executed_at (nullable)
- notes
- meta

Constraints:
- Only Managing Authority can set `distributable_value_amount` and approve.

---

### 4.10 DistributionLineItem
Line items created from the waterfall and internal pool credit balances.

Fields:
- id
- distribution_event_id
- pool_id
- directive_percent_snapshot: decimal(5,2)
- pool_amount: decimal
- recipient_person_id
- recipient_credit_balance_snapshot: integer
- pool_total_credit_snapshot: integer
- recipient_share_percent: decimal(9,6)
- payout_amount: decimal
- payout_method: CASH | DEFERRED | INTERNAL_ACCOUNTING | OTHER
- payout_ref (optional)
- meta

Rules:
- The `directive_percent_snapshot` locks the rule at the time of calculation.
- Recipients with 0 credits receive 0 payout from that pool.

---

### 4.11 Policy
Versioned policies used by pools and tasks.

Fields:
- id
- policy_type: CREDIT_POLICY | REWARD_POLICY | VALIDATION_POLICY | CONVERSION_RULE
- name
- version
- status: DRAFT | ACTIVE | RETIRED
- body: JSON  # machine-readable rules
- effective_at
- retired_at (nullable)
- authored_by (person_id)
- meta

Examples:
- Credit decay rule
- Reward expiration rule
- Validation requirements
- Reward→Credit conversion rates and caps

---

### 4.12 AuditEvent
Tamper-evident audit log for all critical actions.

Fields:
- id
- event_type
- actor_id (person_id or service_account_id)
- actor_role: AuthorityRole
- occurred_at
- target_type
- target_id
- action
- before_state_hash (optional)
- after_state_hash (optional)
- rationale
- request_id (optional)
- meta

Notes:
- Every ledger entry creation MUST have a corresponding AuditEvent.

---

## 5. Optional: Equity Ledger Mirror

### 5.1 EquityLedgerEntry (Mirror Only)
This tracks equity positions as a mirror of legal ownership documents.

Fields:
- id
- person_id
- equity_class: COMMON | PREFERRED | LLC_UNIT
- amount: integer
- vesting_schedule_ref (optional)
- transfer_restricted: boolean
- authorized_by (person_id)
- authority_role_used
- effective_at
- meta

Constraints:
- This ledger does not define legal equity; it mirrors it.

---

## 6. Optional: On-Chain Anchoring

### 6.1 OnChainAnchor
Anchors audit hashes to an external chain without putting rights on-chain.

Fields:
- id
- anchor_type: AUDIT_BATCH | DISTRIBUTION_EVENT | POLICY_VERSION
- target_id
- merkle_root_hash
- chain_name
- tx_hash
- anchored_at
- meta

Principle:
- Anchor proofs, not rights.

---

## 7. Invariants & Compliance Guards (Must Enforce)

1. Non-transferability:
   - No entity supports peer-to-peer token transfers.

2. Authority separation:
   - No decisions based on token balances.

3. Human finality:
   - AI validation cannot be sole final approval for reward issuance.

4. Constitutional lock:
   - Directive percentages require Constitutional Authority to change.

5. Ledger immutability:
   - Never update ledger entries; append only.

6. Audit completeness:
   - Every sensitive write produces an AuditEvent.

---

## 8. Workflow Summary (Happy Path)

1. Managing/Steward creates Task.
2. Participants submit Submissions.
3. Validations recorded (AI/Algo/Manual).
4. Steward/Managing approves winner(s).
5. RewardLedgerEntry MINT entries issued.
6. (Optional) Reward tokens redeemed into Allocation Credits via policy.
7. Managing sets DistributionEvent distributable_value_amount.
8. System calculates pool splits and DistributionLineItems using credit snapshots.
9. Managing approves and executes; payout references recorded.
10. Audit batch anchored on-chain (optional).

---

## 9. API Surface (Suggested)

Minimum endpoints / commands:

- POST /tasks
- POST /tasks/{id}/submissions
- POST /submissions/{id}/validations
- POST /rewards/issue
- POST /rewards/redeem
- POST /credits/issue
- POST /distributions/create
- POST /distributions/{id}/approve
- POST /distributions/{id}/execute
- GET  /people/{id}/balances
- GET  /audit/events

All write endpoints require:
- actor identity
- actor role context
- rationale string
- request_id for traceability

---

## 10. Storage Notes

Relational DB recommended for:
- Ledger consistency
- Referential integrity
- Auditability

Content storage for submissions should be separate:
- Object store or filesystem
- Hash references stored in Submission

---

## 11. Schema Versioning

This data model is versioned.
Changes require:
- Policy update
- Migration plan
- Audit note explaining the reason

Recommended:
- `schema_version` recorded globally and per entity row via meta.

