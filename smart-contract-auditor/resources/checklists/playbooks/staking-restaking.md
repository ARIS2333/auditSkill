# Staking / Restaking Playbook

## 1. Withdrawal Delay Manipulation

**What to look for:**
- Can an attacker trigger a withdrawal delay and then exploit the locked-in position (e.g., slashing while funds are in cooldown)?
- Can the delay be changed by an admin while withdrawals are pending?
- Is there a maximum delay that prevents funds from being locked indefinitely?

---

## 2. Slashing Accounting

**What to look for:**
- Does slashing reduce stake proportionally across all delegators?
- Can slashing cause underflow or leave dust?
- What happens if slashing exceeds the staked amount?
- Is slashed amount correctly reflected in share-based accounting?

---

## 3. Reward Distribution

**What to look for:**
- Is reward calculation based on time-weighted stake? Can a just-in-time staker capture rewards disproportionately?
- Rounding in reward-per-share calculations that accumulates over time
- Are rewards correctly distributed when new stakers join mid-epoch?
- Can unclaimed rewards be stolen or lost?

---

## 4. Operator Delegation

**What to look for:**
- If users delegate to operators, can a malicious operator grief delegators (intentional slashing, withholding rewards)?
- Can an operator change delegation terms after users have delegated?
- Is there a path for delegators to exit if the operator becomes malicious?

---

## 5. Unbonding Queue

**What to look for:**
- Is there a maximum queue length? Can the queue be DoS'd by many small unbonding requests?
- What happens if the queue fills up — are new unbonding requests rejected or do they overwrite?
- Can an attacker create dust unbonding requests to block legitimate withdrawals?

---

## 6. Re-Staking Compounding

**What to look for:**
- If rewards are automatically restaked, is the compounding math correct?
- Does it account for rounding in each compounding step?
- Can the compound frequency be manipulated for profit?

---

## 7. Checkpoint / Epoch Boundaries

**What to look for:**
- Can operations that span epoch boundaries produce inconsistent state?
- Is there a window at epoch transitions where invariants temporarily break?
- Can an attacker time actions to exploit the transition window?

---

## 8. Minimum Stake

**What to look for:**
- Can dust stakes pollute the staker set or consume unbounded storage?
- Is there a minimum stake enforced on entry? On partial withdrawal?
- Can minimum stake requirements trap funds (stake below minimum but non-zero)?
