# Phase 3: Attack Planning

**Purpose:** Using only Phase 1 and Phase 2 outputs, plan attack assumptions by analyzing interaction workflows, money flows, access control patterns, and privilege structures. Do NOT read source code in this phase — reason from the documentation you have already built.

**Gate:** Attack plan covers all five sections (value flow, access control, domain-specific, privilege risk, optional Slither). Verification priority order produced.

**Inputs:**
- `audit-output/phase-1-recon/structural-summary.md` — structural reference (entry points, access control, unguarded functions, storage layout)
- `audit-output/phase-1-recon/coverage-report.txt` (if tests exist) — low-coverage areas are higher-priority targets
- `audit-output/phase-2-docs/codebase-overview.md` — your map (user flows, value flows, invariants, trust assumptions, access control matrix)
- Phase 0 detected audit scenarios
- Phase 0 Slither scanning decision (opt-in / opt-out)

**Outputs:** All output to `audit-output/phase-3-planning/`:
- `attack-plan.md` — prioritized attack vectors and privilege risk analysis
- `slither-full-report.json` (only if Slither scanning enabled)
- `slither-high-report.json` (only if Slither scanning enabled)
- `slither-low-info.md` (only if Slither scanning enabled — grouped Low/Informational findings)
- `slither-scenario.json` (only if Slither scanning enabled)

---

## Step 1: Analyze Value Flows for Attack Vectors

Read Phase 2 codebase overview sections: User Flows (§6), Value Flow (§8), Token Accounting (§8), Key Invariants (§12).

For each user flow that moves value (deposits, withdrawals, swaps, claims, liquidations, fee collection):

1. **Trace the money.** Where do tokens/ETH enter? Where do they exit? Who profits?
2. **Challenge every exchange rate.** Share price calculations, oracle-dependent valuations, fee computations — can any be manipulated?
3. **Check boundary conditions.** First depositor, empty vault/pool, zero amounts, maximum values, dust amounts.
4. **Check ordering assumptions.** Can calling functions in an unexpected order break invariants?
5. **Check flash loan amplification.** Can borrowed capital exploit any state within a single transaction?
6. **Check MEV exposure.** Are there sandwich-vulnerable swaps, front-runnable state changes, or missing commit-reveal patterns?

Record each potential vector in the attack plan template §1 (Value Flow Attack Vectors).

---

## Step 2: Analyze Access Control for Attack Vectors

Read Phase 1 structural summary sections: Entry Points & Access Control (§3), Unguarded State-Modifying Functions (§4). Read Phase 2 codebase overview: Access Control Matrix (§7).

1. **Start with unguarded functions.** Every entry in structural summary §4 is a top-priority target.
2. **Check every state-modifying entry point.** Does the modifier/require actually enforce the intended access control? (You will verify in Phase 4 — here you are flagging targets.)
3. **Check initializer safety.** Can `initialize()` be called more than once? Is it protected on the implementation contract?
4. **Check indirect call paths.** Can a public function call an internal function that bypasses access control?

Record each potential vector in the attack plan template §2 (Access Control Attack Vectors).

---

## Step 3: Apply Domain-Specific Playbooks and Checklists

Based on the audit scenarios detected in Phase 0 Step 5, read the matching playbook files from `resources/checklists/playbooks/` and the relevant checklists from `resources/checklists/`.

For each playbook/checklist item:
1. Check whether the pattern exists in the Phase 2 codebase documentation.
2. If the codebase documentation describes a mechanism that matches, record it as a potential vector.
3. Cross-reference with the adversarial framework (`resources/checklists/adversarial-framework.md`) for modern attack patterns (read-only reentrancy, donation attacks, permit front-running, etc.).

If the protocol interacts with arbitrary external tokens, consult `resources/checklists/non-standard-tokens.md` and flag any token behaviors the protocol does not appear to handle.

Record findings in the attack plan template §3 (Domain-Specific Attack Vectors).

---

## Step 4: Privilege & Management Risk Analysis

Read `resources/checklists/privilege-risk.md` for the full checklist. Use Phase 2 codebase overview: Access Control Matrix (§7), Known Risks & Trust Assumptions (§13).

1. **Enumerate every privileged role** from the access control matrix.
2. **Per-role power analysis:** For each role, list what functions it can call and what state/funds it can affect — direct fund access, indirect fund access (upgrade, oracle change, strategy change), DoS capability, role escalation.
3. **Key compromise scenarios:** For each role, describe the worst case if the private key is compromised. Assess: funds at risk, time to exploit (instant vs. timelocked), recovery paths.
4. **Mitigation assessment:** Check for timelocks, multisig, caps, pausability, two-step transfers, role separation.
5. **Role escalation paths:** Can any role grant itself additional privileges or bypass restrictions?

Record in the attack plan template §4 (Privilege & Management Risk Analysis).

---

## Step 5: Run Slither Vulnerability Scanning (Optional)

**This step runs ONLY if the user opted in to Slither scanning in Phase 0.**

If Slither scanning is enabled, run the scans described below. If not, skip to Step 6.

See `resources/tools/slither.md` for detector details, scan strategies, and known limitations.

### 5.1 Full Scan

```bash
slither . --foundry-out-directory out \
  --exclude-dependencies \
  --filter-paths 'lib|node_modules|test|script' \
  --json audit-output/phase-3-planning/slither-full-report.json
```

### 5.2 High-Impact Focused Scan

```bash
slither . --foundry-out-directory out \
  --exclude-dependencies \
  --filter-paths 'lib|node_modules|test|script' \
  --detect reentrancy-eth,reentrancy-no-eth,arbitrary-send-eth,arbitrary-send-erc20,arbitrary-send-erc20-permit,controlled-delegatecall,controlled-array-length,delegatecall-loop,suicidal,unprotected-upgrade,uninitialized-state,uninitialized-storage,shadowing-state,unchecked-transfer,weak-prng,msg-value-loop,incorrect-return,return-leave,incorrect-exp,storage-array \
  --json audit-output/phase-3-planning/slither-high-report.json
```

### 5.3 Scenario-Specific Scans

Based on Phase 0 Step 5 detection:

| Scenario Detected | Scan |
|-------------------|------|
| Proxy/Upgradeable | `slither-check-upgradeability . ImplementationContract > audit-output/phase-3-planning/upgradeability-check.txt 2>&1` |
| ERC20/721 | `slither-check-erc . ContractName > audit-output/phase-3-planning/erc-check.txt 2>&1` |
| DeFi/Oracle | Oracle-focused detectors: `pyth-deprecated-functions,pyth-unchecked-confidence,pyth-unchecked-publishtime,chronicle-unchecked-price,incorrect-equality,divide-before-multiply,weak-prng` — use `--json audit-output/phase-3-planning/slither-scenario.json` |
| Access Control | `suicidal,unprotected-upgrade,tx-origin,arbitrary-send-eth,arbitrary-send-erc20,controlled-delegatecall,protected-vars` — use `--json audit-output/phase-3-planning/slither-scenario.json` |

### 5.4 Process and Append Results

1. Parse all JSON output.
2. Add High/Medium findings to the attack plan template §5 (Slither Detector Findings). Cross-reference each finding against your existing attack vectors from §1-§4 — note which ones confirm your assumptions and which are new.
3. Group all Low and Informational findings into a separate file: **`audit-output/phase-3-planning/slither-low-info.md`** with sections for Low and Informational.

---

## Step 6: Produce Verification Priority Order

Consolidate all attack vectors from Steps 1-5 into a single prioritized list in the attack plan template §6 (Verification Priority Order).

### Priority criteria

| Priority | Criteria |
|----------|----------|
| **Critical** | Unguarded state-modifying functions, direct fund theft vectors, single EOA with no timelock controlling upgrades/funds |
| **High** | Value extraction via price/rate manipulation, flash loan amplified attacks, missing access control on fund-handling functions |
| **Medium** | DoS vectors, rounding/precision issues, domain-specific patterns that may or may not apply |
| **Low** | Well-guarded functions with potential edge cases, informational privilege observations |

### Verify Method

For each vector, specify the verification approach:
- **PoC** — for H/M code vulnerabilities: read code, confirm the bug, write Foundry PoC
- **Write-up only** — for privilege/management risks: write finding report without PoC
- **Code review** — for potential vectors that need deeper code reading to confirm or deny

Functions with zero or low test coverage (from `coverage-report.txt`) should be weighted higher in priority.
