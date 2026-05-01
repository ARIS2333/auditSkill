# Vault / ERC4626 Playbook

## 1. First Depositor Inflation Attack

**What it is:** Attacker deposits 1 wei, then donates a large amount directly to the vault. The next depositor's shares round down to 0 due to the inflated share price, and the attacker redeems all assets.

**What to look for in code:**
- `convertToShares` when `totalSupply == 0` — does it use the raw `totalAssets()` or does it have an offset?
- Missing virtual shares / virtual assets (the OpenZeppelin mitigation adds a `_decimalsOffset()`)
- Missing minimum deposit requirements

**Vulnerable pattern:**
```solidity
function convertToShares(uint256 assets) public view returns (uint256) {
    uint256 supply = totalSupply();
    return supply == 0 ? assets : assets.mulDiv(supply, totalAssets());
}
```

**Safe pattern:**
```solidity
function convertToShares(uint256 assets) public view returns (uint256) {
    return assets.mulDiv(totalSupply() + 10 ** _decimalsOffset(), totalAssets() + 1);
}
```

**PoC strategy:** Attacker deposits 1 wei, transfers 10000 tokens directly to vault, victim deposits 9999 tokens, assert victim gets 0 shares. Cheaper with low-decimal tokens (USDC: ~$10 attack vs 18-decimal: ~$10M).

---

## 2. Share Price Manipulation via Donation

**What it is:** Direct token transfer to the vault inflates `totalAssets()` (via `balanceOf(address(this))`) without minting shares, skewing the share price for all subsequent operations.

**What to look for in code:**
- `totalAssets()` uses `balanceOf(address(this))` — any direct transfer changes the return value
- Accounting that compares internal tracking against actual balance

**Impact beyond share pricing:** Donation can also break precondition gates. Search for every `balanceOf(address(this))` usage — administrative functions, lifecycle checks, and unregistration gates may be affected.

---

## 3. Rounding Direction

**Rule:** Deposits and mints should round DOWN (user gets fewer shares). Withdrawals and redeems should round DOWN (user gets fewer assets). Always favor the vault.

**What to look for:**
- `mulDiv` with wrong rounding argument (Math.Rounding.Floor vs Math.Rounding.Ceil)
- Custom math that divides without explicit rounding control
- Dust accumulation over many small operations that creates an exploitable gap

**Check all four ERC4626 entry points:** `deposit`, `mint`, `withdraw`, `redeem`. Each must round consistently.

---

## 4. Four-Function Consistency

`deposit`, `mint`, `redeem`, `withdraw` must produce consistent results:
- `deposit(assets)` should give the same shares as `mint(convertToShares(assets))`
- `withdraw(assets)` should burn the same shares as `redeem(convertToAssets(shares))`

**What to look for:** Different code paths for each function that can diverge due to rounding, fee application, or state reads at different points.

---

## 5. Empty Vault Edge Case

When `totalSupply == 0`, all four functions must behave correctly:
- `convertToShares` and `convertToAssets` must not divide by zero
- `maxWithdraw` and `maxRedeem` should return 0 for all users
- The first depositor should receive a fair number of shares

---

## 6. Preview vs Actual Discrepancy

`previewDeposit`, `previewMint`, `previewWithdraw`, `previewRedeem` must return values that match (or are more pessimistic than) the actual operation. If preview returns X shares but the actual deposit gives X-1, integrators break.

**What to look for:** Preview functions that don't account for fees, slippage, or state changes between preview and execution.

---

## 7. `maxDeposit` / `maxMint` / `maxWithdraw` / `maxRedeem`

These must return accurate limits. If `maxWithdraw` returns a value larger than what `withdraw` can actually process (e.g., due to liquidity constraints in underlying strategies), callers that trust the max value will get unexpected reverts.

---

## 8. Fee-on-Transfer Token Interaction

If the vault accepts arbitrary ERC20 tokens, check `resources/checklists/non-standard-tokens.md` §1 (Fee-on-Transfer). The vault must use balance-before/balance-after accounting, not the raw `amount` from `transferFrom`.
