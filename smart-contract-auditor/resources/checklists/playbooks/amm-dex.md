# AMM / DEX Playbook

## 1. Price Manipulation via Flash Loans

**What it is:** Attacker borrows a large amount, swaps to move the AMM price, exploits the skewed price in a dependent protocol (lending, options, etc.), then swaps back and repays. All in one transaction.

**What to look for:**
- Any contract that reads AMM reserves or spot prices for valuation decisions
- `getAmountOut`, `getReserves`, `slot0` (Uniswap V3 current tick) used as price oracle
- Missing TWAP usage — spot prices are flash-loan-manipulable, TWAPs are resistant

**Vulnerable pattern:**
```solidity
(uint112 reserve0, uint112 reserve1, ) = pair.getReserves();
uint256 price = reserve1 * 1e18 / reserve0; // Spot price — manipulable
```

**PoC strategy:** Fork mainnet, flash-borrow from Aave/Balancer, execute large swap on the AMM, show the dependent protocol now uses the skewed price, extract profit, swap back, repay.

---

## 2. Sandwich Attack Vulnerability

**What to look for:**
- Swaps with user-provided slippage tolerance that is too loose (or hardcoded to 0)
- Missing `deadline` parameter on swap functions (transaction can be held in mempool and executed at an unfavorable time)
- `amountOutMin = 0` as a default

**Key question:** "Can a MEV bot front-run this transaction, move the price, and profit from the user's worse execution?"

---

## 3. Constant Product Invariant Violations

For `x * y = k` AMMs:
- Does every swap maintain `k` (accounting for fees)? A swap that reduces `k` allows value extraction.
- Does adding liquidity proportionally to reserves? Imbalanced adds can gift value to the pool.
- Does removing liquidity return proportional assets? A miscalculation here is direct fund loss.

**What to look for:**
- Rounding in the swap formula that consistently favors the trader over the pool
- Fee calculation that doesn't correctly reduce the input amount before computing the output
- `skim()` functions that can be called to extract excess tokens

---

## 4. Liquidity Addition / Removal Edge Cases

**What to look for:**
- First liquidity provider: how is the initial LP token amount calculated? Is there a minimum liquidity lock (Uniswap V2 burns 1000 wei to address(0))?
- Single-sided liquidity addition: if supported, is the implicit swap correctly priced?
- Removing 100% of liquidity: does it leave dust? Can the last LP provider get stuck?
- Liquidity removal when pool is imbalanced (e.g., after a large swap)

---

## 5. Fee Accounting

**What to look for:**
- Are fees accumulated correctly in the pool reserves?
- Protocol fee extraction: can the admin extract fees in a way that reduces reserves below what LP tokens represent?
- Fee-on-transfer tokens: if the pool accepts arbitrary tokens, fee-on-transfer tokens will cause reserve accounting to diverge from actual balances.

---

## 6. Concentrated Liquidity (Uniswap V3 Style)

**What to look for:**
- Tick boundary arithmetic: off-by-one errors at tick boundaries can cause price discontinuities
- Position NFT management: can someone else modify or close your position?
- `tickSpacing` enforcement: can liquidity be added at invalid ticks?
- Fee growth accounting: is `feeGrowthOutside` correctly flipped when price crosses a tick?

---

## 7. Multi-Hop and Routing

**What to look for:**
- Intermediate token balances: if a multi-hop swap fails midway, are intermediate tokens stuck in the router?
- Callback trust: Uniswap V3-style `swapCallback` — is the callback caller validated? Can a malicious pool contract call the router's callback?
- Leftover handling: does the router return unused tokens/ETH to the user?

---

## 8. Oracle Integration

If the AMM provides TWAP or other oracle functionality:
- Can the TWAP be manipulated by keeping the price skewed across multiple blocks?
- What is the observation cardinality? Insufficient observations make the TWAP easier to manipulate.
- Is the TWAP read from the correct observation window?
