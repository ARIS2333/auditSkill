# Structural Summary Template

Use this template for `audit-output/phase-1-recon/structural-summary.md`. This is the **pre-digested structural reference** for the entire audit — not just an index. Extract the most critical data from printer output into the compact tables below so that later phases can look up structural facts here without re-reading large raw printer files.

**How to populate:** For each section, read the listed printer file(s), extract the relevant data into the table, and move on. Do NOT copy raw printer output verbatim — summarize into the structured format shown.

---

```markdown
# Structural Summary

## 1. Codebase Snapshot

| Metric | Value |
|--------|-------|
| Total contracts (in scope) | [N] |
| Source lines of code (SLOC) | [N] |
| External/public functions | [N] |
| ERCs detected | [e.g., ERC20, ERC721, None] |
| Scope classification | [Small / Medium / Large — from Phase 0 Step 6] |

**Source:** `printers/human-summary.txt`

## 2. Inheritance Tree

Extract the full inheritance hierarchy as a compact text tree.

```
ContractA
├── BaseContractX
│   └── OpenZeppelinY
└── InterfaceZ

ContractB
└── BaseContractX
    └── OpenZeppelinY
```

**Source:** `printers/inheritance.txt` + `printers/inheritance-graph*.dot`

## 3. Entry Points & Access Control — Compact Function Reference

The most critical table. Primary lookup for "what does function X do, who can call it, and what state does it touch?"

| Contract | Function | Vis | Modifiers | State Vars Written | Auth Check |
|----------|----------|-----|-----------|-------------------|------------|
| [Vault] | [deposit(uint256)] | [ext] | [whenNotPaused, nonReentrant] | [balances, totalSupply] | [None (open)] |
| [Vault] | [withdraw(uint256)] | [ext] | [nonReentrant] | [balances, totalSupply] | [require(balances[msg.sender] >= amount)] |
| [Vault] | [setStrategy(address)] | [ext] | [onlyOwner] | [strategy] | [onlyOwner] |

**Rules:**
- Include ALL external and public state-modifying functions
- For `Modifiers`: list actual modifier names. If none, write `NONE`
- For `Auth Check`: summarize access control — modifier name, inline require pattern, or `NONE (open)` if unguarded
- Omit view/pure functions unless called by state-modifying functions via internal calls

**Source:** `printers/function-summary.txt` + `printers/modifiers.txt` + `printers/vars-and-auth.txt`

## 4. Unguarded State-Modifying Functions

Functions that modify state but have NO modifiers AND NO require/assert checks. Highest-priority audit targets.

| Contract | Function | State Vars Written | Notes |
|----------|----------|--------------------|-------|
| [Contract] | [function()] | [var1, var2] | [e.g., "has inline msg.sender check" or "completely open"] |

If this table is empty, state so explicitly.

**Source:** `printers/function-summary.txt` + `printers/modifiers.txt` + `printers/require.txt`

## 5. Storage Layout Overview

Per-contract storage slot layout. Critical for proxy/upgradeable patterns and storage collision analysis.

| Contract | Slot | Variable | Type |
|----------|------|----------|------|
| [Vault] | [0] | [owner] | [address] |
| [Vault] | [1] | [totalAssets] | [uint256] |
| [Vault] | [3-49] | [__gap] | [uint256[47]] |

**Source:** `printers/variable-order.txt`

## 6. Printer Output Index

All raw printer files are in `audit-output/phase-1-recon/printers/`. Use this index when you need detail beyond the tables above.

| File | What to Find Here |
|------|-------------------|
| `human-summary.txt` | Contract count, SLOC, ERCs, complexity overview |
| `function-summary.txt` | **Anti-hallucination ground truth.** Per-function: visibility, modifiers, state vars read/written |
| `entry-points.txt` | All state-changing external/public functions and their accessed variables |
| `vars-and-auth.txt` | Which state variables each function modifies + authorization checks |
| `inheritance.txt` | Text-based inheritance relationships between contracts |
| `inheritance-graph*.dot` | DOT visualization of inheritance hierarchy |
| `variable-order.txt` | Storage slot layout — variable ordering per contract |
| `modifiers.txt` | Which modifiers are applied to each function |
| `require.txt` | All `require` and `assert` conditions per function |
| `not-pausable.txt` | Functions missing `whenNotPaused` guard |
| `call-graph*.dot` | DOT visualization of inter-function call relationships |
| `function-id.txt` | Keccak256 function selectors (check for collisions in proxy patterns) |

## 7. Notes

[Record anything unusual: errors, warnings, contracts that failed to analyze, printers that produced empty output, validation failures, coverage report highlights.]
```
