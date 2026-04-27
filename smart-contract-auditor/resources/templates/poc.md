## Template 1: ETH Drain PoC

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
```

---

## Template 2: ERC20 Token Drain PoC

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/TargetVault.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract TokenDrainPoC is Test {
    TargetVault public vault;
    IERC20 public token;

    address public constant ATTACKER = address(0x1337);
    address public constant VICTIM = address(0xdead);
    uint256 public constant VICTIM_DEPOSIT = 1_000_000e18;

    function setUp() public {
        // Option A: Local deployment with mock token
        token = IERC20(address(new MockERC20()));
        vault = new TargetVault(address(token));

        // Option B: Fork mainnet
        // vm.createSelectFork(vm.rpcUrl("mainnet"), 18_000_000);
        // token = IERC20(0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48); // USDC
        // vault = TargetVault(0xDeployedAddress);

        // Fund accounts
        deal(address(token), ATTACKER, 10_000e18);
        deal(address(token), VICTIM, VICTIM_DEPOSIT);

        // Victim deposits into vault
        vm.startPrank(VICTIM);
        token.approve(address(vault), type(uint256).max);
        vault.deposit(VICTIM_DEPOSIT);
        vm.stopPrank();

        vm.label(address(vault), "Vault");
        vm.label(address(token), "Token");
        vm.label(ATTACKER, "Attacker");
        vm.label(VICTIM, "Victim");
    }

    function test_TokenDrain() public {
        uint256 attackerTokenBefore = token.balanceOf(ATTACKER);
        uint256 vaultTokenBefore = token.balanceOf(address(vault));
        console.log("Pre-exploit attacker tokens:", attackerTokenBefore);
        console.log("Pre-exploit vault tokens:", vaultTokenBefore);

        vm.startPrank(ATTACKER);
        token.approve(address(vault), type(uint256).max);

        // --- EXPLOIT LOGIC HERE ---

        vm.stopPrank();

        uint256 attackerTokenAfter = token.balanceOf(ATTACKER);
        uint256 vaultTokenAfter = token.balanceOf(address(vault));
        console.log("Post-exploit attacker tokens:", attackerTokenAfter);
        console.log("Post-exploit vault tokens:", vaultTokenAfter);

        assertGt(attackerTokenAfter, attackerTokenBefore, "Exploit failed: attacker did not profit");
        assertLt(vaultTokenAfter, vaultTokenBefore, "Exploit failed: vault not drained");
    }
}
```

---

## Template 3: Access Control Bypass PoC

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/TargetContract.sol";

contract AccessControlBypassPoC is Test {
    TargetContract public target;

    address public constant ADMIN = address(0xAD);
    address public constant ATTACKER = address(0x1337);

    function setUp() public {
        vm.prank(ADMIN);
        target = new TargetContract();

        vm.label(address(target), "Target");
        vm.label(ADMIN, "Admin");
        vm.label(ATTACKER, "Attacker");
    }

    function test_AccessControlBypass() public {
        // 1. Verify the attacker is NOT the admin / privileged role
        assertTrue(ATTACKER != target.owner(), "Attacker should not be owner");

        // 2. Verify the action should be restricted (optional — show it works for admin)
        vm.prank(ADMIN);
        target.restrictedFunction();  // Should succeed

        // 3. Execute the bypass as unprivileged attacker
        vm.startPrank(ATTACKER);

        // --- BYPASS LOGIC HERE ---
        // e.g., target.restrictedFunction(); // should revert but doesn't

        vm.stopPrank();

        // 4. Assert the attacker achieved the privileged action
        // assertEq(target.owner(), ATTACKER, "Attacker took ownership");
        // OR: assert some restricted state was changed
    }
}
```

---

## Template 4: Flash Loan Attack PoC

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/TargetProtocol.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

interface IFlashLender {
    function flashLoan(address receiver, address token, uint256 amount, bytes calldata data) external;
}

contract FlashLoanAttack {
    TargetProtocol public target;
    IFlashLender public lender;
    IERC20 public token;

    constructor(address _target, address _lender, address _token) {
        target = TargetProtocol(_target);
        lender = IFlashLender(_lender);
        token = IERC20(_token);
    }

    function attack() external {
        uint256 borrowAmount = 10_000_000e18;
        lender.flashLoan(address(this), address(token), borrowAmount, "");
    }

    // Flash loan callback (naming depends on the lender: onFlashLoan, executeOperation, etc.)
    function onFlashLoan(
        address initiator,
        address _token,
        uint256 amount,
        uint256 fee,
        bytes calldata
    ) external returns (bytes32) {
        // --- EXPLOIT LOGIC DURING FLASH LOAN ---
        // e.g., manipulate price oracle, inflate collateral, drain funds

        // Repay flash loan
        IERC20(_token).approve(msg.sender, amount + fee);
        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }
}

contract FlashLoanPoC is Test {
    TargetProtocol public target;
    IERC20 public token;
    FlashLoanAttack public attackContract;

    address public constant ATTACKER = address(0x1337);
    address public constant VICTIM = address(0xdead);

    function setUp() public {
        // Fork mainnet (flash loan attacks usually need real liquidity)
        vm.createSelectFork(vm.rpcUrl("mainnet"), 18_000_000);

        target = TargetProtocol(0xTargetAddress);
        token = IERC20(0xTokenAddress);
        // lender = IFlashLender(0xLenderAddress); // Aave, Balancer, etc.

        vm.startPrank(ATTACKER);
        attackContract = new FlashLoanAttack(
            address(target),
            address(0xLenderAddress),
            address(token)
        );
        vm.stopPrank();

        // Set up victim state
        deal(address(token), VICTIM, 1_000_000e18);
        vm.startPrank(VICTIM);
        token.approve(address(target), type(uint256).max);
        target.deposit(1_000_000e18);
        vm.stopPrank();

        vm.label(address(target), "Target");
        vm.label(address(attackContract), "AttackContract");
        vm.label(ATTACKER, "Attacker");
    }

    function test_FlashLoanAttack() public {
        uint256 attackerBefore = token.balanceOf(ATTACKER);
        console.log("Pre-exploit attacker balance:", attackerBefore);

        vm.prank(ATTACKER);
        attackContract.attack();

        // Transfer profits from attack contract to attacker EOA
        vm.prank(ATTACKER);
        uint256 profit = token.balanceOf(address(attackContract));
        // attackContract.withdrawProfit();

        uint256 attackerAfter = token.balanceOf(ATTACKER);
        console.log("Post-exploit attacker balance:", attackerAfter);

        assertGt(attackerAfter, attackerBefore, "Flash loan attack did not profit");
    }
}
```

---

## Template 5: Oracle Manipulation PoC

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/TargetLending.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

interface IOracle {
    function getPrice(address token) external view returns (uint256);
}

contract OracleManipulationPoC is Test {
    TargetLending public lending;
    IOracle public oracle;
    IERC20 public collateralToken;
    IERC20 public borrowToken;

    address public constant ATTACKER = address(0x1337);

    function setUp() public {
        // Deploy or fork
        // vm.createSelectFork(vm.rpcUrl("mainnet"), 18_000_000);

        lending = new TargetLending();
        oracle = IOracle(address(lending.oracle()));
        collateralToken = IERC20(address(lending.collateralToken()));
        borrowToken = IERC20(address(lending.borrowToken()));

        deal(address(collateralToken), ATTACKER, 100e18);
        deal(address(borrowToken), address(lending), 1_000_000e18);

        vm.label(address(lending), "Lending");
        vm.label(address(oracle), "Oracle");
        vm.label(ATTACKER, "Attacker");
    }

    function test_OracleManipulation() public {
        uint256 attackerBorrowBefore = borrowToken.balanceOf(ATTACKER);

        // 1. Show normal price
        uint256 normalPrice = oracle.getPrice(address(collateralToken));
        console.log("Normal collateral price:", normalPrice);

        // 2. Mock the oracle to return manipulated price
        vm.mockCall(
            address(oracle),
            abi.encodeWithSelector(IOracle.getPrice.selector, address(collateralToken)),
            abi.encode(normalPrice * 100) // 100x inflated price
        );

        // 3. Exploit: deposit small collateral, borrow at inflated valuation
        vm.startPrank(ATTACKER);
        collateralToken.approve(address(lending), type(uint256).max);
        lending.depositCollateral(1e18);  // Small amount
        lending.borrow(500_000e18);       // Borrow far more than collateral is worth
        vm.stopPrank();

        // 4. Clear mock to restore normal pricing
        vm.clearMockedCalls();

        uint256 attackerBorrowAfter = borrowToken.balanceOf(ATTACKER);
        console.log("Attacker borrowed:", attackerBorrowAfter - attackerBorrowBefore);

        assertGt(
            attackerBorrowAfter - attackerBorrowBefore,
            normalPrice * 1e18 / 1e18,  // More than fair value of collateral
            "Oracle manipulation failed"
        );
    }
}
```
