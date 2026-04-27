// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/TargetContract.sol";

contract ExploitPoC is Test {
    TargetContract public target;
    address public constant ATTACKER = address(0x1337);
    address public constant VICTIM = address(0xdead);

    function setUp() public {
        // Option A: Local deployment
        target = new TargetContract();

        // Option B: Fork mainnet and reference deployed contract
        // vm.createSelectFork(vm.rpcUrl("mainnet"), 18_000_000);
        // target = TargetContract(0xDeployedAddress);

        // Fund accounts
        vm.deal(ATTACKER, 100 ether);
        vm.deal(VICTIM, 1000 ether);

        // Set up initial protocol state
        vm.prank(VICTIM);
        target.deposit{value: 500 ether}();

        // Label addresses for readable traces
        vm.label(address(target), "Target");
        vm.label(ATTACKER, "Attacker");
        vm.label(VICTIM, "Victim");
    }

    function test_Exploit() public {
        // 1. Record pre-exploit state
        uint256 attackerBalBefore = ATTACKER.balance;
        uint256 targetBalBefore = address(target).balance;
        console.log("Pre-exploit attacker balance:", attackerBalBefore);
        console.log("Pre-exploit target balance:", targetBalBefore);

        // 2. Execute exploit
        vm.startPrank(ATTACKER);

        // --- EXPLOIT LOGIC HERE ---

        vm.stopPrank();

        // 3. Validate exploit success
        uint256 attackerBalAfter = ATTACKER.balance;
        uint256 targetBalAfter = address(target).balance;
        console.log("Post-exploit attacker balance:", attackerBalAfter);
        console.log("Post-exploit target balance:", targetBalAfter);

        assertGt(attackerBalAfter, attackerBalBefore, "Exploit failed: attacker did not profit");
        assertLt(targetBalAfter, targetBalBefore, "Exploit failed: target funds not drained");
    }
}
