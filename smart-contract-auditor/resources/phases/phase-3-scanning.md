# Phase 3: Automated Scanning

**Purpose:** Run Slither detectors against a codebase you already deeply understand from Phase 2. Because you have the codebase documentation in hand, you can immediately contextualize every detector finding — distinguishing real bugs from false positives with confidence.

**Gate:** Full scan JSON output available and parsed. Findings categorized by severity and contextualized against the Phase 2 codebase documentation.

See `tools/slither.md` §4 for all 103 detectors, §6 for targeted scan strategies, §16 for known limitations (what Slither cannot catch).

---

## Inputs

- **Phase 2 output:** `audit-output/phase-2-docs/codebase-overview.md`
- **Phase 1 output:** `audit-output/phase-1-recon/hypothesis-list.md`
- **Phase 0 output:** Detected audit scenarios, Slither base flags

## Outputs

Direct all JSON output to `audit-output/phase-3-scanning/`.

---

## Scans to Run

### 1. Full Scan

All detectors, exclude dependencies and test/script paths:

```bash
slither . --foundry-out-directory out \
  --exclude-dependencies \
  --filter-paths "lib|node_modules|test|script" \
  --json audit-output/phase-3-scanning/slither-full-report.json
```

### 2. High-Impact Focused Scan

Reentrancy, arbitrary-send, delegatecall, suicidal, unprotected-upgrade, uninitialized, unchecked-transfer, shadowing-state, weak-prng, msg-value-loop:

```bash
slither . --foundry-out-directory out \
  --detect reentrancy-eth,reentrancy-balance,reentrancy-no-eth,token-reentrancy,arbitrary-send-eth,arbitrary-send-erc20,arbitrary-send-erc20-permit,controlled-delegatecall,controlled-array-length,delegatecall-loop,suicidal,unprotected-upgrade,uninitialized-state,uninitialized-storage,shadowing-state,unchecked-transfer,weak-prng,msg-value-loop,msg-value-in-nonpayable \
  --json audit-output/phase-3-scanning/slither-high-report.json
```

### 3. Scenario-Specific Scans

Based on Phase 0.4 detection, save to `audit-output/phase-3-scanning/slither-scenario.json`:

| Scenario Detected | Scan |
|-------------------|------|
| Proxy/Upgradeable | `slither-check-upgradeability . ImplementationContract` |
| ERC20/721 | `slither-check-erc . ContractName` |
| DeFi/Oracle | Oracle-focused detectors: `pyth-deprecated-functions,pyth-unchecked-confidence,pyth-unchecked-publishtime,chronicle-unchecked-price,incorrect-equality,divide-before-multiply,weak-prng` |
| Access Control | `suicidal,unprotected-upgrade,tx-origin,arbitrary-send-eth,arbitrary-send-erc20,controlled-delegatecall,protected-vars` |

### 4. Noise Reduction (if needed)

Exclude informational/optimization, or exclude specific low-value detectors:

```bash
slither . --foundry-out-directory out \
  --exclude-informational --exclude-optimization \
  --exclude-dependencies \
  --filter-paths "lib|node_modules|test|script" \
  --json audit-output/phase-3-scanning/slither-clean-report.json
```

---

## Prepare Scan Summary

Write **`audit-output/phase-3-scanning/scan-summary.md`**:

1. Categorize detector findings by severity
2. Overlay them onto the Phase 1 hypothesis list — do any detectors confirm your hypotheses? Do any flag functions you ranked "Low"?
3. For each finding, reference the relevant section of the Phase 2 codebase documentation — the architecture diagram, access control matrix, or user flow that provides context for whether the finding is real
4. Note which findings need code reading to confirm vs. which are clearly false positives from context you already have
