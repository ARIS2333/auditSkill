# Sherlock Submission Templates

All findings follow Sherlock's submission format. Each finding is submitted individually regardless of severity. Sherlock uses a distinct deduplication model — findings with the same root cause are grouped into a single issue.

---

## Individual Finding Template

Submit one per vulnerability. All fields are mandatory.

```markdown
### [Title describing the vulnerability]

### Summary

[1-2 sentence summary of what is wrong and why it matters.]

### Root Cause

In [`Contract.sol:L44`](https://github.com/sherlock-audit/YYYY-MM-project/blob/commit/src/Contract.sol#L44), [describe the specific code-level root cause — what line(s) are wrong and what they should do instead].

### Internal Pre-conditions

[Numbered list of conditions within the protocol that must be true for the attack to succeed. These are states the protocol or its users can reach through normal usage.]

1. [Admin/User needs to call `function()` to set `variable` to a specific value]
2. [Pool needs to have at least X tokens deposited]

### External Pre-conditions

[Numbered list of conditions outside the protocol that must be true. If none, write "None."]

1. [Token price needs to drop by X%]
2. [Chainlink oracle needs to return stale data]

### Attack Path

[Numbered step-by-step sequence of actions the attacker takes. Be specific — include function names, parameters, and what state changes at each step.]

1. [Attacker calls `deposit()` with X amount]
2. [Attacker calls `withdraw()` which triggers the vulnerable code path]
3. [Due to the root cause, attacker receives Y instead of Z]

### Impact

[Describe the concrete impact. Quantify if possible: dollar amount, percentage of TVL, affected users. State who suffers the loss and what they lose.]

[The affected party (users/protocol) suffers [approximate loss]. The attacker gains [approximate profit, if applicable].]

### PoC

<details>
<summary>Proof of Concept</summary>

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/TargetContract.sol";

contract ExploitPoC is Test {
    TargetContract public target;
    address public constant ATTACKER = address(0x1337);
    address public constant VICTIM = address(0xdead);

    function setUp() public {
        // Deploy or fork
        target = new TargetContract();
        // Or: vm.createSelectFork(vm.rpcUrl("mainnet"), BLOCK_NUMBER);

        vm.deal(ATTACKER, 100 ether);
        vm.deal(VICTIM, 1000 ether);

        vm.prank(VICTIM);
        target.deposit{value: 500 ether}();

        vm.label(address(target), "Target");
        vm.label(ATTACKER, "Attacker");
        vm.label(VICTIM, "Victim");
    }

    function test_Exploit() public {
        uint256 attackerBalBefore = ATTACKER.balance;
        console.log("Pre-exploit attacker balance:", attackerBalBefore);

        vm.startPrank(ATTACKER);
        // --- EXPLOIT LOGIC ---
        vm.stopPrank();

        uint256 attackerBalAfter = ATTACKER.balance;
        console.log("Post-exploit attacker balance:", attackerBalAfter);

        assertGt(attackerBalAfter, attackerBalBefore, "Exploit failed: no profit");
    }
}
```

</details>

Run: `forge test --match-test test_Exploit -vvvv`

### Mitigation

[Specific fix. Include a code diff when possible:]

```diff
 function withdraw() external {
     uint256 amount = balances[msg.sender];
+    balances[msg.sender] = 0;
     (bool success, ) = msg.sender.call{value: amount}("");
     require(success);
-    balances[msg.sender] = 0;
 }
```
```

---

## Severity Classification (Sherlock)

| Severity | Criteria |
|----------|----------|
| **High** | Definite loss of funds without extensive limitations on attack conditions. OR Inflicts serious non-material losses (e.g., permanent DoS of critical protocol functions). |
| **Medium** | Causes a loss of funds but requires certain external conditions or specific states. OR Breaks core contract functionality, rendering the contract useless or causing significant inconvenience. |

### Key Distinctions from Code4rena

| Aspect | Sherlock | Code4rena |
|--------|---------|-----------|
| **Deduplication** | Same root cause = same finding (even if different impacts) | More granular — different impacts can be separate findings |
| **Low/Info** | Not accepted in contests (unless specified) | Consolidated into QA report |
| **PoC requirement** | Strongly recommended for all severities | Mandatory for High/Medium |
| **Attack Path** | Mandatory numbered steps | Embedded in description |
| **Pre-conditions** | Explicitly separated (internal vs external) | Part of the general description |

### Severity Boundary Rules

**High vs Medium:**
- High: the attack path is viable with **no or minimal** external pre-conditions
- Medium: the attack path requires **specific but realistic** external conditions
- If it requires admin error, unlikely oracle failure, or >50% token price movement → Medium at best

**Medium vs Invalid:**
- If it requires the admin to be malicious AND the protocol has stated that admins are trusted → Invalid
- If it requires user error (sending wrong token, approving wrong amount) → Invalid
- If the loss is dust amounts → Invalid
- If it requires future code changes or hypothetical integrations → Invalid

**Sherlock-specific rules:**
- **EIP compliance deviations** are valid Medium findings ONLY if they cause material loss or break integrations
- **Admin trust:** If contest README states admins are TRUSTED, admin-privilege issues are Invalid. If RESTRICTED, admin overreach is valid.
- **Known issues** listed in the contest README or previous audits are Invalid
- **Gas griefing** without material impact is Invalid

---

## Submission Checklist

Before submitting each finding:

- [ ] Title clearly describes the vulnerability in one line
- [ ] Summary is concise (1-2 sentences)
- [ ] Root cause links to exact line number(s) in the contest GitHub repo
- [ ] Internal pre-conditions list realistic protocol states (not "admin sets value to max")
- [ ] External pre-conditions are genuinely external (market conditions, oracle state)
- [ ] Attack path is numbered, specific, and follows a logical sequence
- [ ] Impact quantifies the loss and identifies the affected party
- [ ] PoC compiles and demonstrates the exploit
- [ ] Severity is accurate per Sherlock's criteria
- [ ] Finding does not duplicate a known issue from the README
- [ ] Finding is within the defined audit scope
- [ ] If the finding depends on admin trust, verify the contest's trust assumptions
- [ ] The root cause is the true underlying issue, not a symptom (Sherlock deduplicates by root cause)
