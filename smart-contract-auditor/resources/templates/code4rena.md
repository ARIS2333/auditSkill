# Code4rena Submission Templates

All findings follow Code4rena's submission format. High/Medium findings are submitted individually. Low and Centralization findings are consolidated into a single QA report.

---

## Individual High/Medium Finding Template

Submit one per finding. All fields are mandatory for Solidity/EVM audits.

```markdown
## [H-01] Title describing the vulnerability (max 255 chars)

### Root Cause

In [`Contract.sol#L44-L55`](https://github.com/code-423n4/YYYY-MM-project/blob/commit/src/Contract.sol#L44-L55), [describe the root cause — what is wrong in the code and why].

### Impact

[What can an attacker achieve? Quantify if possible: fund theft amount, percentage of TVL at risk, permanent vs temporary DoS, etc.]

### Proof of Concept

<details>

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

    function test_H01_Exploit() public {
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

Run command (from repo root):

```
forge test --match-test test_H01_Exploit -vv
```

Output:

```
[PASTE ACTUAL forge test OUTPUT HERE — must be from a real run, never fabricated]
```

The output confirms:

- [Bullet point explaining what each passing assertion proves about the vulnerability]
- [Bullet point linking the test result to the root cause described above]
- [Bullet point showing the concrete impact demonstrated by the PoC]

### Recommended Mitigation

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

## QA Report Template

One consolidated report per warden per audit. Contains all Low-risk and Centralization/Governance findings.

Use ONLY these labels:
- `L-01`, `L-02`, ... — Low-risk findings
- `C-01`, `C-02`, ... — Centralization/governance findings

Do NOT use `R-` (refactor) or `I-` (informational) labels.

```markdown
# QA Report

## Summary

| Label | Count |
|-------|-------|
| L-XX  | [N]   |
| C-XX  | [N]   |

## Low Risk

### [L-01] Title

In [`Contract.sol#L88`](link), [describe the issue].

**Impact:** [Low-severity impact description]

**Recommended Mitigation:** [Fix]

### [L-02] Title

...

## Centralization / Governance

### [C-01] Title

In [`Contract.sol#L22`](link), [describe the centralization risk].

**Impact:** [What can the privileged role do, and why is it a risk]

**Recommended Mitigation:** [Fix — e.g., timelock, multisig, governance vote]

### [C-02] Title

...
```

---

## Severity Quick Reference (Code4rena)

| Severity | Criteria | Submission Type |
|----------|----------|-----------------|
| **High (3)** | Assets stolen/lost/compromised directly or via valid attack path. Direct theft, loss of matured yield, real fee losses. | Individual, coded PoC mandatory |
| **Medium (2)** | Assets not at direct risk, but protocol function/availability impacted, or value leaks via hypothetical path with stated assumptions. Loss of unmatured yield. | Individual, coded PoC mandatory |
| **Low (QA)** | State handling errors, spec deviations, missing validation, dust losses. | Consolidated QA report |
| **Centralization (QA)** | Admin privilege risks, governance centralization, single points of failure. | Consolidated QA report |

### Boundary Rules

- **High vs Medium:** High = direct viable path to asset loss. Medium = requires external conditions/assumptions.
- **Medium vs QA:** If it depends on user error, speculative future changes, or non-standard token behavior → QA.
- **Do not overstate severity.** Code4rena penalizes QA findings submitted as Medium/High.
- **Known issues** in the audit repo README are out of scope **only if same severity and same root cause**. If you can demonstrate higher severity or a substantively distinct attack path, the finding is in scope — reference the known issue and explain what is new.

---

## Submission Checklist

Before submitting each finding:

- [ ] Title is under 255 characters and clearly describes the issue
- [ ] Root cause links point to exact line numbers in the audit repo GitHub
- [ ] Root cause explanation identifies WHAT is wrong and WHY
- [ ] Impact section quantifies the damage where possible
- [ ] PoC is coded, compiles, and runs (High/Medium only)
- [ ] PoC uses the project's test suite / Foundry framework
- [ ] Recommended mitigation is specific and actionable
- [ ] Severity is accurate (not overstated)
- [ ] Finding is not a true duplicate of a known issue (same severity + same root cause). If it overlaps with a known issue, the write-up explicitly states what is new (higher severity or distinct attack path).
- [ ] Finding is within the defined audit scope
