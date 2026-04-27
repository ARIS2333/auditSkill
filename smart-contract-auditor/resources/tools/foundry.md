# Foundry Audit Reference Guide

Official GitHub: https://github.com/foundry-rs/foundry

Comprehensive reference for smart contract auditors using Foundry to build, test, and execute Proof-of-Concept (PoC) exploit scripts. All commands and cheatcodes sourced from the official Foundry documentation at [getfoundry.sh](https://www.getfoundry.sh/).

---

## Table of Contents

1. [Installation & Environment Setup](#1-installation--environment-setup)
2. [Project Setup for Audit Engagements](#2-project-setup-for-audit-engagements)
3. [Build Troubleshooting](#3-build-troubleshooting)
4. [Forge CLI Reference](#4-forge-cli-reference)
5. [Forge Test Command](#5-forge-test-command)
6. [Cheatcodes for PoC Development](#6-cheatcodes-for-poc-development)
7. [Forge-std Helpers & Assertions](#7-forge-std-helpers--assertions)
8. [Mainnet Forking](#8-mainnet-forking)
9. [Anvil Local Testnet](#9-anvil-local-testnet)
10. [Cast CLI for Chain Interaction](#10-cast-cli-for-chain-interaction)
11. [Configuration Reference (foundry.toml)](#11-configuration-reference-foundrytoml)
12. [PoC Development Workflow](#12-poc-development-workflow)
13. [PoC Template](#13-poc-template)
14. [Invariant Testing for Audits](#14-invariant-testing-for-audits)

---

## 1. Installation & Environment Setup

### Install Foundry

```bash
# Recommended: foundryup installer
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Via cargo (Rust toolchain required)
cargo install --git https://github.com/foundry-rs/foundry --profile release forge cast anvil chisel
```

### Verify Installation

```bash
forge --version
cast --version
anvil --version
chisel --version
```

### Foundry Toolkit Overview

| Tool | Purpose |
|------|---------|
| `forge` | Build, test, debug, deploy, and verify smart contracts |
| `cast` | Interact with contracts, send transactions, query chain data |
| `anvil` | Local Ethereum node with forking capabilities |
| `chisel` | Solidity REPL for rapid prototyping |

---

## 2. Project Setup for Audit Engagements

### Scenario A: Target Codebase is a Foundry Project

```bash
cd target-project/
forge install
forge build
```

### Scenario B: Initialize a Fresh Foundry Project for Audit PoCs

```bash
forge init audit-pocs
cd audit-pocs
```

Default project structure:

```
.
├── foundry.toml       # Project configuration
├── src/               # Source contracts (place target contracts here)
├── test/              # Test/PoC files
├── script/            # Deployment/execution scripts
└── lib/               # Dependencies (git submodules)
```

### Scenario C: Non-Foundry Codebase (Hardhat/Truffle)

Create a Foundry workspace alongside the existing project:

```bash
# In the target project root
forge init --force .
```

Or create a standalone PoC project and copy target contracts:

```bash
forge init audit-pocs
cp -r ../target-project/contracts/ audit-pocs/src/
```

### Installing Dependencies

```bash
# Install via git submodules (default)
forge install OpenZeppelin/openzeppelin-contracts
forge install OpenZeppelin/openzeppelin-contracts-upgradeable
forge install transmissions11/solmate
forge install Uniswap/v3-core

# Install specific version/tag
forge install OpenZeppelin/openzeppelin-contracts@v5.0.0

# Update all dependencies
forge update

# Remove a dependency
forge remove solmate
```

### Configure Remappings

After installing dependencies, Foundry auto-detects remappings. View them:

```bash
forge remappings
```

Add custom remappings in `foundry.toml`:

```toml
[profile.default]
remappings = [
    "@openzeppelin/=lib/openzeppelin-contracts/",
    "@openzeppelin/contracts-upgradeable/=lib/openzeppelin-contracts-upgradeable/contracts/",
    "solmate/=lib/solmate/src/",
]
```

Or in a `remappings.txt` file (one per line):

```
@openzeppelin/=lib/openzeppelin-contracts/
solmate/=lib/solmate/src/
```

---

## 3. Build Troubleshooting

### Common Build Failures & Fixes

**Missing dependencies / unresolved imports:**

```bash
# Check what's missing from import paths
forge build 2>&1 | grep "not found"

# Install the missing library
forge install <github-org>/<repo>

# If using @openzeppelin imports
forge install OpenZeppelin/openzeppelin-contracts
```

**Solidity version mismatch:**

```toml
# foundry.toml — pin compiler version to match target
[profile.default]
solc = "0.8.20"
# Or enable auto-detection
auto_detect_solc = true
```

**EVM version incompatibility:**

```toml
[profile.default]
evm_version = "shanghai"  # Options: london, paris, shanghai, cancun, prague, osaka (default: osaka)
```

**Remapping errors after dependency install:**

```bash
# Regenerate remappings
forge remappings > remappings.txt

# Verify correct paths
ls lib/
```

**Optimizer-related compilation failures:**

```toml
[profile.default]
optimizer = true
optimizer_runs = 200
via_ir = false  # Disable if causing issues
```

**Stack too deep errors:**

```toml
[profile.default]
via_ir = true
```

**Out-of-memory during compilation:**

```bash
# Limit parallelism
forge build --threads 1
```

**Clean rebuild:**

```bash
forge clean
forge build
```

---

## 4. Forge CLI Reference

### Essential Subcommands

| Command | Alias | Description |
|---------|-------|-------------|
| `forge build` | `b`, `compile` | Compile smart contracts |
| `forge test` | `t` | Run test suite |
| `forge script` | — | Execute contract as script |
| `forge install` | `i`, `add` | Install dependencies |
| `forge update` | `u` | Update dependencies |
| `forge remove` | `rm` | Remove dependencies |
| `forge remappings` | `re` | Show inferred remappings |
| `forge clean` | `cl` | Remove build artifacts and cache |
| `forge init` | — | Create new project |
| `forge flatten` | `f` | Flatten contract with imports into single file |
| `forge inspect` | `in` | Get contract ABI, bytecode, storage layout, etc. |
| `forge coverage` | — | Generate code coverage report |
| `forge snapshot` | `s` | Create gas snapshot per test |
| `forge tree` | `tr` | Dependency graph visualization |
| `forge config` | `co` | Display current configuration |
| `forge create` | `c` | Deploy a contract |
| `forge verify-contract` | `v` | Verify contract on Etherscan |
| `forge doc` | — | Generate NatSpec documentation |
| `forge selectors` | `se` | Function selector utilities |
| `forge cache` | — | Manage Foundry cache |
| `forge clone` | — | Retrieve contract from Etherscan |
| `forge fmt` | — | Format Solidity files |
| `forge eip712` | — | Generate EIP-712 struct encodings |

### Global Flags

| Flag | Description |
|------|-------------|
| `-v, --verbosity` | Increase output detail (stackable: `-v` to `-vvvvv`) |
| `-j, --threads <N>` | Set thread count (0 = all logical cores) |
| `-q, --quiet` | Suppress log output |
| `--json` | Output as JSON |
| `--color <auto\|always\|never>` | Control colored output |

### Inspect Subcommand

```bash
forge inspect <contract> <field>
```

All available fields (primary name listed first, common aliases in parentheses):

| Field | Aliases | Description |
|-------|---------|-------------|
| `abi` | — | Contract ABI |
| `bytecode` | `bytes`, `b` | Creation bytecode |
| `deployedBytecode` | `deployed-bytecode`, `deployed` | Runtime bytecode |
| `assembly` | `asm` | Assembly output |
| `assemblyOptimized` | `asmOptimized`, `asmo` | Optimized assembly |
| `legacyAssembly` | — | Legacy assembly JSON |
| `methodIdentifiers` | `methods`, `mi` | Function selectors |
| `gasEstimates` | `gas` | Gas estimates |
| `storageLayout` | `storage-layout`, `storage` | Storage layout |
| `devdoc` | `dev-doc` | Developer documentation |
| `userdoc` | `user-doc` | User documentation |
| `ir` | `IR` | Yul IR |
| `irOptimized` | `ir-optimized`, `iro` | Optimized Yul IR |
| `metadata` | `meta` | Contract metadata |
| `ewasm` | `e-wasm` | eWASM output |
| `errors` | `er` | Error selectors |
| `events` | `ev` | Event signatures and topics |
| `standardJson` | `standard-json` | Standard JSON input |
| `libraries` | `lib`, `libs` | Linked libraries |
| `linearization` | `linearizedBases` | C3 linearization order |

Auditor-relevant examples:

```bash
forge inspect src/Target.sol:Target abi
forge inspect src/Target.sol:Target storageLayout
forge inspect src/Target.sol:Target methodIdentifiers
forge inspect src/Target.sol:Target bytecode
forge inspect src/Target.sol:Target deployedBytecode
forge inspect src/Target.sol:Target errors
forge inspect src/Target.sol:Target events
forge inspect src/Target.sol:Target gasEstimates
```

---

## 5. Forge Test Command

### Syntax

```bash
forge test [OPTIONS]
```

### Verbosity Levels

| Flag | Output Level |
|------|-------------|
| (none) | Pass/fail summary only |
| `-v` | Test names |
| `-vv` | Print logs for all tests |
| `-vvv` | Execution traces for failing tests |
| `-vvvv` | Execution traces for all tests + setup traces for failing tests |
| `-vvvvv` | Execution and setup traces for all tests + storage changes and backtraces with line numbers |

For PoC development, always use `-vvvv` or `-vvvvv`.

### Test Filtering

```bash
# Match specific test function (short alias: --mt)
forge test --match-test test_ExploitReentrancy

# Match test contract name (short alias: --mc)
forge test --match-contract VaultExploitTest

# Match file path (short alias: --mp)
forge test --match-path test/exploits/Reentrancy.t.sol

# Regex pattern matching
forge test --match-test "test_Exploit.*"

# Exclude specific tests (short aliases: --nmt, --nmc, --nmp)
forge test --no-match-test test_Skip
forge test --no-match-contract HelperTest
forge test --no-match-path test/helpers/
```

### Forking Options

```bash
# Fork mainnet for tests (--fork-url is an alias for --rpc-url)
forge test --rpc-url https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY

# Fork at specific block
forge test --rpc-url $RPC_URL --fork-block-number 18000000
```

Or configure in `foundry.toml`:

```toml
[profile.default]
eth_rpc_url = "https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"
fork_block_number = 18000000
```

### Fuzz Testing

Configure in `foundry.toml`:

```toml
[fuzz]
runs = 1000
max_test_rejects = 65536
seed = "0x1234"
```

Use in tests:

```solidity
function testFuzz_Deposit(uint256 amount) public {
    vm.assume(amount > 0 && amount < 100 ether);
    // or: amount = bound(amount, 1, 100 ether);
    vault.deposit{value: amount}();
}
```

### Gas Reporting

```bash
forge test --gas-report
forge snapshot  # creates .gas-snapshot file
```

### Watch Mode

```bash
forge test --watch
```

---

## 6. Cheatcodes for PoC Development

Cheatcodes are special functions on the `Vm` interface accessed via `vm` in Forge tests. They manipulate EVM state for testing exploit scenarios.

### 6.1 Identity Spoofing

#### `vm.prank` — Spoof msg.sender for One Call

```solidity
function prank(address msgSender) external;
function prank(address msgSender, address txOrigin) external;
function prank(address msgSender, bool delegateCall) external;
function prank(address msgSender, address txOrigin, bool delegateCall) external;
```

Sets `msg.sender` for the **next call only**. Includes static calls but not delegate calls (unless `delegateCall = true`).

```solidity
vm.prank(owner);
vault.withdraw(100 ether);  // executes as owner
// msg.sender reverts to test contract after this call
```

#### `vm.startPrank` / `vm.stopPrank` — Spoof msg.sender for Multiple Calls

```solidity
function startPrank(address msgSender) external;
function startPrank(address msgSender, address txOrigin) external;
function startPrank(address msgSender, bool delegateCall) external;
function startPrank(address msgSender, address txOrigin, bool delegateCall) external;
function stopPrank() external;
```

Sets `msg.sender` for **all subsequent calls** until `stopPrank()`.

```solidity
vm.startPrank(attacker);
token.approve(address(vault), type(uint256).max);
vault.deposit{value: 10 ether}();
vault.withdraw(10 ether);
vm.stopPrank();
```

### 6.2 Balance Manipulation

#### `vm.deal` — Set ETH Balance

```solidity
function deal(address account, uint256 newBalance) external;
```

Sets an account's ETH balance (in wei).

```solidity
vm.deal(attacker, 100 ether);
assertEq(attacker.balance, 100 ether);
```

#### `deal` (StdCheats) — Set ERC20 Balance

```solidity
// From forge-std StdCheats (not a vm cheatcode)
deal(address token, address to, uint256 amount);
deal(address token, address to, uint256 amount, bool adjust);
```

Manipulates storage slots directly. May not work for tokens with non-standard storage layouts or rebasing tokens.

```solidity
deal(address(USDC), attacker, 1_000_000e6);
assertEq(USDC.balanceOf(attacker), 1_000_000e6);
```

### 6.3 Time & Block Manipulation

#### `vm.warp` — Set block.timestamp

```solidity
function warp(uint256 newTimestamp) external;
```

```solidity
vm.warp(block.timestamp + 7 days);
// Use vm.getBlockTimestamp() instead of block.timestamp when compiling with --via-ir
```

#### `vm.roll` — Set block.number

```solidity
function roll(uint256 newHeight) external;
```

```solidity
vm.roll(block.number + 100);
// Use vm.getBlockNumber() instead of block.number when compiling with --via-ir
```

### 6.4 Storage Manipulation

#### `vm.store` — Write to Storage Slot

```solidity
function store(address target, bytes32 slot, bytes32 value) external;
```

Directly writes a value to a contract's storage slot. Use `forge inspect <Contract> storage-layout` to identify slot positions.

```solidity
// Overwrite the owner variable at slot 0
vm.store(address(target), bytes32(uint256(0)), bytes32(uint256(uint160(attacker))));
```

#### `vm.load` — Read from Storage Slot

```solidity
function load(address target, bytes32 slot) external view returns (bytes32 data);
```

```solidity
bytes32 ownerSlot = vm.load(address(target), bytes32(uint256(0)));
address currentOwner = address(uint160(uint256(ownerSlot)));
```

### 6.5 Code Injection

#### `vm.etch` — Set Bytecode at Address

```solidity
function etch(address target, bytes calldata newRuntimeBytecode) external;
```

Deploys arbitrary runtime bytecode at any address. No constructor logic is executed.

```solidity
// Deploy a malicious contract at a specific address
MaliciousReceiver malicious = new MaliciousReceiver();
vm.etch(targetAddress, address(malicious).code);
```

Common use: inject mock precompiles for chain-specific testing (Arbitrum, Blast, etc.).

### 6.6 Forking Cheatcodes

#### `vm.createFork` — Create a Fork

```solidity
function createFork(string calldata urlOrAlias) external returns (uint256 forkId);
function createFork(string calldata urlOrAlias, uint256 blockNumber) external returns (uint256 forkId);
function createFork(string calldata urlOrAlias, bytes32 txHash) external returns (uint256 forkId);
```

Creates a fork but does **not** activate it. Use an RPC URL or alias from `foundry.toml` `[rpc_endpoints]`.

#### `vm.createSelectFork` — Create and Activate Fork

```solidity
function createSelectFork(string calldata urlOrAlias) external returns (uint256 forkId);
function createSelectFork(string calldata urlOrAlias, uint256 blockNumber) external returns (uint256 forkId);
function createSelectFork(string calldata urlOrAlias, bytes32 txHash) external returns (uint256 forkId);
```

Creates and immediately activates the fork.

```solidity
uint256 forkId = vm.createSelectFork("mainnet", 18_000_000);
// Now operating on mainnet state at block 18,000,000
```

#### `vm.selectFork` — Switch Active Fork

```solidity
function selectFork(uint256 forkId) external;
```

#### `vm.activeFork` — Get Current Fork ID

```solidity
function activeFork() external view returns (uint256 forkId);
```

#### `vm.makePersistent` — Persist State Across Forks

```solidity
function makePersistent(address account) external;
function makePersistent(address account0, address account1) external;
function makePersistent(address account0, address account1, address account2) external;
function makePersistent(address[] calldata accounts) external;
```

By default, each fork has isolated storage. `makePersistent` marks accounts whose state survives fork switches. The test contract and `msg.sender` are persistent by default.

```solidity
vm.makePersistent(address(attackerContract));
vm.selectFork(optimismFork);
// attackerContract state is preserved from the previous fork
```

#### `vm.rollFork` — Roll Fork to Different Block

```solidity
function rollFork(uint256 blockNumber) external;
function rollFork(bytes32 txHash) external;
function rollFork(uint256 forkId, uint256 blockNumber) external;
function rollFork(uint256 forkId, bytes32 txHash) external;
```

### 6.7 Call Mocking

#### `vm.mockCall` — Mock External Call Return Data

```solidity
function mockCall(address callee, bytes calldata data, bytes calldata returnData) external;
function mockCall(address callee, uint256 msgValue, bytes calldata data, bytes calldata returnData) external;
function mockCall(address callee, bytes4 data, bytes calldata returnData) external;
function mockCall(address callee, uint256 msgValue, bytes4 data, bytes calldata returnData) external;
```

Intercepts calls to `callee` matching `data` and returns `returnData`. Supports partial matching (selector-only via `bytes4` overloads).

```solidity
// Mock a price oracle to return a manipulated price
vm.mockCall(
    address(oracle),
    abi.encodeWithSelector(IOracle.getPrice.selector),
    abi.encode(1)  // Return price of 1 wei
);
```

**Important:** Target address must have bytecode. Use `vm.etch` first if needed. Persists until `vm.clearMockedCalls()`.

### 6.8 Assertions & Expectations

#### `vm.expectRevert` — Assert Next Call Reverts

```solidity
function expectRevert() external;
function expectRevert(bytes4 revertData) external;
function expectRevert(bytes calldata revertData) external;
function expectRevert(address reverter) external;
function expectRevert(bytes4 revertData, address reverter) external;
function expectRevert(bytes calldata revertData, address reverter) external;
function expectRevert(uint64 count) external;
function expectRevert(bytes4 revertData, uint64 count) external;
function expectRevert(bytes calldata revertData, uint64 count) external;
function expectRevert(bytes4 revertData, address reverter, uint64 count) external;
function expectRevert(bytes calldata revertData, address reverter, uint64 count) external;
function expectRevert(address reverter, uint64 count) external;
function expectPartialRevert(bytes4 revertData) external;
function expectPartialRevert(bytes4 revertData, address reverter) external;
```

```solidity
// Expect any revert
vm.expectRevert();
vault.withdraw(type(uint256).max);

// Expect specific error string
vm.expectRevert("insufficient balance");
vault.withdraw(1000 ether);

// Expect custom error selector
vm.expectRevert(Vault.InsufficientBalance.selector);
vault.withdraw(1000 ether);

// Expect custom error with parameters
vm.expectRevert(abi.encodeWithSelector(Vault.InsufficientBalance.selector, 100, 1000));
vault.withdraw(1000 ether);
```

#### `vm.expectEmit` — Assert Event Emission

```solidity
function expectEmit() external;
function expectEmit(address emitter) external;
function expectEmit(uint64 count) external;
function expectEmit(address emitter, uint64 count) external;
function expectEmit(bool checkTopic1, bool checkTopic2, bool checkTopic3, bool checkData) external;
function expectEmit(bool checkTopic1, bool checkTopic2, bool checkTopic3, bool checkData, address emitter) external;
function expectEmit(bool checkTopic1, bool checkTopic2, bool checkTopic3, bool checkData, uint64 count) external;
function expectEmit(bool checkTopic1, bool checkTopic2, bool checkTopic3, bool checkData, address emitter, uint64 count) external;
```

Topic 0 (event signature) is always checked. Workflow: call `expectEmit`, emit the expected event, then execute the call.

```solidity
vm.expectEmit(address(vault));
emit Vault.Withdrawal(attacker, 100 ether);
vault.withdraw(100 ether);
```

#### `vm.recordLogs` / `vm.getRecordedLogs` — Capture Events

```solidity
function recordLogs() external;
function getRecordedLogs() external view returns (Vm.Log[] memory logs);
```

```solidity
vm.recordLogs();
vault.deposit{value: 1 ether}();
Vm.Log[] memory entries = vm.getRecordedLogs();
assertEq(entries[0].topics[0], keccak256("Deposit(address,uint256)"));
```

### 6.9 Signing

#### `vm.sign` — Sign a Digest

```solidity
function sign(uint256 privateKey, bytes32 digest) external pure returns (uint8 v, bytes32 r, bytes32 s);
function sign(Wallet calldata wallet, bytes32 digest) external pure returns (uint8 v, bytes32 r, bytes32 s);
function sign(bytes32 digest) external pure returns (uint8 v, bytes32 r, bytes32 s);
function sign(address signer, bytes32 digest) external pure returns (uint8 v, bytes32 r, bytes32 s);
```

```solidity
(address alice, uint256 alicePk) = makeAddrAndKey("alice");
bytes32 hash = keccak256("Signed by Alice");
(uint8 v, bytes32 r, bytes32 s) = vm.sign(alicePk, hash);
address signer = ecrecover(hash, v, r, s);
assertEq(signer, alice);
```

**OpenZeppelin ECDSA caveat:** encode signature as `abi.encodePacked(r, s, v)` (not `v, r, s`).

### 6.10 Utility Cheatcodes

#### `vm.label` — Label Address in Traces

```solidity
function label(address account, string calldata newLabel) external;
```

```solidity
vm.label(address(vault), "Vault");
vm.label(attacker, "Attacker");
// Addresses show labels in -vvvv traces
```

### 6.11 Cheatcode Categories Summary

| Category | Key Cheatcodes |
|----------|---------------|
| **Identity** | `prank`, `startPrank`, `stopPrank` |
| **Balances** | `deal` |
| **Time/Block** | `warp`, `roll` |
| **Storage** | `store`, `load` |
| **Bytecode** | `etch` |
| **Forking** | `createFork`, `createSelectFork`, `selectFork`, `makePersistent`, `rollFork` |
| **Mocking** | `mockCall`, `clearMockedCalls` |
| **Assertions** | `expectRevert`, `expectEmit`, `expectPartialRevert` |
| **Events** | `recordLogs`, `getRecordedLogs` |
| **Signing** | `sign` |
| **Labels** | `label` |
| **Snapshots** | `snapshotState`, `revertToState` |
| **Gas** | `txGasPrice` |
| **Environment** | `setEnv`, `envOr`, `envString`, `envUint` |
| **Files** | `readFile`, `writeFile`, `readLine` |

---

## 7. Forge-std Helpers & Assertions

The `Test` base contract from `forge-std` provides helper functions beyond raw cheatcodes.

### Helper Functions

Functions below come from two sources: **VM cheatcodes** (native to forge) and **forge-std Solidity wrappers** (from the `forge-std` library, which internally call VM cheatcodes).

| Function | Source | Description |
|----------|--------|-------------|
| `makeAddr(string)` | forge-std | Generate deterministic address from a label (calls `vm.addr` + `vm.label`) |
| `makeAddrAndKey(string)` | forge-std | Generate address + private key pair; returns `(address, uint256)` |
| `hoax(address)` | forge-std | `prank` + `deal` combined (single call) |
| `startHoax(address)` | forge-std | `startPrank` + `deal` combined |
| `deal(address token, address to, uint256 amount)` | forge-std | Set ERC20 token balance via storage manipulation |
| `deployCode(string)` | VM cheatcode | Deploy contract from artifact name (8 overloads with constructorArgs, value, salt) |
| `bound(uint256 val, uint256 min, uint256 max)` | VM cheatcode | Clamp fuzz input to range (also has `int256` overload) |
| `skip(bool)` | VM cheatcode | Conditionally skip test (also has `skip(bool, string)` overload) |
| `noGasMetering` | forge-std modifier | Disable gas metering for a function (wraps `vm.pauseGasMetering` / `vm.resumeGasMetering`) |

**Note:** `changePrank(address)` was a forge-std helper but has been deprecated in favor of `vm.stopPrank()` + `vm.startPrank(newAddress)`.

### Assertion Functions

All assertions below are VM cheatcodes (available via the `Vm` interface) unless noted. They also have `Decimal` variants (e.g., `assertEqDecimal`, `assertGtDecimal`) for formatted output.

| Function | Description |
|----------|-------------|
| `assertEq(a, b)` | Assert equality (uint, int, address, bytes, string, bool, arrays) |
| `assertNotEq(a, b)` | Assert inequality |
| `assertGt(a, b)` | Assert `a > b` |
| `assertGe(a, b)` | Assert `a >= b` |
| `assertLt(a, b)` | Assert `a < b` |
| `assertLe(a, b)` | Assert `a <= b` |
| `assertTrue(condition)` | Assert true |
| `assertFalse(condition)` | Assert false (VM cheatcode only, not in legacy DSTest) |
| `assertApproxEqAbs(a, b, delta)` | Assert `\|a - b\| <= delta` |
| `assertApproxEqRel(a, b, pct)` | Assert relative difference within percentage |
| `fail()` | Explicitly fail the test (DSTest internal function, not a VM cheatcode) |

All assertion functions accept an optional trailing `string` parameter for custom error messages:

```solidity
assertGt(attacker.balance, initialBalance, "Exploit failed: attacker balance did not increase");
```

---

## 8. Mainnet Forking

### In-Test Forking (Cheatcodes)

```solidity
function setUp() public {
    // Fork mainnet at specific block
    vm.createSelectFork(vm.rpcUrl("mainnet"), 18_000_000);
    
    // Reference deployed contracts
    IERC20 usdc = IERC20(0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48);
    IUniswapV3Pool pool = IUniswapV3Pool(0x8ad599c3A0ff1De082011EFDDc58f1908eb6e6D8);
}
```

### CLI Forking

```bash
forge test --rpc-url $ETH_RPC_URL --fork-block-number 18000000 -vvvv
```

### foundry.toml Fork Configuration

```toml
[profile.default]
eth_rpc_url = "https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"
fork_block_number = 18000000

[rpc_endpoints]
mainnet = "${ETH_RPC_URL}"
arbitrum = "https://arb1.arbitrum.io/rpc"
optimism = "https://mainnet.optimism.io"
polygon = "https://polygon-rpc.com"
bsc = "https://bsc-dataseed.binance.org"
```

### Multi-Chain Fork Testing

```solidity
function testCrossChainExploit() public {
    uint256 ethFork = vm.createSelectFork("mainnet", 18_000_000);
    uint256 arbFork = vm.createFork("arbitrum", 150_000_000);
    
    // Deploy attacker contract on mainnet fork
    AttackerContract attacker = new AttackerContract();
    vm.makePersistent(address(attacker));
    
    // Switch to Arbitrum
    vm.selectFork(arbFork);
    // attacker contract state persists across forks
}
```

---

## 9. Anvil Local Testnet

### Basic Usage

```bash
# Start local node
anvil

# Fork mainnet
anvil --fork-url $ETH_RPC_URL

# Fork at specific block
anvil --fork-url $ETH_RPC_URL --fork-block-number 18000000
```

### Key Flags

| Flag | Default | Description |
|------|---------|-------------|
| `-p, --port <PORT>` | 8545 | Listen port |
| `--host <IP>` | 127.0.0.1 | Server host |
| `-a, --accounts <NUM>` | 10 | Number of dev accounts |
| `--balance <NUM>` | 10000 | Initial ETH per account |
| `-m, --mnemonic <PHRASE>` | — | BIP39 mnemonic for accounts |
| `--chain-id <ID>` | 31337 | Chain ID |
| `--hardfork <NAME>` | — | EVM version (cancun, shanghai, paris, london, prague) |
| `-b, --block-time <SEC>` | — | Auto-mine interval (instant if omitted) |
| `--no-mining` | — | Disable auto-mining |
| `-f, --fork-url <URL>` | — | Remote RPC endpoint to fork |
| `--fork-block-number <N>` | latest | Block number to fork from |
| `--auto-impersonate` | — | Allow impersonating any account |
| `--gas-limit <LIMIT>` | — | Block gas limit |
| `--state <PATH>` | — | Load/dump state from file |
| `--dump-state <PATH>` | — | Dump state on exit |
| `--load-state <PATH>` | — | Load state from snapshot |

### Auditor Workflow with Anvil

```bash
# Terminal 1: Start forked node
anvil --fork-url $ETH_RPC_URL --fork-block-number 18000000 --auto-impersonate

# Terminal 2: Interact with the forked chain
cast call 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 "balanceOf(address)" 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url http://127.0.0.1:8545
```

---

## 10. Cast CLI for Chain Interaction

### Transaction & Call Commands

| Command | Description |
|---------|-------------|
| `cast call <addr> <sig> [args]` | Read-only call (no tx) |
| `cast send <addr> <sig> [args]` | Sign and send transaction |
| `cast estimate <addr> <sig> [args]` | Estimate gas cost |
| `cast receipt <tx_hash>` | Get transaction receipt |
| `cast tx <tx_hash>` | Get transaction info |
| `cast run <tx_hash>` | Replay transaction locally |

### Storage & State Inspection

| Command | Description |
|---------|-------------|
| `cast storage <addr> <slot>` | Read raw storage slot |
| `cast balance <addr>` | Get ETH balance (wei) |
| `cast nonce <addr>` | Get account nonce |
| `cast code <addr>` | Get runtime bytecode |
| `cast codesize <addr>` | Get bytecode size |
| `cast implementation <addr>` | Get EIP-1967 implementation address |
| `cast admin <addr>` | Get EIP-1967 admin address |

### ABI & Encoding

| Command | Description |
|---------|-------------|
| `cast sig <sig_string>` | Get 4-byte function selector |
| `cast abi-encode <sig> [args]` | ABI-encode function args |
| `cast decode-abi <sig> <data>` | Decode ABI data |
| `cast calldata <sig> [args]` | Encode full calldata |
| `cast decode-calldata <sig> <data>` | Decode calldata |
| `cast 4byte <selector>` | Look up function signature |
| `cast 4byte-calldata <data>` | Decode calldata by selector |
| `cast interface <addr>` | Generate Solidity interface from on-chain ABI |
| `cast source <addr>` | Fetch verified source from Etherscan |

### Block & Chain

| Command | Description |
|---------|-------------|
| `cast block <block_num>` | Get block info |
| `cast block-number` | Get latest block number |
| `cast chain-id` | Get chain ID |
| `cast gas-price` | Get current gas price |
| `cast base-fee` | Get block base fee |
| `cast logs <sig> [args]` | Get event logs by signature |

### Utility

| Command | Description |
|---------|-------------|
| `cast to-wei <value> <unit>` | Convert to wei |
| `cast from-wei <value>` | Convert from wei |
| `cast keccak <data>` | Keccak-256 hash |
| `cast index <key_type> <key> <slot>` | Compute mapping storage slot |
| `cast index-erc7201 <id>` | Compute ERC-7201 storage slot |
| `cast proof <addr> <slots>` | Generate storage proof |
| `cast create2` | Compute CREATE2 address |
| `cast disassemble <bytecode>` | Disassemble bytecode |
| `cast access-list <addr> <sig>` | Create access list for tx |

### Cast Examples for Auditors

```bash
# Read a storage slot of a deployed contract
cast storage 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 0 --rpc-url $ETH_RPC_URL

# Compute a mapping slot (e.g., balances[address] at slot 9)
cast index address 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 9

# Get a contract's ABI as a Solidity interface
cast interface 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 --rpc-url $ETH_RPC_URL

# Decode calldata from a transaction
cast 4byte-calldata 0xa9059cbb000000000000000000000000...

# Get EIP-1967 proxy implementation
cast implementation 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 --rpc-url $ETH_RPC_URL
```

---

## 11. Configuration Reference (foundry.toml)

### Minimal Audit Configuration

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc = "0.8.20"
evm_version = "shanghai"
optimizer = true
optimizer_runs = 200
via_ir = false
ffi = false
fs_permissions = [{ access = "read", path = "./"}]

[rpc_endpoints]
mainnet = "${ETH_RPC_URL}"
arbitrum = "${ARB_RPC_URL}"
optimism = "${OP_RPC_URL}"

[fuzz]
runs = 256
max_test_rejects = 65536

[profile.default.fmt]
line_length = 120
tab_width = 4
```

### Key Configuration Options

| Setting | Description |
|---------|-------------|
| `src` | Source contract directory |
| `out` | Build output directory |
| `libs` | Dependency directories |
| `solc` | Pin Solidity compiler version (`solc_version` is a deprecated alias) |
| `auto_detect_solc` | Auto-detect solc from pragmas |
| `evm_version` | Target EVM version |
| `optimizer` | Enable Solidity optimizer |
| `optimizer_runs` | Optimizer runs parameter |
| `via_ir` | Use IR-based compilation pipeline |
| `remappings` | Import path remappings |
| `ffi` | Allow FFI (foreign function interface) in tests |
| `eth_rpc_url` | Default RPC endpoint |
| `fork_block_number` | Default fork block |
| `fs_permissions` | File system access rules |

---

## 12. PoC Development Workflow

### Step-by-Step Process

1. **Set up the environment**
   ```bash
   forge init exploit-poc
   cd exploit-poc
   forge install OpenZeppelin/openzeppelin-contracts  # if needed
   ```

2. **Place target contracts in `src/`**

3. **Configure `foundry.toml`** (compiler version, remappings, RPC endpoints)

4. **Compile and verify**
   ```bash
   forge build
   ```

5. **Write the PoC test in `test/`** (see template below)

6. **Execute with full traces**
   ```bash
   forge test --match-test test_Exploit -vvvv
   ```

7. **Debug failing tests**
   ```bash
   forge test --match-test test_Exploit -vvvvv  # includes storage changes
   forge test --match-test test_Exploit --debug  # step-through debugger
   ```

8. **Validate on mainnet fork** (for deployed contracts)
   ```bash
   forge test --match-test test_Exploit --rpc-url $ETH_RPC_URL --fork-block-number 18000000 -vvvv
   ```

---

## 13. PoC Template

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
        // Record pre-exploit state
        uint256 attackerBalBefore = ATTACKER.balance;
        uint256 targetBalBefore = address(target).balance;
        console.log("Pre-exploit attacker balance:", attackerBalBefore);
        console.log("Pre-exploit target balance:", targetBalBefore);

        // Execute exploit
        vm.startPrank(ATTACKER);

        // --- EXPLOIT LOGIC HERE ---

        vm.stopPrank();

        // Validate exploit success
        uint256 attackerBalAfter = ATTACKER.balance;
        uint256 targetBalAfter = address(target).balance;
        console.log("Post-exploit attacker balance:", attackerBalAfter);
        console.log("Post-exploit target balance:", targetBalAfter);

        assertGt(attackerBalAfter, attackerBalBefore, "Exploit failed: attacker did not profit");
        assertLt(targetBalAfter, targetBalBefore, "Exploit failed: target funds not drained");
    }
}
```

### Run the PoC

```bash
forge test --match-test test_Exploit -vvvv
```

---

## 14. Invariant Testing for Audits

Invariant testing (also called stateful fuzz testing) is one of the most powerful techniques for finding complex, multi-step vulnerabilities that single-function fuzz tests miss. Foundry's invariant tester randomly calls sequences of functions and checks that protocol invariants hold after every call.

### Core Concepts

- **Invariant functions** are prefixed with `invariant_` — they are called after every random function sequence
- **Handler contracts** wrap target contract calls to constrain inputs to realistic values and manage test state
- **Ghost variables** track expected state in the test so you can compare against actual contract state

### Basic Invariant Test

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/Vault.sol";

contract VaultInvariantTest is Test {
    Vault public vault;
    IERC20 public token;

    function setUp() public {
        token = new MockERC20();
        vault = new Vault(address(token));

        // Target only the vault for random calls
        targetContract(address(vault));
    }

    // This is checked after EVERY random call sequence
    invariant_totalAssetsMatchesBalance() public view {
        assertGe(
            token.balanceOf(address(vault)),
            vault.totalAssets(),
            "Vault claims more assets than it holds"
        );
    }

    invariant_sharePriceNeverZero() public view {
        if (vault.totalSupply() > 0) {
            assertGt(
                vault.convertToAssets(1e18),
                0,
                "Share price collapsed to zero"
            );
        }
    }
}
```

### Handler Pattern (Recommended for Real Audits)

Without a handler, the fuzzer calls random functions with random arguments — most calls revert on invalid inputs and the test explores very little state space. A handler constrains inputs and tracks state.

```solidity
contract VaultHandler is Test {
    Vault public vault;
    IERC20 public token;

    // Ghost variables — track expected state
    uint256 public ghost_totalDeposited;
    uint256 public ghost_totalWithdrawn;
    address[] public actors;

    constructor(Vault _vault, IERC20 _token) {
        vault = _vault;
        token = _token;

        // Create a set of actors
        for (uint256 i = 0; i < 5; i++) {
            actors.push(makeAddr(string(abi.encodePacked("actor", i))));
        }
    }

    // Modifiers to bound inputs and select actors
    modifier useActor(uint256 actorSeed) {
        address actor = actors[bound(actorSeed, 0, actors.length - 1)];
        vm.startPrank(actor);
        _;
        vm.stopPrank();
    }

    function deposit(uint256 amount, uint256 actorSeed) external useActor(actorSeed) {
        amount = bound(amount, 1, 100_000e18);

        address actor = actors[bound(actorSeed, 0, actors.length - 1)];
        deal(address(token), actor, amount);
        token.approve(address(vault), amount);
        vault.deposit(amount, actor);

        ghost_totalDeposited += amount;
    }

    function withdraw(uint256 amount, uint256 actorSeed) external useActor(actorSeed) {
        address actor = actors[bound(actorSeed, 0, actors.length - 1)];
        uint256 maxWithdraw = vault.maxWithdraw(actor);
        amount = bound(amount, 0, maxWithdraw);
        if (amount == 0) return;

        vault.withdraw(amount, actor, actor);

        ghost_totalWithdrawn += amount;
    }
}

contract VaultInvariantTest is Test {
    Vault public vault;
    MockERC20 public token;
    VaultHandler public handler;

    function setUp() public {
        token = new MockERC20();
        vault = new Vault(address(token));
        handler = new VaultHandler(vault, token);

        // Target ONLY the handler — not the vault directly
        targetContract(address(handler));
    }

    invariant_solvency() public view {
        assertGe(
            token.balanceOf(address(vault)),
            vault.totalAssets(),
            "Vault is insolvent"
        );
    }

    invariant_depositWithdrawAccounting() public view {
        assertGe(
            handler.ghost_totalDeposited(),
            handler.ghost_totalWithdrawn(),
            "More withdrawn than deposited"
        );
    }

    // Called after the invariant test completes — useful for final assertions
    function afterInvariant() public view {
        // Log final state for analysis
        console.log("Total calls executed");
        console.log("Final vault balance:", token.balanceOf(address(vault)));
    }
}
```

### Configuration Functions

Call these in `setUp()` to control what gets fuzzed:

| Function | Description |
|----------|-------------|
| `targetContract(address)` | Add contract to the set of targets (fuzzer only calls targets) |
| `excludeContract(address)` | Remove contract from target set |
| `targetSelector(FuzzSelector)` | Restrict which functions can be called on a target |
| `excludeSender(address)` | Prevent an address from being used as `msg.sender` |
| `targetArtifact(string)` | Target a contract by artifact name |
| `targetArtifactSelector(FuzzArtifactSelector)` | Target specific functions in an artifact |

```solidity
// Only fuzz specific functions
bytes4[] memory selectors = new bytes4[](2);
selectors[0] = VaultHandler.deposit.selector;
selectors[1] = VaultHandler.withdraw.selector;
targetSelector(FuzzSelector({addr: address(handler), selectors: selectors}));
```

### foundry.toml Configuration

```toml
[invariant]
runs = 256              # Number of call sequences to execute
depth = 128             # Number of calls per sequence
fail_on_revert = false  # Don't fail if a call reverts (expected with random inputs)
call_override = false   # Don't override msg.sender for calls
dictionary_weight = 80  # Weight for using values from storage vs random (0-100)
shrink_run_limit = 5000 # Max attempts to shrink failing sequence
```

### Common Invariants for DeFi Audits

| Protocol Type | Invariant | What It Catches |
|---------------|-----------|-----------------|
| **Vault/ERC4626** | `totalAssets >= sum(user shares × share price)` | Inflation attacks, accounting bugs |
| **Vault/ERC4626** | `share price is monotonically non-decreasing (outside loss events)` | Rounding exploits, donation attacks |
| **Lending** | `total borrows <= total deposits` | Over-borrowing bugs |
| **Lending** | `user health factor >= 1 OR position is liquidatable` | Liquidation threshold bugs |
| **AMM/DEX** | `x * y >= k (constant product)` | Price manipulation, rounding errors |
| **AMM/DEX** | `sum(LP token supply) corresponds to reserves` | LP accounting bugs |
| **Staking** | `total staked == sum(individual stakes)` | Reward distribution errors |
| **Governance** | `total voting power == sum(individual voting power)` | Vote manipulation |

### Run Invariant Tests

```bash
# Run all invariant tests
forge test --match-test invariant_ -vvv

# Run with more depth (longer sequences)
forge test --match-test invariant_ -vvv --fuzz-runs 1000
```
