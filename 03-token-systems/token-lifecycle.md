# Token Lifecycle

This document defines the lifecycle of all internal units used by the organization,
including:

- Equity Units
- Allocation Credits
- Reward Tokens

Each unit has a distinct legal meaning, authority path, and lifecycle.
No unit may bypass its defined lifecycle.

---

## 1. Design Principle

No token, credit, or unit:
- Exists by default
- Confers authority
- Is permanent by assumption

All units:
- Are created intentionally
- Exist for a defined purpose
- Are governed by human authority
- Are auditable

---

## 2. Unit Classes Overview

| Unit Type | Purpose | Transferable | Represents Property |
|---------|--------|--------------|---------------------|
| Equity Units | Ownership | Restricted | Yes |
| Allocation Credits | Distribution weighting | No | No |
| Reward Tokens | Incentives | No | No |

Each unit follows a separate lifecycle.

---

## 3. Equity Units Lifecycle

### 3.1 Creation (Issuance)

Equity Units are created only by:
- Constitutional Authority (via charter or operating agreement)
- Managing Authority (within authorized limits)

Issuance requires:
- Legal documentation
- Vesting terms
- Cap table update

Blockchain systems, if used, may only mirror issuance.

---

### 3.2 Vesting

Equity Units may vest:
- Over time
- By milestone
- By performance

Unvested units:
- Are subject to forfeiture
- Carry no distribution rights unless explicitly stated

---

### 3.3 Transfer

Transfers:
- Require approval
- Are subject to right of first refusal
- May be prohibited entirely

No automated or on-chain transfer is valid without legal approval.

---

### 3.4 Termination

Equity Units terminate upon:
- Buyback
- Forfeiture
- Dissolution
- Conversion to another equity class

Equity Units do not expire automatically.

---

## 4. Allocation Credits Lifecycle

### 4.1 Creation (Minting)

Allocation Credits are minted only when:
- A documented contribution occurs
- A Steward or Managing Authority authorizes issuance

Credits are minted:
- Into a specific directive pool
- With a defined rationale
- With an audit record

There is no fixed supply.

---

### 4.2 Accumulation

Credits accumulate as:
- Contribution continues
- Milestones are met
- Authority approves increases

Credits do NOT:
- Compound
- Earn interest
- Appreciate independently

---

### 4.3 Adjustment

Credits may be adjusted:
- Upward for additional contribution
- Downward for inactivity
- Rebalanced during periodic reviews

Adjustments must:
- Follow predefined rules
- Be logged
- Be reviewable

---

### 4.4 Usage

Credits are used only to:
- Calculate internal distribution proportions
- Determine participation weight within a pool

Credits do NOT:
- Guarantee payout
- Trigger distributions automatically
- Persist across unrelated pools

---

### 4.5 Suspension

Credits may be suspended when:
- Contribution pauses
- Role becomes inactive
- Investigation is underway

Suspended credits:
- Do not participate in distributions
- Are not destroyed

---

### 4.6 Expiration & Revocation

Credits expire or are revoked upon:
- Exit from the organization
- Breach of agreement
- End of contribution window

Upon expiration:
- Credits are burned
- No residual rights remain

Credits are never transferable or inheritable.

---

## 5. Reward Tokens Lifecycle

### 5.1 Creation (Earning)

Reward Tokens are earned by:
- Completing defined tasks
- Meeting operational criteria
- Achieving performance goals

Creation requires:
- Task validation
- Operational Authority confirmation
- Steward or Managing approval

Reward Tokens may be inflationary.

---

### 5.2 Holding

Reward Tokens:
- Are held internally
- Carry no governance rights
- Do not represent value

They exist solely to:
- Track effort
- Enable incentives
- Signal contribution

---

### 5.3 Redemption

Reward Tokens may be redeemed for:
- Bonuses
- Access
- Priority
- Conversion into Allocation Credits (if authorized)

Redemption rules are:
- Pool-specific
- Time-bound
- Revocable

---

### 5.4 Expiration

Reward Tokens may:
- Expire after a time window
- Reset periodically
- Be revoked upon exit

Expired tokens have no residual effect.

---

## 6. Cross-Unit Transitions

### 6.1 Reward Tokens → Allocation Credits

This transition:
- Is NOT automatic
- Requires Steward approval
- Must follow a defined exchange rule

Example:
- 100 Reward Tokens → 10 Allocation Credits
- Conversion limited by pool caps

---

### 6.2 Allocation Credits → Equity Units

This transition:
- Is rare
- Requires Constitutional Authority
- Typically occurs during:
  - Conversion to corporation
  - Formal equity grants

No individual has a right to convert credits to equity.

---

## 7. Authority Involvement Matrix

| Action | Required Authority |
|-----|------------------|
| Issue Equity | Constitutional / Managing |
| Vest Equity | Automatic / Managing |
| Mint Credits | Steward / Managing |
| Adjust Credits | Steward |
| Revoke Credits | Managing |
| Issue Rewards | Operational + Approval |
| Redeem Rewards | Steward |
| Amend Rules | Constitutional |

---

## 8. Technical Representation

```markdown
Tokens and credits may be represented digitally,
including via blockchain or databases,
but such representations do not alter legal status.
