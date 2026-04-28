# Domain-Specific Playbooks

When Phase 0 Step 5 detects specific project types, activate the corresponding playbook(s) during Phase 4 Step 4. Each playbook is a separate file — **read only the ones that match the detected scenarios** to keep context focused.

## Scenario-to-Playbook Mapping

| Phase 0 Scenario Detected | Playbook File | Grep Signals |
|------------------------------|---------------|-------------|
| Proxy / Upgradeable | `playbooks/proxy-upgradeable.md` | `delegatecall`, `ERC1967`, `TransparentUpgradeableProxy`, `UUPSUpgradeable`, `BeaconProxy`, `initializer` |
| DeFi / Oracle → Lending | `playbooks/lending-borrowing.md` | `borrow`, `lend`, `liquidat`, `collateral`, `oracle`, `getPrice`, `latestRoundData` |
| DeFi / Oracle → AMM | `playbooks/amm-dex.md` | `swap`, `liquidity`, `addLiquidity`, `removeLiquidity`, `getAmountOut` |
| Token (ERC20/721/1155) | (use `resources/checklists/non-standard-tokens.md`) | `ERC20`, `ERC721`, `ERC1155`, `_mint`, `_burn`, `permit` |
| Vault / ERC4626 | `playbooks/vault-erc4626.md` | `ERC4626`, `deposit`, `withdraw`, `redeem`, `mint`, `convertToShares`, `convertToAssets`, `totalAssets` |
| Cross-chain / Bridge | `playbooks/other-playbooks.md` (Cross-Chain section) | `bridge`, `relayer`, `lzReceive`, `ccipReceive` |
| Staking / Restaking | `playbooks/other-playbooks.md` (Staking section) | `stake`, `unstake`, `restake`, `slash`, `operator`, `delegation`, `withdrawal` |
| Governance / Timelock | `playbooks/other-playbooks.md` (Governance section) | `propose`, `vote`, `execute`, `queue`, `timelock`, `quorum` |
| Perpetuals / Derivatives | `playbooks/other-playbooks.md` (Perpetuals section) | `openPosition`, `closePosition`, `liquidate`, `fundingRate`, `markPrice`, `margin` |
| Token Launch / Bonding Curve | `playbooks/other-playbooks.md` (Token Launch section) | `bondingCurve`, `LBP`, `launch`, `vesting`, `cliff`, `unlock` |
| Yield Aggregator (Multi-Strategy) | `playbooks/other-playbooks.md` (Yield Aggregator section) | `strategy`, `harvest`, `vault`, `compound`, `rebalance`, `emergencyWithdraw` |

**Multiple playbooks can apply.** A lending protocol with upgradeable proxies and oracle dependencies should activate `lending-borrowing.md` + `proxy-upgradeable.md` + any relevant token checklists.

## How to Use

1. **Phase 0 Step 5** detects scenarios via grep signals.
2. **Phase 4 Step 4** reads the matching playbook file(s) and applies each check to every relevant function.
3. For each playbook item that flags a potential issue, classify it per Phase 4 Step 5 triage.
