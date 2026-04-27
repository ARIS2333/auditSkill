# Domain-Specific Playbooks

When Phase 0.4 detects specific project types, activate the corresponding playbook(s) during Phase 4 code reading. Each playbook is a separate file — **read only the ones that match the detected scenarios** to keep context focused.

## Scenario-to-Playbook Mapping

| Phase 0.4 Scenario Detected | Playbook File | Grep Signals |
|------------------------------|---------------|-------------|
| Proxy / Upgradeable | `playbooks/proxy-upgradeable.md` | `delegatecall`, `ERC1967`, `TransparentUpgradeableProxy`, `UUPSUpgradeable`, `BeaconProxy`, `initializer` |
| DeFi / Oracle → Lending | `playbooks/lending-borrowing.md` | `borrow`, `lend`, `liquidat`, `collateral`, `oracle`, `getPrice`, `latestRoundData` |
| DeFi / Oracle → AMM | `playbooks/amm-dex.md` | `swap`, `liquidity`, `addLiquidity`, `removeLiquidity`, `getAmountOut` |
| Token (ERC20/721/1155) | (use `checklists/non-standard-tokens.md`) | `ERC20`, `ERC721`, `ERC1155`, `_mint`, `_burn`, `permit` |
| Vault / ERC4626 | `playbooks/vault-erc4626.md` | `ERC4626`, `deposit`, `withdraw`, `redeem`, `mint`, `convertToShares`, `convertToAssets`, `totalAssets` |
| Cross-chain / Bridge | `playbooks/cross-chain-bridge.md` | `bridge`, `relayer`, `lzReceive`, `ccipReceive` |
| Staking / Restaking | `playbooks/staking-restaking.md` | `stake`, `unstake`, `restake`, `slash`, `operator`, `delegation`, `withdrawal` |
| Governance / Timelock | `playbooks/governance-timelock.md` | `propose`, `vote`, `execute`, `queue`, `timelock`, `quorum`, `GovernorBravo` |
| Perpetuals / Derivatives | `playbooks/perpetuals-derivatives.md` | `openPosition`, `closePosition`, `liquidate`, `fundingRate`, `markPrice`, `indexPrice`, `margin` |
| Token Launch / Bonding Curve | `playbooks/token-launch-bonding-curve.md` | `bondingCurve`, `LBP`, `launch`, `vesting`, `cliff`, `unlock` |
| Yield Aggregator (Multi-Strategy) | `playbooks/yield-aggregator.md` | `strategy`, `harvest`, `vault`, `compound`, `rebalance`, `emergencyWithdraw` |

**Multiple playbooks can apply.** A lending protocol with upgradeable proxies and oracle dependencies should activate `lending-borrowing.md` + `proxy-upgradeable.md` + any relevant token checklists.

## How to Use

1. **Phase 0.4** detects scenarios via grep signals (right column above)
2. **Phase 4.5** reads the matching playbook file(s) and applies each check to every relevant function
3. For each playbook item that flags a potential issue, classify it per the Phase 4.6 triage framework
