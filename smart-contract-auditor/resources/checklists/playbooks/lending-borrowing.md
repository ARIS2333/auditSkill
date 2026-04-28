# Lending / Borrowing Playbook

## 1. Oracle Staleness and Manipulation

**What to look for:**
- Chainlink: missing `updatedAt` staleness check, missing sequencer uptime check (L2s), missing `answeredInRound >= roundId` check
- Pyth: missing `publishTime` validation, missing confidence interval check
- Spot price oracles (AMM reserves, `balanceOf`): manipulable via flash loan within a single transaction
- Hardcoded heartbeat values that don't match the actual oracle's update frequency

**Vulnerable pattern:**
```solidity
(, int256 price, , , ) = priceFeed.latestRoundData();
return uint256(price); // No staleness check, no negative check
```

**Safe pattern:**
```solidity
(uint80 roundId, int256 price, , uint256 updatedAt, uint80 answeredInRound) = priceFeed.latestRoundData();
require(price > 0, "Negative price");
require(updatedAt > block.timestamp - HEARTBEAT, "Stale price");
require(answeredInRound >= roundId, "Stale round");
```

**PoC strategy:** Use `vm.mockCall` to return stale or zero prices, then demonstrate incorrect liquidation or borrowing behavior.

---

## 2. Liquidation Threshold Edge Cases

**What to look for:**
- Positions exactly at the liquidation boundary — can a small price movement cause mass liquidations?
- Self-liquidation: can a user liquidate their own position for profit (especially with liquidation bonuses)?
- Partial liquidation math: does liquidating 50% of a position leave the remaining 50% in a worse health state?
- Liquidation bonuses that exceed the collateral value for underwater positions

**PoC strategy:** Create a position at exactly the liquidation threshold, mock a 0.1% price drop, attempt liquidation, check the liquidator's profit and the remaining position health.

---

## 3. Bad Debt Scenarios

**What to look for:**
- Underwater positions where collateral value < debt — can these be liquidated profitably?
- If liquidation is unprofitable, bad debt accumulates. Is there a socialization mechanism?
- Cascading liquidations: does liquidating one large position crash the collateral price (via AMM), triggering more liquidations?

**Key question:** "What happens when a position's collateral is worth less than the gas cost to liquidate it?"

---

## 4. Interest Rate Model Edge Cases

**What to look for:**
- Utilization at 0%: does the interest rate formula handle zero utilization without division by zero?
- Utilization at 100%: are interest rates so high they cause overflow? Do borrowers have any path to repay?
- Interest accrual precision: does rounding dust in interest calculation accumulate over time?
- Jump rate model: is the kink point calibrated correctly? A misconfigured kink can cause utilization to jump directly from low to 100%.

---

## 5. Flash Loan Attacks on Collateral Valuation

**What to look for:**
- Collateral valued using spot prices (AMM reserves) — flash-loan-manipulable
- Borrow → manipulate oracle → borrow more → repay → profit
- Deposit manipulated collateral → borrow against inflated value → let position get liquidated

**PoC strategy:** Fork mainnet, use a flash loan provider (Aave, Balancer), manipulate the price source, demonstrate profitable exploitation within a single transaction.

---

## 6. Repayment and Withdrawal Ordering

**What to look for:**
- Can a user repay debt and withdraw collateral in the same transaction to avoid liquidation checks?
- Is the health factor checked after each operation or only at the end?
- Borrow/repay rounding: does repaying `borrowBalance` exactly clear the debt, or does dust remain?

---

## 7. Reserve Factor and Fee Extraction

**What to look for:**
- Is the reserve factor applied correctly to interest? (Protocol takes a cut of interest income)
- Can the admin set reserve factor to 100% and steal all interest?
- Are fees calculated on the correct base (new interest accrued, not total outstanding)?

---

## 8. Cross-Asset Risks

**What to look for:**
- Correlated assets: can depositing correlated collateral (stETH + ETH) create hidden concentration risk?
- Token-specific behaviors: rebasing tokens as collateral (balance changes without transfers), fee-on-transfer tokens, pausable tokens blocking liquidation
- Isolation mode bypass: if certain assets are isolated, can cross-collateral mixing circumvent the isolation?

**Cross-reference:** `resources/checklists/non-standard-tokens.md` for token-specific risks.
