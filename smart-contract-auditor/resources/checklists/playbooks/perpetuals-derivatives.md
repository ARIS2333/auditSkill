# Perpetuals / Derivatives Playbook

## 1. Funding Rate Manipulation

**What to look for:**
- Can a large position holder skew the funding rate to extract value from counterparties?
- Is the funding rate calculation based on manipulable inputs (spot price, order book depth)?
- Can funding rate be forced to extreme values via flash loans?

---

## 2. Mark Price vs Index Price Divergence

**What to look for:**
- If the mark price deviates significantly from the index, can positions be unfairly liquidated?
- Is the mark price smoothed or does it follow spot directly?
- Can an attacker manipulate the mark price without moving the index?

---

## 3. Liquidation Cascade

**What to look for:**
- Can liquidating one large position crash the mark price and trigger cascading liquidations?
- Is there a circuit breaker or price band to prevent cascading?
- Does the liquidation engine have priority ordering that can be gamed?

---

## 4. ADL (Auto-Deleveraging)

**What to look for:**
- If the insurance fund is depleted, is the ADL mechanism fair?
- Can it be gamed by opening opposing positions?
- Is the ADL priority queue transparent and correctly implemented?

---

## 5. Position Sizing Limits

**What to look for:**
- Are there maximum position sizes? Can an attacker open a position larger than the protocol can handle?
- Can position limits be bypassed via multiple accounts?
- Does the open interest cap account for both long and short sides?

---

## 6. Oracle Latency

**What to look for:**
- Derivatives are extremely sensitive to oracle timing — a few seconds of staleness can cause incorrect liquidations
- Can an attacker exploit the delay between price movement and oracle update?
- Is there a price deviation threshold that triggers a pause?

---

## 7. Margin Calculation

**What to look for:**
- Is isolated vs cross-margin correctly implemented?
- Can switching margin mode create an exploitable window?
- Does the margin calculation account for unrealized PnL on other positions (cross-margin)?

---

## 8. Fee Accounting

**What to look for:**
- Are fees applied correctly and consistently on open vs close?
- Can a zero-profit round-trip result in a loss greater than fees?
- Are maker/taker fees correctly distinguished?

---

## 9. Negative PnL Overflow

**What to look for:**
- Can a position's loss exceed the deposited margin?
- Who absorbs the excess — insurance fund, counterparties, or LPs?
- Is the socialization mechanism fair and correctly implemented?
