# Phase 4: Verification & PoC

**Purpose:** Verify each attack vector from the Phase 3 attack plan by diving deep into the source code and writing Foundry PoCs. This phase merges targeted code reading with PoC development — you read the code, confirm (or deny) the vulnerability, and write the PoC while context is fresh.

**Gate:** Every Critical/High attack vector verified. Every confirmed H/M has a passing PoC and complete finding write-up. Every privilege/management risk has a finding write-up (no PoC required). All False Positives justified with specific code references.

**Inputs:**
- `audit-output/phase-3-planning/attack-plan.md` (prioritized hit list)
- `audit-output/phase-2-docs/codebase-overview.md` (your map — user flows, value flows, invariants, access control)
- `audit-output/phase-1-recon/structural-summary.md` (structural reference — function signatures, modifiers, state writes)
- Platform template: `resources/templates/code4rena.md` or `resources/templates/sherlock.md`

**Outputs:** Per finding in `audit-output/phase-4-findings/`:
- `H-01/H-01.t.sol` — Foundry PoC (H/M code vulnerabilities only)
- `H-01/H-01.md` — Finding write-up
- `C-01/C-01.md` — Privilege/management risk write-up (no PoC)

**Resources:**
- `resources/tools/foundry.md` (Cheatcodes and PoC Workflow sections)
- `resources/templates/poc.md` for starter templates
- `resources/checklists/code-reading-checklist.md` for systematic code review
- `resources/checklists/adversarial-framework.md` for attack thinking
- `resources/tools/slither.md` (Detector-to-PoC Mapping section, if Slither was used)

---

## Step 1: Determine Verification Order

Work through the Phase 3 attack plan §6 (Verification Priority Order) from highest to lowest rank. Read base contracts before derived contracts (from the inheritance tree in the structural summary §2).

---

## Step 2: Per-Vector Verification Workflow

For each attack vector, follow this workflow:

### 2.1 Read the Relevant Code

1. **Open the structural summary** — read the target function's entry in §3 (Entry Points & Access Control). Note visibility, modifiers, state variables written.
2. **Read the source code** for the target function and all functions it calls internally.
3. **Apply the code reading checklist** (`resources/checklists/code-reading-checklist.md`) — work through every item systematically for the target function.
4. **Apply adversarial thinking** (`resources/checklists/adversarial-framework.md`) — try to break the invariants identified in Phase 2.

### 2.2 Classify the Finding

| Classification | Criteria | Action |
|----------------|----------|--------|
| **Confirmed** | Real, reachable, impact clear | Write PoC + finding report (Step 3) |
| **Potential** | May be real, needs more context | Attempt PoC to confirm or deny |
| **Prior-Audit-Overlap** | Overlaps with `audit-output/prior-audit-catalog.md` | Evaluate severity — same root cause + same severity = out of scope; higher severity or distinct attack path = promote |
| **False Positive** | Not exploitable | Justify with specific code reference and which checklist items ruled it out |

### 2.3 Pattern Propagation

**When you confirm a vulnerability pattern, immediately search for the same pattern across ALL in-scope contracts.** Vulnerability patterns rarely occur in isolation.

- Found a `balanceOf(address(this))` donation issue? → `grep -rn 'balanceOf' src/` and review every hit.
- Found a rounding issue in deposit math? → Check every function that reads the rounded value.
- Found missing access control on one setter? → Check all setters in the same and sibling contracts.

Add newly discovered vectors back to the attack plan and verify them.

---

## Step 3: Write PoC + Finding Report (Code Vulnerabilities)

For each confirmed or potential H/M code vulnerability, create a subfolder (e.g., `audit-output/phase-4-findings/H-01/`) and complete all steps before moving to the next finding:

### 3.1 Write the PoC

- Write the PoC test file to `audit-output/phase-4-findings/H-01/H-01.t.sol`
- Also place it (or symlink it) under `test/audit/` so `forge test` can find it
- Non-Foundry projects: create minimal Foundry project, copy targets, configure remappings
- Deployed contracts: configure fork URL and block number
- Every PoC must: (1) record pre-exploit state, (2) execute the exploit, (3) assert exploit success with meaningful failure messages
- Use `vm.label` for readable traces

See `resources/templates/poc.md` for starter templates. See `resources/tools/foundry.md` (Cheatcodes section) for cheatcode signatures.

### 3.2 Run the PoC and Capture Output

- Run: `forge test --match-test test_H01_Exploit -vv`
- Capture the **actual test output** (pass/fail, gas, assertion results)
- If the PoC fails, debug with `-vvvv` or `-vvvvv` traces before proceeding
- **Never fabricate test output.** If you cannot run the test, say so.

### 3.3 Write the Finding Report

- Write to `audit-output/phase-4-findings/H-01/H-01.md`
- Load the platform template from `resources/templates/`
- The PoC section must include all three parts:
  1. **Complete, runnable Solidity code** — the full test file, not a snippet
  2. **Run command and actual output** — the exact `forge test` command and real output from Step 3.2
  3. **"The output confirms" section** — bullet points explaining what each passing assertion proves
- Write root cause, impact, and mitigation while the exploit logic is fresh
- Link to exact line numbers in the codebase

### 3.4 Verify

- PoC compiles and passes
- Finding report is complete and severity is accurate
- Reported test output matches the actual run
- If PoC fails: analyze the `-vvvv` trace, diagnose (wrong setUp state, wrong fork block, missed access control, gas limits). If genuinely irreproducible, reclassify as **Potential** with explanation.

---

## Step 4: Write Finding Report (Privilege/Management Risks)

For each confirmed privilege/management risk from the attack plan §4, create a subfolder (e.g., `audit-output/phase-4-findings/C-01/`) and write a finding report:

- Write to `audit-output/phase-4-findings/C-01/C-01.md`
- **No PoC required** — these are design/architecture observations, not code bugs
- Include: the role, what it can do, worst-case scenario, existing mitigations (or lack thereof), recommended mitigation
- These are typically Centralization findings (C-XX in Code4rena QA) or trust assumption notes (Sherlock)

---

## Step 5: Anti-Hallucination Enforcement

Every claim about a function's behavior must be verified against `function-summary` in the structural summary. If your reading disagrees with the printer:

1. The printer output is correct (derived from the AST).
2. Re-read. Common causes: wrong contract (override), local vs state variable confusion, inheritance chain.
3. Only override the printer if you can point to the exact line AND explain why (e.g., inline assembly, delegatecall).

---

## Step 6: Update Phase 2 Document

If code reading reveals information that the Phase 2 codebase document is missing or got wrong — new user flows, incorrect access control claims, additional invariants, undocumented trust assumptions — update `audit-output/phase-2-docs/codebase-overview.md` now.

---

## Step 7: Additional Verification (When Time Permits)

- **Fuzz testing** for protocol invariants (`testFuzz_` pattern, `bound()` for input ranges)
- **Stateful invariant testing** for complex protocols (`invariant_` pattern with handler contracts) — see `resources/tools/foundry.md` (Invariant Testing section)
- **Gas analysis** (`forge test --gas-report`) for DoS vectors — functions exceeding block gas limit
