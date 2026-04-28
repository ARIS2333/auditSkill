# Phase 5: PoC & Finding Documentation

**Purpose:** For each confirmed/potential finding, write the PoC AND the finding report together while context is fresh. Do not split these into separate steps.

**Gate:** Every confirmed High/Medium has a passing PoC and a complete finding write-up.

**Inputs:**
- `audit-output/phase-4-analysis/triage.md` (confirmed and potential findings)
- `audit-output/phase-2-docs/codebase-overview.md` (protocol context for realistic PoC setup)
- Platform template: `resources/templates/code4rena.md` or `resources/templates/sherlock.md`

**Outputs:** Per finding in `audit-output/phase-5-findings/`:
- `H-01/H-01.t.sol` — Foundry PoC
- `H-01/H-01.md` — Finding write-up

**Reference:** Use the Phase 2 codebase overview for protocol context — user flows help set up realistic exploit scenarios, value flow clarifies where funds move, invariants clarify what "success" means for the exploit.

**Resources:**
- `resources/tools/foundry.md` (Cheatcodes and PoC Workflow sections)
- `resources/templates/poc.md` for starter templates
- `resources/tools/slither.md` (Detector-to-PoC Mapping section)

---

## Priority

- All confirmed High: **must** have PoC
- All confirmed Medium: **must** have PoC
- Potential High: attempt PoC to confirm or deny

---

## Per-Finding Workflow

For each finding, create a subfolder (e.g., `audit-output/phase-5-findings/H-01/`) and complete all steps before moving to the next finding:

### Step 1: Write the PoC

- Write the PoC test file to `audit-output/phase-5-findings/H-01/H-01.t.sol`
- Also place it (or symlink it) under `test/audit/` so `forge test` can find it
- Non-Foundry projects: create minimal Foundry project, copy targets, configure remappings
- Deployed contracts: configure fork URL and block number
- Every PoC must: (1) record pre-exploit state, (2) execute the exploit, (3) assert exploit success with meaningful failure messages
- Use `vm.label` for readable traces

### Step 2: Run the PoC and Capture Output

- Run: `forge test --match-test test_H01_Exploit -vv`
- Capture the **actual test output** (pass/fail, gas, assertion results)
- If the PoC fails, debug with `-vvvv` or `-vvvvv` traces before proceeding
- **Never fabricate test output.** If you cannot run the test, say so.

### Step 3: Write the Finding Report

- Write to `audit-output/phase-5-findings/H-01/H-01.md`
- Load the platform template from `resources/templates/`
- The PoC section must include all three parts:
  1. **Complete, runnable Solidity code** — the full test file, not a snippet
  2. **Run command and actual output** — the exact `forge test` command and real output from Step 2
  3. **"The output confirms" section** — bullet points explaining what each passing assertion proves
- Write root cause, impact, and mitigation while the exploit logic is fresh
- Link to exact line numbers in the codebase

### Step 4: Verify

- PoC compiles and passes
- Finding report is complete and severity is accurate
- Reported test output matches the actual run
- If PoC fails: analyze the `-vvvv` trace, diagnose (wrong setUp state, wrong fork block, missed access control, gas limits). If genuinely irreproducible, reclassify as **Potential** with explanation.

---

## Additional Verification

When time permits:
- **Fuzz testing** for protocol invariants (`testFuzz_` pattern, `bound()` for input ranges)
- **Stateful invariant testing** for complex protocols (`invariant_` pattern with handler contracts) — see `resources/tools/foundry.md` (Invariant Testing section)
- **Gas analysis** (`forge test --gas-report`) for DoS vectors — functions exceeding block gas limit
