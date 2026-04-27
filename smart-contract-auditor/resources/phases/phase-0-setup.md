# Phase 0: Environment Setup & Scope Discovery

**Gate:** `forge build` succeeds AND `slither . --print human-summary` succeeds (or Slither fallback is documented).

---

## 0.0 Create Output Directory

```bash
mkdir -p audit-output/{phase-1-recon/printers,phase-2-docs,phase-3-scanning,phase-4-analysis,phase-5-findings,phase-6-report}
```

## 0.1 Detect Project Type

Check for `foundry.toml` (Foundry), `hardhat.config.js`/`hardhat.config.ts` (Hardhat), or raw `.sol` files.

## 0.2 Verify Compilation

Run `forge build`. See `tools/foundry.md` §3 for troubleshooting (solc mismatch, missing deps, remapping errors, stack too deep).

## 0.3 Verify Slither

Run `slither --version`. If Slither fails to analyze the project:

| Tier | When | Action |
|------|------|--------|
| 1: Fix & retry | Solc mismatch, missing remaps | `slither-doctor .`, adjust `--solc`, `--solc-remaps`, `--compile-force-framework foundry` |
| 2: Partial analysis | Some contracts fail | Use `--include-paths` to target compilable contracts, document scope limitation |
| 3: Foundry-only | Slither completely fails | Fall back to `forge inspect` for storage/selectors, manual code reading. Mark as degraded-confidence audit. |

## 0.4 Detect Audit Scenarios

Grep the codebase for patterns that activate conditional checks in later phases:
- **Proxy/Upgradeable:** `delegatecall`, `ERC1967`, `TransparentUpgradeableProxy`, `UUPSUpgradeable`, `BeaconProxy`, `initializer`
- **DeFi/Oracle:** `swap`, `liquidity`, `borrow`, `lend`, `oracle`, `getPrice`, `latestRoundData`, `latestAnswer`
- **Token:** `ERC20`, `ERC721`, `ERC1155`, `ERC777`, `_mint`, `_burn`, `permit`
- **Token interactions (arbitrary tokens):** `transferFrom`, `safeTransferFrom`, `IERC20`, `SafeERC20` — flag for non-standard token review using `checklists/non-standard-tokens.md`
- **Cross-chain:** `bridge`, `relayer`, `lzReceive`, `ccipReceive`
- **Staking/Restaking:** `stake`, `unstake`, `restake`, `slash`, `operator`, `delegation`, `withdrawal`
- **Transient storage:** `TSTORE`, `TLOAD`, `tstore`, `tload` — flag for EIP-1153 security review

## 0.5 Establish Slither Base Flags

Determine which flags to append to every `slither` command. For Foundry projects, the standard flags are:

```
--foundry-out-directory out --exclude-dependencies --filter-paths 'lib|node_modules|test|script'
```

**Important:** Do NOT store the full command in a shell variable and then execute the variable. Always write out the complete `slither` command directly each time. Shell variable expansion mangles multi-word arguments and pipe characters.

## 0.6 Select Submission Platform

Ask the user which platform. Default: Code4rena. Available templates:
- Code4rena: `templates/code4rena.md` (under `resources/templates/`)
- Sherlock: `templates/sherlock.md` (under `resources/templates/`)

If selected platform's template doesn't exist, inform the user and fall back to Code4rena format.

## 0.7 Contest Pre-Flight (Competitive Audits)

Before starting analysis:
1. **Read the contest README thoroughly** — known issues and out-of-scope items save you from wasted effort.
2. **Check for bot reports** (`bot-report.md`) — automated findings are already submitted, don't duplicate.
3. **Check for prior audits** — read any linked audit reports. Verify that reported issues were actually fixed. Look for patterns the codebase is prone to.
4. **Note the scope boundaries** — which contracts are in scope? Are there specific areas the sponsor wants auditors to focus on?
