# Phase 4: Targeted Code Reading & Triage

**Purpose:** Deep security-focused code reading guided by Phase 1 (structural model), Phase 2 (codebase documentation), and Phase 3 (detector findings). Classify findings as you go.

**Gate:** All critical and high-target functions read. Every High and Medium finding (both automated and manual) classified with written justification.

---

## Inputs

- **Phase 2:** `audit-output/phase-2-docs/hypothesis-list.md` (ranked targets)
- **Phase 2:** `audit-output/phase-2-docs/codebase-overview.md` (your map)
- **Phase 3:** `audit-output/phase-3-scanning/scan-summary.md` (detector findings)

## Outputs

- **`audit-output/phase-4-analysis/code-reading-notes.md`** — observations per function, cross-referenced against printers
- **`audit-output/phase-4-analysis/triage.md`** — every finding classified (confirmed/potential/false positive) with justification

## Reference

Keep `audit-output/phase-2-docs/codebase-overview.md` open as your map. The architecture diagram tells you where to look, the user flows tell you what to trace, the access control matrix tells you what to verify, and the key invariants tell you what to try to break.

---

## 4.1 Reading Order

From the inheritance graph, determine dependency order: **read base contracts before derived contracts.** Within that order, prioritize by the Phase 2 hypothesis ranking, weighted by Phase 3 detector findings.

## 4.2 Read Base Contracts First

Verify that modifier behavior matches the name — a modifier called `onlyOwner` that doesn't check ownership is a critical finding.

## 4.3 Code Reading Checklist

For each function flagged in Phase 1:

1. **Verify Phase 1 data** — does the function actually write the variables listed in `function-summary`?
2. **Inline access control** — `require(msg.sender == owner)` won't show in `modifiers` printer
3. **External calls** — all `.call`, `.delegatecall`, `.transfer`, `.send`, interface calls
4. **Reentrancy** — external call before state update? Cross-function via shared state? Read-only reentrancy via external view calls?
5. **Access control** — can a non-privileged user reach this path?
6. **Integer overflow/underflow** — `unchecked` blocks, type casting (`uint256` -> `uint128`)
7. **Precision loss** — division before multiplication, rounding direction, dust amounts, rounding accumulation over many operations
8. **Front-running / MEV** — sandwich potential? Missing commit-reveal?
9. **Oracle manipulation** — price feeds manipulable within a single tx (flash loans)?
10. **Flash loan vectors** — can borrowed capital exploit state in one tx?
11. **DoS vectors** — unbounded loops over user-controlled arrays, grief via revert callbacks, gas griefing via return data bombs
12. **Timestamp dependence** — `block.timestamp` in critical logic?
13. **`tx.origin` vs `msg.sender`** — `tx.origin` used for authorization?
14. **`delegatecall` correctness** — storage layout alignment, context preservation
15. **ERC compliance** — return values, zero/self-transfer edge cases, event emissions
16. **Boundary values** — zero amounts, `type(uint256).max`, empty arrays, first depositor / empty vault attacks
17. **Return value handling** — unchecked low-level calls, `try`/`catch` that silently swallows errors, missing success check on `.call`
18. **Signature malleability** — `ecrecover` without `s`-value normalization (use OpenZeppelin `ECDSA`), missing `address(0)` check on recovered signer, cross-chain replay (missing chain ID in domain separator)
19. **Multi-call / batch atomicity** — can combining multiple operations in one tx break invariants that hold for individual calls? Permit + transferFrom front-running?
20. **Initializer safety** — `initialize()` callable multiple times? Missing `initializer` modifier? Missing `_disableInitializers()` in constructor of implementation contract?
21. **Event emission correctness** — wrong parameter values in events, missing events for critical state changes (ownership transfer, parameter updates), events emitted before state finalized
22. **Non-standard token behavior** — if the function handles arbitrary ERC20 tokens, check against `checklists/non-standard-tokens.md` (fee-on-transfer, rebasing, missing return values, pausable, low decimals, callbacks)
23. **Donation / direct transfer attacks** — does the contract use `balanceOf(address(this))` for accounting instead of internal bookkeeping? Can direct token transfers manipulate share prices or invariants?
24. **Transient storage (EIP-1153)** — if `TSTORE`/`TLOAD` used: reentrancy guards relying on transient storage, end-of-tx cleanup assumptions, cross-call data passing assumptions

---

## 4.4 Adversarial Thinking & Logic Bugs

Static analysis cannot find logic bugs. Use the Phase 2 documentation — especially the **Key Invariants** and **User Flows** sections — as your target list. For every invariant, ask: "Can I break this?"

**Think like an attacker:**
- For every user flow in the Phase 2 doc, **follow the money**: "How would I extract more value than I put in?"
- For every access control entry in the Phase 2 access control matrix: "Can I reach this path without the expected privilege?"
- For every external dependency listed in Phase 2: "What if the external contract behaves maliciously?"

**Logic bugs static analysis misses:**
- Business logic errors (wrong fee calc, wrong rounding direction, off-by-one)
- TOCTOU patterns
- Incorrect assumptions about external contracts (fee-on-transfer tokens, rebasing tokens, tokens with callbacks) — see `checklists/non-standard-tokens.md`
- Cross-function invariant violations (consistent within one function, broken across a call sequence)
- Economic exploits (first depositor inflation, sandwich attacks on AMM interactions)
- Governance/multi-step attack vectors

**Modern attack patterns (often missed by static analysis):**
- **Read-only reentrancy** — Protocol A reads state from Protocol B (e.g., LP token price) while Protocol B is mid-execution. The read returns stale/manipulated values because Protocol B hasn't finalized its state update yet. Common in lending protocols that price LP tokens by querying pool reserves. Slither's reentrancy detectors do NOT catch this because the reentrant call is a `view` function.
- **Donation attacks beyond ERC4626** — any contract using `balanceOf(address(this))` or `address(this).balance` for accounting can be manipulated via direct token transfer or `selfdestruct` (pre-Dencun) ETH injection. Check every place the contract reads its own balance.
- **Permit front-running** — attacker front-runs a `permit` + `transferFrom` combo by calling `permit` with the victim's signature first. Victim's tx reverts on the duplicate `permit` nonce. Safe pattern: wrap `permit` in try/catch.
- **Signature replay across chains** — EIP-712 signatures missing chain ID in domain separator are replayable on any chain where the contract is deployed at the same address.
- **CREATE2 address collision** — if a protocol uses `CREATE2` to deploy contracts, an attacker who controls the salt may be able to predict and front-run the address. For `create` + `selfdestruct` + `create2` patterns, an attacker can deploy a different contract at the same address.
- **Transient storage assumptions** — reentrancy guards using `TSTORE`/`TLOAD` (EIP-1153) clear at end of transaction, but nested calls within the same tx share transient storage. If a guard is set in a `TSTORE` slot, a callback from an external contract (same tx) may bypass the guard if using a different slot or if the check is order-dependent.

---

## 4.5 Domain-Specific Checks

When Phase 0.4 detected specific project types, activate the corresponding playbook from `checklists/domain-playbooks.md` during this phase. Read the relevant playbook section and apply its checks to every matching function.

---

## 4.6 Triage As You Go

For each finding — whether from a Phase 3 detector or from manual code reading — classify it immediately while the context is fresh:

| Classification | Criteria | Action |
|----------------|----------|--------|
| **Confirmed** | Real, reachable, impact clear | Queue for Phase 5 (PoC + write-up) |
| **Potential** | May be real, needs more context | Note what's needed, attempt PoC in Phase 5 |
| **False Positive** | Incorrect due to context detector can't see | Justify with specific code reference, move on |

For each Slither finding specifically:
1. Does it match something you observed during code reading?
2. Does Phase 1 structural data support it?
3. Is the code path actually reachable? (Check `call-graph`)

---

## 4.7 Anti-Hallucination Enforcement

Every claim about a function's behavior must be verified against `function-summary`. If your reading disagrees with the printer:
1. The printer output is correct (derived from the AST).
2. Re-read. Common causes: wrong contract (override), local vs state variable confusion, inheritance chain.
3. Only override the printer if you can point to the exact line AND explain why (e.g., inline assembly, delegatecall).

---

## 4.8 Verification via Foundry

Use Foundry as a **comprehension tool** during code reading:
- **Trace existing tests** (`forge test -vvvv`) to understand multi-contract flows
- **Run existing test suite** (`forge test -vvv`) — what did devs test? What did they NOT test?
- **Check coverage** (`forge coverage`) — focus manual review on uncovered critical paths

---

## 4.9 Update Phase 2 Document

If code reading reveals information that the Phase 2 codebase document is missing or got wrong — new user flows, incorrect access control claims, additional invariants, undocumented trust assumptions — update `audit-output/phase-2-docs/codebase-overview.md` now. Apply the same fact-checking standard.
