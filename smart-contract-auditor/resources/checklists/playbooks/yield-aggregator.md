# Yield Aggregator (Multi-Strategy) Playbook

## 1. Strategy Trust

**What to look for:**
- When the vault deposits into a strategy, can a malicious strategy steal or lock funds?
- Is there a maximum allocation limit per strategy?
- Is strategy addition/removal gated by timelock?

---

## 2. Harvest Sandwich

**What to look for:**
- Can a bot sandwich the `harvest()` transaction to extract value from yield being compounded?
- Is harvest callable by anyone, or restricted to keepers?
- Does the harvest include a swap that can be front-run?

---

## 3. Strategy Withdrawal Ordering

**What to look for:**
- If the vault needs to withdraw but the primary strategy has locked funds, does it try secondary strategies?
- Can this ordering be exploited (e.g., force withdrawal from a strategy with unfavorable exit terms)?
- Is there a maximum withdrawal queue that can be DoS'd?

---

## 4. Emergency Withdrawal

**What to look for:**
- Does the emergency path sacrifice yield but guarantee fund recovery?
- Can it be called by anyone, or only admin?
- Is there a scenario where emergency withdrawal also fails (strategy is bricked)?

---

## 5. Total Assets Calculation

**What to look for:**
- Does `totalAssets` correctly sum assets in all strategies, accounting for unrealized gains/losses?
- Can a strategy report inflated assets to manipulate share price?
- Is there a lag between strategy reporting and vault accounting?

---

## 6. Strategy Migration

**What to look for:**
- When rotating from old to new strategy, is there a window where funds are in transit and neither strategy accounts for them?
- Can the migration be sandwiched?
- Does migration preserve user accounting correctly?

---

## 7. Reward Token Handling

**What to look for:**
- Are reward tokens (CRV, COMP, etc.) correctly swapped and compounded?
- Can the swap be sandwiched for MEV extraction?
- Are reward tokens with non-standard behavior handled (rebasing rewards, vesting rewards)?
- Can unclaimed rewards be lost during strategy migration?
