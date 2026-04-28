# Phase 3: Automated Scanning

**Purpose:** Run Slither detectors against a codebase you already deeply understand from Phase 2. Because you have the codebase documentation, you can immediately contextualize every detector finding.

**Gate:** Full scan JSON output available and parsed. Findings categorized by severity and contextualized against Phase 2 documentation.

**Inputs:**
- `audit-output/phase-2-docs/codebase-overview.md`
- `audit-output/phase-2-docs/hypothesis-list.md`
- Phase 0 detected audit scenarios and Slither base flags

**Outputs:** All JSON/text output to `audit-output/phase-3-scanning/`:
- `slither-full-report.json`
- `slither-high-report.json`
- `slither-scenario.json` (and/or `upgradeability-check.txt`, `erc-check.txt`)
- `slither-clean-report.json` (optional)
- `scan-summary.md`

See `resources/tools/slither.md` (Vulnerability Detection section) for all 99 detectors, (Targeted Scan Strategies section) for scan strategies, (Known Limitations section) for blind spots.

---

## Step 1: Run Full Scan

All detectors, exclude dependencies and test/script paths:

```bash
slither . --foundry-out-directory out \
  --exclude-dependencies \
  --filter-paths 'lib|node_modules|test|script' \
  --json audit-output/phase-3-scanning/slither-full-report.json
```

## Step 2: Run High-Impact Focused Scan

```bash
slither . --foundry-out-directory out \
  --exclude-dependencies \
  --filter-paths 'lib|node_modules|test|script' \
  --detect reentrancy-eth,reentrancy-no-eth,arbitrary-send-eth,arbitrary-send-erc20,arbitrary-send-erc20-permit,controlled-delegatecall,controlled-array-length,delegatecall-loop,suicidal,unprotected-upgrade,uninitialized-state,uninitialized-storage,shadowing-state,unchecked-transfer,weak-prng,msg-value-loop,incorrect-return,return-leave,incorrect-exp,storage-array \
  --json audit-output/phase-3-scanning/slither-high-report.json
```

## Step 3: Run Scenario-Specific Scans

Based on Phase 0 Step 5 detection:

| Scenario Detected | Scan |
|-------------------|------|
| Proxy/Upgradeable | `slither-check-upgradeability . ImplementationContract > audit-output/phase-3-scanning/upgradeability-check.txt 2>&1` |
| ERC20/721 | `slither-check-erc . ContractName > audit-output/phase-3-scanning/erc-check.txt 2>&1` |
| DeFi/Oracle | Oracle-focused detectors: `pyth-deprecated-functions,pyth-unchecked-confidence,pyth-unchecked-publishtime,chronicle-unchecked-price,incorrect-equality,divide-before-multiply,weak-prng` — use `--json audit-output/phase-3-scanning/slither-scenario.json` |
| Access Control | `suicidal,unprotected-upgrade,tx-origin,arbitrary-send-eth,arbitrary-send-erc20,controlled-delegatecall,protected-vars` — use `--json audit-output/phase-3-scanning/slither-scenario.json` |

**Note:** `slither-check-upgradeability` and `slither-check-erc` are standalone tools that output text to stdout. They do not support `--json`. Redirect to `.txt` files as shown.

## Step 4: Noise Reduction (if needed)

```bash
slither . --foundry-out-directory out \
  --exclude-informational --exclude-optimization \
  --exclude-dependencies \
  --filter-paths 'lib|node_modules|test|script' \
  --json audit-output/phase-3-scanning/slither-clean-report.json
```

---

## Step 5: Prepare Scan Summary

Write **`audit-output/phase-3-scanning/scan-summary.md`**:

1. Categorize detector findings by severity.
2. Overlay them onto the Phase 2 hypothesis list — do any detectors confirm your hypotheses? Do any flag functions you ranked "Low"?
3. For each finding, reference the relevant section of the Phase 2 codebase documentation — the architecture diagram, access control matrix, or user flow that provides context for whether the finding is real.
4. Note which findings need code reading to confirm vs. which are clearly false positives from context you already have.
