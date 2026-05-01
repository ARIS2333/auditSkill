# Token Launch / Bonding Curve Playbook

## 1. Bonding Curve Manipulation

**What to look for:**
- Can early buyers extract disproportionate value from later buyers by timing purchases around curve inflection points?
- Is the curve formula monotonically increasing? Flat or decreasing segments can be arbitraged.
- Can buying and selling in quick succession extract value from the curve (sandwich the curve)?

---

## 2. LBP (Liquidity Bootstrapping Pool) Sniping

**What to look for:**
- Can MEV bots snipe the launch by buying in the first block?
- Is there a whitelist period or anti-bot mechanism?
- Can the weight change schedule be front-run?

---

## 3. Vesting Cliff Bypass

**What to look for:**
- Can tokens locked under vesting be transferred, delegated, or used as collateral before the cliff?
- Can the vesting contract be upgraded to change terms?
- Are vesting tokens represented as transferable ERC20 wrappers?

---

## 4. Unlock Schedule Math

**What to look for:**
- Is the linear unlock calculation correct? Off-by-one in time boundaries can unlock tokens early or late.
- Does the unlock function handle partial claims correctly?
- What happens if `block.timestamp` is exactly on a boundary?

---

## 5. Rug Pull Vectors

**What to look for:**
- Can the token creator drain liquidity from the pool?
- Can transfers be paused or blocked by the owner?
- Can the owner mint unlimited tokens after launch?
- Is there a renounce-ownership mechanism, and has it been called?

---

## 6. Maximum Supply Enforcement

**What to look for:**
- Is there a hard cap that cannot be bypassed by the owner or governance?
- Can the cap be changed after deployment?
- Does the mint function correctly enforce the cap (check for overflow)?
