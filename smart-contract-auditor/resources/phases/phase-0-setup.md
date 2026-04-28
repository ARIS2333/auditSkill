# Phase 0: Environment Setup & Scope Discovery

**Purpose:** Set up the audit environment, verify tooling, discover scope, and create the output directory.

**Gate:** `forge build` succeeds AND `slither . --print human-summary` succeeds (or Slither fallback is documented).

**Inputs:** Target codebase.

**Outputs:** `audit-output/` directory structure, detected scenarios, Slither base flags, scope assessment.

**Checkpoint:** Present scope assessment (contract count, SLOC estimate, detected scenarios, workflow intensity) to the user. Pause and wait for confirmation before proceeding to Phase 1.

---

## Step 1: Create Output Directory

```bash
mkdir -p audit-output/{phase-1-recon/printers/per-contract,phase-2-docs,phase-3-scanning,phase-4-analysis,phase-5-findings,phase-6-report}
```

### Output Directory Structure

All intermediate outputs, analysis notes, and deliverables go here — nothing gets lost between phases.

```
audit-output/
├── prior-audit-catalog.md        # (If prior audits exist) Per-finding catalog with original severity
├── phase-1-recon/
│   ├── printers/                  # One file per printer (12 total)
│   │   ├── human-summary.txt
│   │   ├── function-summary.txt
│   │   ├── entry-points.txt
│   │   ├── vars-and-auth.txt
│   │   ├── inheritance.txt
│   │   ├── inheritance-graph*.dot
│   │   ├── variable-order.txt
│   │   ├── modifiers.txt
│   │   ├── require.txt
│   │   ├── not-pausable.txt
│   │   ├── call-graph*.dot
│   │   ├── function-id.txt
│   │   └── per-contract/          # (Large codebases >15 contracts)
│   ├── structural-summary.md
│   ├── preliminary-hypotheses.md
│   └── coverage-report.txt        # (if tests exist)
├── phase-2-docs/
│   ├── codebase-overview.md
│   └── hypothesis-list.md
├── phase-3-scanning/
│   ├── slither-full-report.json
│   ├── slither-high-report.json
│   ├── slither-scenario.json
│   ├── slither-clean-report.json   # (optional)
│   └── scan-summary.md
├── phase-4-analysis/
│   ├── code-reading-notes.md
│   └── triage.md
├── phase-5-findings/
│   ├── H-01/
│   │   ├── H-01.t.sol
│   │   └── H-01.md
│   ├── M-01/
│   │   ├── M-01.t.sol
│   │   └── M-01.md
│   └── ...
└── phase-6-report/
    ├── final-report.md
    └── qa-report.md
```

**Rules:**
- Create `audit-output/` at project root.
- Each phase writes outputs to its corresponding folder.
- PoC test files in `phase-5-findings/` are also symlinked or copied to `test/audit/` so `forge test` can find them.
- Intermediate `.md` files capture your reasoning — they are working documents, not deliverables.

## Step 2: Detect Project Type

Check for `foundry.toml` (Foundry), `hardhat.config.js`/`hardhat.config.ts` (Hardhat), or raw `.sol` files.

## Step 3: Verify Compilation

Run `forge build`. See `resources/tools/foundry.md` (Build Troubleshooting section) for troubleshooting solc mismatch, missing deps, remapping errors, stack too deep.

## Step 4: Verify Slither

Run `slither --version`. If Slither fails to analyze the project:

| Tier | When | Action |
|------|------|--------|
| 1: Fix & retry | Solc mismatch, missing remaps | `slither-doctor .`, adjust `--solc`, `--solc-remaps`, `--compile-force-framework foundry` |
| 2: Partial analysis | Some contracts fail | Use `--include-paths` to target compilable contracts, document scope limitation |
| 3: Foundry-only | Slither completely fails | Fall back to `forge inspect` for storage/selectors, manual code reading. Mark as degraded-confidence audit. |

## Step 5: Detect Audit Scenarios

Grep the codebase for patterns that activate conditional checks in later phases:
- **Proxy/Upgradeable:** `delegatecall`, `ERC1967`, `TransparentUpgradeableProxy`, `UUPSUpgradeable`, `BeaconProxy`, `initializer`
- **DeFi/Oracle:** `swap`, `liquidity`, `borrow`, `lend`, `oracle`, `getPrice`, `latestRoundData`, `latestAnswer`
- **Token:** `ERC20`, `ERC721`, `ERC1155`, `ERC777`, `_mint`, `_burn`, `permit`
- **Token interactions (arbitrary tokens):** `transferFrom`, `safeTransferFrom`, `IERC20`, `SafeERC20` — flag for non-standard token review using `resources/checklists/non-standard-tokens.md`
- **Cross-chain:** `bridge`, `relayer`, `lzReceive`, `ccipReceive`
- **Staking/Restaking:** `stake`, `unstake`, `restake`, `slash`, `operator`, `delegation`, `withdrawal`
- **Transient storage:** `TSTORE`, `TLOAD`, `tstore`, `tload` — flag for EIP-1153 security review

## Step 6: Assess Scope Size & Set Workflow Intensity

Note the total SLOC and contract count from compiler output or `human-summary`. Use this to calibrate:

| Size | Criteria | Workflow Adjustment |
|------|----------|-------------------|
| **Small** | <500 SLOC, <5 contracts | Streamlined Phase 2: combine Contract Deep Dives and User Flows, skip State Machine if no discrete states, abbreviate architecture diagram |
| **Medium** | 500-2000 SLOC, 5-15 contracts | Full workflow as documented |
| **Large** | >2000 SLOC or >15 contracts | Prioritize high-value contracts (ETH/token handlers, access control, external calls). Split printer output into per-contract files. Process Phase 2 contract-by-contract. |

Record the size classification and any workflow adjustments in your notes.

## Step 7: Establish Slither Base Flags

Determine which flags to append to every `slither` command. For Foundry projects:

```
--foundry-out-directory out --exclude-dependencies --filter-paths 'lib|node_modules|test|script'
```

**Important:** Do NOT store the full command in a shell variable and then execute the variable. Always write out the complete `slither` command directly each time. Shell variable expansion mangles multi-word arguments and pipe characters.

## Step 8: Select Submission Platform

Ask the user which platform. Default: Code4rena. Available templates:
- Code4rena: `resources/templates/code4rena.md`
- Sherlock: `resources/templates/sherlock.md`

If the selected platform's template doesn't exist, inform the user and fall back to Code4rena format.

## Step 9: Contest Pre-Flight (Competitive Audits)

Before starting analysis:
1. **Read the contest README thoroughly** — known issues and out-of-scope items save wasted effort.
2. **Check for bot reports** (`bot-report.md`) — automated findings are already submitted.
3. **Check for prior audits** — read any linked audit reports. For each prior finding, record it in `audit-output/prior-audit-catalog.md` (see below). Verify reported issues were actually fixed.
4. **Note the scope boundaries** — which contracts are in scope? Are there specific areas the sponsor wants focus on?

### Prior Audit Catalog

When prior audit reports exist, create **`audit-output/prior-audit-catalog.md`**:

```markdown
# Prior Audit Catalog

| # | Source | Original ID | Title | Original Severity | Fixed? | Notes |
|---|--------|-------------|-------|-------------------|--------|-------|
| 1 | [Audit Firm / Report] | [H-01] | [Title] | [High/Medium/Low] | [Yes / No / Partial] | [Fix commit, or what remains unfixed] |
```

**Do NOT treat prior findings as automatically out of scope.** Known issues are out of scope only if your finding is the same severity and same root cause. A finding that shares the same root cause but demonstrates **higher severity** or a **substantively distinct attack path** is in scope.
