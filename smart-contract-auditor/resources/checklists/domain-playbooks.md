# Domain-Specific Playbooks

When Phase 0.4 detects specific project types, activate these focused checks during Phase 4 code reading. Each playbook lists the high-value attack vectors for that protocol type.

---

## Vault / ERC4626

- First depositor inflation attack (donate to inflate share price, victim gets 0 shares)
- Share price manipulation via direct token transfer (donation attack)
- Rounding direction: deposits should round DOWN (fewer shares), withdrawals should round DOWN (fewer assets) — always favor the vault
- `deposit` vs `mint` vs `redeem` vs `withdraw` — all four must be consistent
- Empty vault edge case (totalSupply == 0)

## AMM / DEX

- Sandwich attack on swaps (front-run + back-run)
- Price manipulation via flash loans within same tx
- Slippage protection: is `minAmountOut` enforced? Can it be set to 0?
- Fee-on-transfer token compatibility
- Reentrancy via token callbacks (ERC777, ERC1155)

## Lending / Borrowing

- Oracle staleness and manipulation
- Liquidation threshold edge cases
- Bad debt scenarios (underwater positions that can't be liquidated profitably)
- Interest rate model edge cases at 0% and 100% utilization
- Flash loan attacks on collateral valuation

## Proxy / Upgradeable

- Storage collision between proxy and implementation
- Uninitialized implementation (can attacker call `initialize`?)
- Function selector collision between proxy and implementation
- Storage gaps (`__gap`) — are they properly sized?
- `delegatecall` context: `msg.sender` and `msg.value` are preserved, storage is proxy's

## Governance / Timelock

- Flash loan governance attacks (borrow tokens, vote, return)
- Proposal lifecycle manipulation (cancel/execute timing)
- Timelock bypass paths
- Quorum manipulation

## Cross-Chain / Bridge

- Message replay across chains
- Failed message handling (stuck funds)
- Relayer trust assumptions
- Nonce management

## Staking / Restaking

- Withdrawal delay manipulation (can attacker front-run slashing by initiating withdrawal?)
- Slashing edge cases — does share accounting remain correct after partial slashing? Can slashing create bad debt?
- Operator trust boundaries — what can a malicious operator do? Can they grief delegators?
- Reward distribution timing — can staking just before reward distribution and unstaking after capture disproportionate rewards?
- Exchange rate manipulation between staking token and underlying
- Re-delegation during pending withdrawal — can user double-count stake?
- Minimum stake / dust stake DoS (many tiny stakes to bloat operator sets)

## Perpetuals / Derivatives

- Funding rate manipulation — can a large position skew the funding rate for profit?
- Liquidation cascades — does liquidating one position trigger liquidation of others (death spiral)?
- Mark price vs index price divergence — can manipulating the mark price trigger liquidations while index price is stable?
- ADL (auto-deleveraging) fairness — are profitable positions deleveraged fairly?
- Maximum open interest limits — can an attacker fill the OI cap to prevent new positions?
- Settlement price manipulation in expiring contracts
- Cross-margin vs isolated margin accounting errors
- Funding rate calculation at utilization boundaries

## Token Launch / Bonding Curves / LBP

- Price manipulation during initial liquidity — first buyer advantage, pre-launch sniping
- Bonding curve parameter attacks — can curve parameters be set to allow near-zero cost minting of large supply?
- LBP (Liquidity Bootstrapping Pool) weight manipulation timing
- Vesting cliff/unlock manipulation — can tokens be sold before intended unlock via wrapping, lending, or derivative creation?
- Anti-sniping mechanism bypass

## Yield Aggregators / Vaults (Multi-Strategy)

- Strategy composability — can a malicious or compromised strategy drain the vault?
- Harvest timing manipulation — front-running harvest to capture yield without proportional time exposure
- Loss socialization — when one strategy loses, is the loss spread fairly across all depositors?
- Strategy migration — are assets safe during the transition between old and new strategies?
- Reward token re-investment slippage — can sandwich attacks extract value during auto-compound?
- Emergency withdrawal paths — do they work when the underlying protocol is paused or compromised?
