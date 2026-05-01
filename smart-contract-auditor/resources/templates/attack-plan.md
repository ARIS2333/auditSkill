# Attack Plan Template

Use this template for `audit-output/phase-3-planning/attack-plan.md`. This is the **prioritized hit list** for Phase 4 verification — every entry should identify a specific attack vector with enough detail to guide targeted code reading and PoC development.

**How to populate:** Work through each section using only Phase 1 and Phase 2 outputs (no source code reading in this phase). Reference checklists and playbooks for ideas. Be specific — not "reentrancy might exist" but "Vault.withdraw() sends ETH via low-level call at L55 before updating balances at L60, attacker could re-enter via receive()."

---

```markdown
# Attack Plan

> **Generated from:** Phase 1 structural summary + Phase 2 codebase overview
> **Date:** [YYYY-MM-DD]

---

## 1. Value Flow Attack Vectors

Derived from Phase 2: User Flows (§6), Value Flow (§8), Token Accounting (§8), Key Invariants (§12). Apply Phase 3 Step 1 analysis and `resources/checklists/adversarial-framework.md` Core Questions.

| # | Target Flow | Attack Vector | Contracts Involved | Priority |
|---|-------------|--------------|-------------------|----------|
| 1 | [e.g., Deposit → Mint shares] | [e.g., First depositor inflation — donate to inflate share price before victim deposits] | [Vault.deposit, Vault.convertToShares] | [Critical/High/Medium] |

---

## 2. Access Control Attack Vectors

Derived from Phase 1: Entry Points & Access Control (§3), Unguarded Functions (§4). Phase 2: Access Control Matrix (§7). Apply Phase 3 Step 2 analysis.

| # | Target Function | Attack Vector | Why Suspicious | Priority |
|---|----------------|--------------|----------------|----------|
| 1 | [e.g., Registry.setOracle()] | [e.g., No modifier, inline require only — check if bypassable] | [Structural summary §4: listed as unguarded] | [Critical/High/Medium] |

---

## 3. Domain-Specific Attack Vectors

Derived from matching playbooks in `resources/checklists/playbooks/` and checklists in `resources/checklists/`. List which playbooks were consulted and which checks flagged potential issues.

**Playbooks consulted:** [e.g., vault-erc4626.md, proxy-upgradeable.md]

**Checklists consulted:** [e.g., non-standard-tokens.md, adversarial-framework.md]

| # | Source (Playbook/Checklist § Item) | Target | Attack Vector | Priority |
|---|-----------------------------------|--------|--------------|----------|
| 1 | [e.g., vault-erc4626.md §1] | [Vault.convertToShares] | [No virtual shares offset — first depositor inflation possible] | [High] |

---

## 4. Privilege & Management Risk Analysis

Derived from Phase 2: Access Control Matrix (§7), Known Risks & Trust Assumptions (§13). Reference `resources/checklists/privilege-risk.md` for the full checklist.

### 4.1 Role Inventory

| Role | Address Type | Key Powers | Funds at Risk |
|------|-------------|-----------|---------------|
| [owner] | [EOA / Multisig / Timelock] | [list critical functions] | [% TVL or description] |

### 4.2 Key Compromise Scenarios

| # | Role | Worst-Case Scenario | Mitigations | Severity |
|---|------|-------------------|-------------|----------|
| 1 | [owner] | [Can upgrade impl and drain all funds] | [No timelock, single EOA] | [Critical centralization risk] |

### 4.3 Role Escalation Paths

[Document any paths where one role can grant/escalate to another role, or bypass intended restrictions. Write "None identified" if none found.]

---

## 5. Slither Detector Findings (Optional)

> This section is populated only if the user opted in to Slither vulnerability scanning in Phase 0.

### High/Medium Findings

| # | Detector | Contract.Function | Description | Relates to Attack Plan # |
|---|----------|-------------------|-------------|-------------------------|
| 1 | [reentrancy-eth] | [Vault.withdraw] | [External call before state update] | [§1 #3] |

Low and Informational findings are grouped separately in `audit-output/phase-3-planning/slither-low-info.md`.

---

## 6. Verification Priority Order

Rank all attack vectors from sections 1-5 by priority for Phase 4 verification:

| Rank | Source | Target | Attack Vector | Verify Method |
|------|--------|--------|--------------|---------------|
| 1 | §1 #1 | [Vault.deposit] | [First depositor inflation] | [PoC: donate + deposit sequence] |
| 2 | §4 #1 | [Owner role] | [Key compromise — no timelock on upgrade] | [Write-up only (centralization)] |
| 3 | §2 #1 | [Registry.setOracle()] | [Missing access control] | [PoC: call as unprivileged user] |
```
