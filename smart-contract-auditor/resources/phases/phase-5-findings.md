# Phase 5: PoC & Finding Documentation

**Purpose:** For each confirmed/potential finding, write the PoC AND the finding report together while context is fresh. Do not split these into separate steps.

**Gate:** Every confirmed High/Medium has a passing PoC and a complete finding write-up.

---

## Resources

- `tools/foundry.md` §6 for cheatcodes, §7 for helpers/assertions, §12-13 for PoC workflow and template, §14 for invariant testing
- `templates/poc.md` for starter templates (ETH drain, ERC20 drain, access control bypass, flash loan, oracle manipulation)
- `tools/slither.md` §10 for detector-to-PoC strategies

## Inputs

- **Phase 4:** `audit-output/phase-4-analysis/triage.md` (confirmed and potential findings)
- **Phase 2:** `audit-output/phase-2-docs/codebase-overview.md` (protocol context for realistic PoC setup)
- **Platform template:** `templates/code4rena.md` or `templates/sherlock.md`

## Reference

Use `audit-output/phase-2-docs/codebase-overview.md` to understand the broader protocol context when writing PoCs — the user flows help you set up realistic exploit scenarios, the value flow section clarifies where funds move, and the invariants section clarifies what "success" means for the exploit.

---

## Priority

- All confirmed High: **must** have PoC
- All confirmed Medium: **must** have PoC
- Potential High: attempt PoC to confirm or deny

---

## Per-Finding Workflow

For each finding, create a subfolder (e.g., `audit-output/phase-5-findings/H-01/`) and complete all three steps before moving to the next finding:

### Step 1: Write the PoC

- Write the PoC test file to `audit-output/phase-5-findings/H-01/H-01.t.sol`
- Also place it (or symlink it) under `test/audit/` so `forge test` can find it
- Non-Foundry projects: create minimal Foundry project, copy targets, configure remappings
- Deployed contracts: configure fork URL and block number
- Every PoC must: (1) record pre-exploit state, (2) execute the exploit, (3) assert exploit success with meaningful failure messages
- Use `vm.label` for readable traces. Run with `-vvvv`.

### Step 2: Write the finding report

- Write to `audit-output/phase-5-findings/H-01/H-01.md`
- Load the platform template from `templates/`
- Write root cause, impact, and mitigation while the exploit logic is fresh in context
- Link to exact line numbers in the codebase

### Step 3: Verify

- PoC compiles and passes
- Finding report is complete and severity is accurate
- If PoC fails: analyze the `-vvvv` trace, diagnose (wrong setUp state, wrong fork block, missed access control, gas limits). If genuinely irreproducible, reclassify as **Potential** with explanation.

---

## Additional Verification

When time permits, go beyond individual PoCs:
- **Fuzz testing** for protocol invariants (`testFuzz_` pattern, `bound()` for input ranges)
- **Stateful invariant testing** for complex protocols (`invariant_` pattern with handler contracts) — see `tools/foundry.md` §14 for the complete invariant testing reference, handler pattern, configuration, and common DeFi invariants
- **Gas analysis** (`forge test --gas-report`) for DoS vectors — functions exceeding block gas limit
