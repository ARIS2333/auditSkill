# Slither Static Analysis Audit Reference Guide

Official GitHub: https://github.com/crytic/slither

Comprehensive reference for smart contract auditors using Slither for static vulnerability analysis, contract inspection, and audit reporting. All commands, detectors, printers, and severity classifications verified against the [Slither source code](https://github.com/crytic/slither) (latest `master` as of April 2026).

---

## Table of Contents

1. [Installation](#1-installation)
2. [Basic Usage & Framework Integration](#2-basic-usage--framework-integration)
3. [CLI Flags Reference](#3-cli-flags-reference)
4. [Vulnerability Detection (Detectors)](#4-vulnerability-detection-detectors)
5. [Contract Inspection (Printers)](#5-contract-inspection-printers)
6. [Targeted Scan Strategies](#6-targeted-scan-strategies)
7. [False Positive Management](#7-false-positive-management)
8. [JSON & SARIF Output](#8-json--sarif-output)
9. [Upgradeability Analysis](#9-upgradeability-analysis)
10. [Additional Slither Tools](#10-additional-slither-tools)
11. [Python API for Custom Analysis](#11-python-api-for-custom-analysis)
12. [Custom Detector Development](#12-custom-detector-development)
13. [Detector-to-PoC Mapping](#13-detector-to-poc-mapping)
14. [Known Limitations](#14-known-limitations)

---

## 1. Installation

### Using uv (Recommended)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv tool install slither-analyzer
```

### Using pip

```bash
python3 -m pip install slither-analyzer
```

### Using Homebrew (macOS)

```bash
brew install slither-analyzer
```

### From Source (Development)

```bash
git clone https://github.com/crytic/slither.git && cd slither
uv tool install -e .
```

### Docker (Trail of Bits Security Toolbox)

```bash
docker pull trailofbits/eth-security-toolbox
docker run -it -v $(pwd):/share trailofbits/eth-security-toolbox
```

### Requirements

- Python 3.10+
- Solidity compiler (`solc`) unless using a supported build framework

### Verify Installation

```bash
slither --version
```

---

## 2. Basic Usage & Framework Integration

### Foundry Projects

```bash
slither .
# Or explicitly specify the output directory
slither . --foundry-out-directory out
```

### Hardhat Projects

```bash
slither .
```

### Truffle / Dapp / Etherlime Projects

```bash
slither .
```

### Embark Projects

```bash
# First run only
slither . --embark-overwrite-config
# Subsequent runs
slither .
```

### Single Solidity File

```bash
slither file.sol
```

### Etherscan Verified Contract

```bash
slither 0xContractAddress --etherscan-apikey YOUR_KEY
```

### AST Input

```bash
slither file.ast.json
```

---

## 3. CLI Flags Reference

### Detector Control

| Flag | Description |
|------|-------------|
| `--detect d1,d2,...` | Run only specified detectors |
| `--exclude d1,d2,...` | Exclude specified detectors |
| `--include-detectors d1,d2,...` | Include only specified detectors (additive) |
| `--exclude-dependencies` | Exclude findings from dependency contracts |
| `--exclude-informational` | Skip all informational findings |
| `--exclude-low` | Skip all low severity findings |
| `--exclude-medium` | Skip all medium severity findings |
| `--exclude-high` | Skip all high severity findings |
| `--exclude-optimization` | Skip all optimization findings |
| `--list-detectors` | List all available detectors |
| `--show-ignored-findings` | Show findings even if suppressed |

### Printer Control

| Flag | Description |
|------|-------------|
| `--print p1,p2,...` | Run specified printers |
| `--list-printers` | List all available printers |
| `--include-interfaces` | Include interfaces in inheritance-graph printer output |

### Output Formatting

| Flag | Description |
|------|-------------|
| `--json <file>` | Export results as JSON (use `-` for stdout) |
| `--json-types <types>` | Filter JSON output by result types |
| `--sarif <file>` | Export results as SARIF JSON (use `-` for stdout) |
| `--sarif-input <file>` | SARIF input (beta) |
| `--sarif-triage <file>` | SARIF triage (beta) |
| `--zip <file>` | Export results as zipped JSON |
| `--zip-type <type>` | Zip compression type (lzma, stored, deflated, bzip2) |
| `--checklist` | Output as Markdown checklist |
| `--checklist-limit <n>` | Limit results per detector in Markdown output |
| `--markdown-root <url>` | Base URL for Markdown generation |
| `--disable-color` | Disable colored terminal output |
| `--solc-disable-warnings` | Suppress solc compiler warnings |
| `--generate-patches` | Generate patches (JSON output only) |

### Path Filtering

| Flag | Description |
|------|-------------|
| `--filter-paths <regex>` | Exclude results from matching paths (e.g., `'mocks/\|test/'`) — always use single quotes around the regex to prevent shell pipe interpretation |
| `--include-paths <regex>` | Include only results from matching paths (opposite of `--filter-paths`) |

### CI / Exit Code Control

| Flag | Description |
|------|-------------|
| `--fail-pedantic` | Fail if any findings are detected (default) |
| `--fail-low` | Fail if any low or greater impact findings detected |
| `--fail-medium` | Fail if any medium or greater impact findings detected |
| `--fail-high` | Fail if any high impact findings detected |
| `--fail-none` | Do not return finding count in exit code |
| `--no-fail` | Do not fail on parsing errors (echidna mode) |

### Triage & Configuration

| Flag | Description |
|------|-------------|
| `--triage-mode` | Interactive triage — saves decisions to triage database |
| `--triage-database <file>` | Custom triage database path (default: `slither.db.json`) |
| `--config-file <file>` | Custom config file (default: `slither.config.json`) |

### Framework-Specific (via crytic-compile)

| Flag | Description |
|------|-------------|
| `--foundry-out-directory <dir>` | Foundry build output directory (default: `out`) |
| `--foundry-ignore-compile` | Do not run `forge build` |
| `--foundry-compile-all` | Don't skip compiling test and script |
| `--hardhat-cache-directory <dir>` | Hardhat cache directory (default: `./cache`) |
| `--hardhat-artifacts-directory <dir>` | Hardhat artifacts directory (default: `./artifacts`) |
| `--hardhat-ignore-compile` | Do not run `hardhat compile` |
| `--compile-force-framework <name>` | Force compilation framework (foundry, hardhat, truffle, etc.) |
| `--compile-remove-metadata` | Remove metadata from bytecodes |
| `--ignore-compile` | Do not run any platform compilation |
| `--solc <path>` | Path to solc binary |
| `--solc-remaps <remaps>` | Add import remappings |
| `--solc-args <args>` | Custom solc arguments |
| `--etherscan-apikey <key>` | Etherscan API key for verified contract analysis |

### Miscellaneous

| Flag | Description |
|------|-------------|
| `--solc-ast` | Provide the contract as a JSON AST |
| `--change-line-prefix <char>` | Change the line prefix for displayed source code (default: `#`) |

---

## 4. Vulnerability Detection (Detectors)

Slither includes **99** built-in detectors (excluding the example `backdoor` detector). Below is the complete list organized by impact severity, with all classifications verified against slither 0.11.4.

### High Impact Detectors (28)

| Detector ID | Description | Confidence |
|-------------|-------------|------------|
| `abiencoderv2-array` | ABI encoder v2 array bug (solc 0.4.7–0.5.9) | High |
| `arbitrary-send-erc20` | `transferFrom` uses arbitrary `from` (not `msg.sender`) | High |
| `arbitrary-send-erc20-permit` | Arbitrary `from` in `transferFrom` with `permit` | Medium |
| `arbitrary-send-eth` | Unprotected ETH transfer to arbitrary destination | Medium |
| `array-by-reference` | Storage array passed by value to internal function | High |
| `controlled-array-length` | Direct array length assignment enables storage manipulation | Medium |
| `controlled-delegatecall` | User-controlled `delegatecall` destination | Medium |
| `delegatecall-loop` | `delegatecall` inside loop in payable function | Medium |
| `encode-packed-collision` | `abi.encodePacked` hash collision with dynamic types | High |
| `incorrect-exp` | Bitwise XOR `^` used instead of exponentiation `**` | Medium |
| `incorrect-return` | Assembly `return` halts execution unexpectedly | Medium |
| `incorrect-shift` | Reversed shift operation parameters | High |
| `msg-value-loop` | `msg.value` used inside loop (double-spend risk) | Medium |
| `multiple-constructors` | Multiple constructor definitions (old + new syntax) | High |
| `name-reused` | Duplicate contract names prevent correct artifact generation | High |
| `protected-vars` | Protected variables lack proper access control | High |
| `public-mappings-nested` | Public nested mapping returns wrong values (pre-0.5) | High |
| `reentrancy-eth` | Reentrancy vulnerability with ETH transfer | Medium |
| `return-leave` | `return` used instead of `leave` in assembly | Medium |
| `rtlo` | Right-to-left override character manipulation | High |
| `shadowing-state` | State variable shadowed in derived contract | High |
| `storage-array` | Compiler bug with signed integer arrays in storage | Medium |
| `suicidal` | Unprotected `selfdestruct` / `suicide` | High |
| `unchecked-transfer` | Unchecked `transfer`/`transferFrom` return value | Medium |
| `uninitialized-state` | State variable never initialized | High |
| `uninitialized-storage` | Uninitialized storage pointer overwrites state | High |
| `unprotected-upgrade` | Upgradeable implementation can be destroyed | High |
| `weak-prng` | Weak PRNG from `block.timestamp` / `block.number` | Medium |

### Medium Impact Detectors (28)

| Detector ID | Description | Confidence |
|-------------|-------------|------------|
| `boolean-cst` | Misuse of boolean constant in conditional | Medium |
| `chronicle-unchecked-price` | Chronicle oracle price not validated | Medium |
| `constant-function-asm` | Constant/pure/view function uses assembly | Medium |
| `constant-function-state` | Constant/pure/view function modifies state | Medium |
| `divide-before-multiply` | Division before multiplication (precision loss) | Medium |
| `domain-separator-collision` | Function selector collides with EIP-2612 `DOMAIN_SEPARATOR` | High |
| `enum-conversion` | Out-of-range enum conversion (solc < 0.4.5) | High |
| `erc20-interface` | ERC20 functions missing return values | High |
| `erc721-interface` | ERC721 functions missing return values | High |
| `gelato-unprotected-randomness` | Unprotected randomness request | Medium |
| `incorrect-equality` | Strict equality check easily manipulable | High |
| `locked-ether` | Payable contract with no withdrawal mechanism | High |
| `mapping-deletion` | Deleting struct with mapping doesn't delete mapping | High |
| `out-of-order-retryable` | Out-of-order retryable transactions | Medium |
| `pyth-deprecated-functions` | Deprecated Pyth protocol function usage | High |
| `pyth-unchecked-confidence` | Pyth oracle confidence interval not checked | High |
| `pyth-unchecked-publishtime` | Pyth oracle `publishTime` not validated | High |
| `reentrancy-no-eth` | Reentrancy without ETH involvement | Medium |
| `reused-constructor` | Base constructor called from multiple paths | Medium |
| `shadowing-abstract` | State variable shadows abstract contract | High |
| `tautological-compare` | Variable compared to itself | High |
| `tautology` | Tautological or contradictory expression | High |
| `tx-origin` | `tx.origin` used for authorization | Medium |
| `unchecked-lowlevel` | Unchecked low-level call return value | Medium |
| `unchecked-send` | Unchecked `send` return value | Medium |
| `uninitialized-local` | Uninitialized local variable | Medium |
| `unused-return` | External call return value not used | Medium |
| `write-after-write` | Variable written then overwritten without read | High |

### Low Impact Detectors (17)

| Detector ID | Description | Confidence |
|-------------|-------------|------------|
| `calls-loop` | External calls inside loop (DoS risk) | Medium |
| `chainlink-feed-registry` | Chainlink Feed Registry (mainnet only) | High |
| `events-access` | Missing events for access control changes | Medium |
| `events-maths` | Missing events for critical math parameters | Medium |
| `incorrect-modifier` | Modifier doesn't execute `_` or revert | High |
| `incorrect-unary` | Dangerous unary expression (e.g., `=+` typo) | Medium |
| `missing-zero-check` | Missing zero-address validation | Medium |
| `optimism-deprecation` | Deprecated Optimism predeploy/function | High |
| `reentrancy-benign` | Reentrancy with equivalent sequential effect | Medium |
| `reentrancy-events` | Reentrancy manipulates event ordering | Medium |
| `return-bomb` | Low-level callee consumes all caller gas | Medium |
| `shadowing-builtin` | Shadowing built-in Solidity symbols | High |
| `shadowing-local` | Local variable shadows outer scope | High |
| `timestamp` | Dangerous `block.timestamp` usage | Medium |
| `uninitialized-fptr-cst` | Uninitialized function pointer in constructor | High |
| `variable-scope` | Variable used before declaration | High |
| `void-cst` | Constructor call with no implementation | High |

### Informational Detectors (21)

| Detector ID | Description | Confidence |
|-------------|-------------|------------|
| `assembly` | Assembly usage detected | High |
| `assert-state-change` | `assert()` modifies state | High |
| `boolean-equal` | Unnecessary comparison to boolean constant | High |
| `costly-loop` | Expensive operations inside loops | Medium |
| `cyclomatic-complexity` | High cyclomatic complexity (>11) | High |
| `dead-code` | Unreachable/unused functions | Medium |
| `deprecated-standards` | Deprecated Solidity constructs (`sha3`, `throw`, etc.) | High |
| `erc20-indexed` | ERC20 events missing `indexed` keyword | High |
| `function-init-state` | State initialized from non-pure function call | High |
| `incorrect-using-for` | Incompatible `using for` library binding | High |
| `low-level-calls` | Low-level call usage | High |
| `missing-inheritance` | Missing interface/base contract inheritance | High |
| `naming-convention` | Solidity naming convention violations | High |
| `pragma` | Inconsistent `pragma` directives across files | High |
| `redundant-statements` | Statements with no effect | High |
| `reentrancy-unlimited-gas` | Reentrancy via `transfer`/`send` | Medium |
| `solc-version` | Outdated Solidity compiler version | High |
| `too-many-digits` | Numeric literals with excessive digits | Medium |
| `unimplemented-functions` | Unimplemented interface functions | High |
| `unindexed-event-address` | Event `address` parameters not indexed | High |
| `unused-state` | Unused state variables | High |

### Optimization Detectors (5)

| Detector ID | Description | Confidence |
|-------------|-------------|------------|
| `cache-array-length` | Array `.length` not cached in for-loop | High |
| `constable-states` | State variable could be `constant` | High |
| `external-function` | Public function never called internally (use `external`) | High |
| `immutable-states` | State variable could be `immutable` | High |
| `var-read-using-this` | Contract reads own variable via `this` | High |

---

## 5. Contract Inspection (Printers)

Slither includes **27** printers that extract structural information from contracts.

### Complete Printer List

| Printer | Command | Output |
|---------|---------|--------|
| `call-graph` | `--print call-graph` | Export call graph as DOT file |
| `cfg` | `--print cfg` | Export control flow graph per function |
| `cheatcode` | `--print cheatcode` | Print usage of Foundry cheatcodes in the code |
| `ck` | `--print ck` | Chidamber and Kemerer (CK) complexity metrics |
| `constructor-calls` | `--print constructor-calls` | Print constructors executed during deployment |
| `contract-summary` | `--print contract-summary` | Summary of all contracts |
| `data-dependency` | `--print data-dependency` | Variable data dependency analysis |
| `declaration` | `--print declaration` | Source code declaration, implementation, and references |
| `dominator` | `--print dominator` | Export dominator tree per function |
| `echidna` | `--print echidna` | Export Echidna fuzzer guidance info |
| `entry-points` | `--print entry-points` | All state-changing entry point functions and their variables |
| `evm` | `--print evm` | Print EVM instructions per function node |
| `function-id` | `--print function-id` | Print keccak256 function selectors |
| `function-summary` | `--print function-summary` | Visibility, modifiers, state vars read/written per function |
| `halstead` | `--print halstead` | Halstead complexity metrics per contract |
| `human-summary` | `--print human-summary` | Human-readable contract summary |
| `inheritance` | `--print inheritance` | Print inheritance relations |
| `inheritance-graph` | `--print inheritance-graph` | Generate inheritance visualization (DOT) |
| `loc` | `--print loc` | Lines of code metrics (LOC, SLOC, CLOC for src/dep/test) |
| `martin` | `--print martin` | Martin agile software metrics (Ca, Ce, I, A, D) |
| `modifiers` | `--print modifiers` | Print modifiers called by each function |
| `not-pausable` | `--print not-pausable` | Print functions that do not use `whenNotPaused` |
| `require` | `--print require` | Print `require` and `assert` calls per function |
| `slithir` | `--print slithir` | Print SlithIR intermediate representation |
| `slithir-ssa` | `--print slithir-ssa` | Print SlithIR in SSA form |
| `variable-order` | `--print variable-order` | Print storage layout (variable order in slots) |
| `vars-and-auth` | `--print vars-and-auth` | State variables modified + authorization checks per function |

### Auditor-Critical Printers

#### Inheritance Mapping

```bash
# Full inheritance graph (DOT format)
slither . --print inheritance-graph

# Text-based inheritance summary
slither . --print inheritance

# For Foundry projects
slither . --print inheritance-graph --foundry-out-directory out
```

#### Attack Surface Enumeration

```bash
# Function visibility, state access, and modifiers
slither . --print function-summary

# State-changing entry points and their variables
slither . --print entry-points

# Authorization checks per function
slither . --print vars-and-auth

# Function selectors (for calldata analysis)
slither . --print function-id
```

#### Storage Layout Analysis

```bash
# Storage variable ordering (slot layout)
slither . --print variable-order

# Data dependency tracking
slither . --print data-dependency
```

#### Access Control Review

```bash
# Modifiers on each function
slither . --print modifiers

# Require/assert conditions
slither . --print require

# Functions missing whenNotPaused guard
slither . --print not-pausable

# Combined: state writes + auth checks
slither . --print vars-and-auth
```

#### Control Flow Analysis

```bash
# Control flow graphs per function
slither . --print cfg

# Call graph (inter-function calls)
slither . --print call-graph

# Dominator tree per function
slither . --print dominator
```

#### Code Quality Metrics

```bash
# Lines of code breakdown (src, dep, test)
slither . --print loc

# CK complexity metrics
slither . --print ck

# Halstead complexity metrics
slither . --print halstead

# Martin coupling metrics (Ca, Ce, Instability, Abstractness, Distance)
slither . --print martin
```

#### Foundry-Specific

```bash
# Detect Foundry cheatcode usage in contracts
slither . --print cheatcode
```

#### Running Multiple Printers

```bash
slither . --print function-summary,vars-and-auth,modifiers,variable-order,entry-points
```

---

## 6. Targeted Scan Strategies

### Full Audit Scan (Default)

```bash
slither . --foundry-out-directory out --json slither-report.json
```

Runs all 103 detectors, outputs structured JSON.

### High-Impact-Only Triage

```bash
slither . --foundry-out-directory out \
  --detect reentrancy-eth,arbitrary-send-eth,arbitrary-send-erc20,arbitrary-send-erc20-permit,controlled-delegatecall,controlled-array-length,delegatecall-loop,suicidal,unprotected-upgrade,uninitialized-state,uninitialized-storage,shadowing-state,unchecked-transfer,weak-prng,msg-value-loop,incorrect-return,return-leave,incorrect-exp,storage-array
```

### Reentrancy-Focused Scan

```bash
slither . --foundry-out-directory out \
  --detect reentrancy-eth,reentrancy-no-eth,reentrancy-benign,reentrancy-events,reentrancy-unlimited-gas
```

### Access Control Scan

```bash
slither . --foundry-out-directory out \
  --detect suicidal,unprotected-upgrade,tx-origin,arbitrary-send-eth,arbitrary-send-erc20,controlled-delegatecall,protected-vars
```

### Oracle/DeFi-Specific Scan

```bash
slither . --foundry-out-directory out \
  --detect weak-prng,incorrect-equality,divide-before-multiply,pyth-deprecated-functions,pyth-unchecked-confidence,pyth-unchecked-publishtime,chronicle-unchecked-price
```

### Noise-Reduced Scan (Exclude Low-Value Findings)

```bash
slither . --foundry-out-directory out \
  --exclude naming-convention,unused-state,solc-version,dead-code,too-many-digits,pragma,assembly,low-level-calls,boolean-equal
```

### Exclude Entire Severity Tiers

```bash
# Security-only (no informational or optimization)
slither . --foundry-out-directory out \
  --exclude-informational --exclude-optimization

# High and medium only
slither . --foundry-out-directory out \
  --exclude-informational --exclude-optimization --exclude-low
```

### Exclude Dependencies

```bash
slither . --foundry-out-directory out \
  --exclude-dependencies \
  --filter-paths 'lib|node_modules|openzeppelin'
```

### Include Only Specific Paths

```bash
slither . --foundry-out-directory out \
  --include-paths "src/|contracts/"
```

---

## 7. False Positive Management

### Triage Mode

```bash
slither . --foundry-out-directory out --triage-mode
```

Interactive mode: for each finding, mark as true positive or false positive. Decisions are saved to `slither.db.json`. Subsequent runs skip marked false positives.

Custom triage database path:

```bash
slither . --foundry-out-directory out --triage-mode --triage-database my-triage.db.json
```

### Inline Suppression

Suppress specific detectors in source code:

```solidity
// Single line
// slither-disable-next-line reentrancy-eth
(bool success, ) = msg.sender.call{value: amount}("");

// Block suppression
// slither-disable-start reentrancy-eth
(bool success, ) = msg.sender.call{value: amount}("");
require(success);
balances[msg.sender] = 0;
// slither-disable-end reentrancy-eth
```


### Security Annotations

Slither recognizes several `@custom:security` NatSpec annotations:

```solidity
/// @custom:security non-reentrant
uint256 public balance;  // Marks variable as protected from reentrancy

/// @custom:security write-protection="onlyOwner()"
uint256 public criticalValue;  // Marks variable as write-protected by the specified modifier

/// @custom:security isDelegatecallProxy
contract MyProxy {
    // Marks contract as a delegatecall proxy
}

/// @custom:security isUpgradeable
contract MyContract {
    // Marks contract as upgradeable
}
```

### Configuration File

Create `slither.config.json`:

```json
{
  "detectors_to_exclude": "naming-convention,unused-state,solc-version",
  "filter_paths": "lib/|node_modules/",
  "exclude_dependencies": true,
  "foundry_out_directory": "out"
}
```

Run with config:

```bash
slither . --config-file slither.config.json
```

---

## 8. JSON & SARIF Output

### Generate JSON Report

```bash
slither . --foundry-out-directory out --json slither-report.json
# Output to stdout
slither . --foundry-out-directory out --json -
```

### SARIF Output (for GitHub Code Scanning / IDE Integration)

```bash
slither . --foundry-out-directory out --sarif slither-report.sarif
```

### Zipped Output

```bash
slither . --foundry-out-directory out --zip slither-report.zip
slither . --foundry-out-directory out --zip slither-report.zip --zip-type deflated
```

### Top-Level JSON Schema

```json
{
  "success": true,
  "error": null,
  "results": {
    "detectors": [ ... ]
  }
}
```

### Detector Finding Schema

Each finding in the `detectors` array:

```json
{
  "check": "reentrancy-eth",
  "impact": "High",
  "confidence": "Medium",
  "description": "Reentrancy in Contract.withdraw() ...",
  "elements": [
    {
      "type": "function",
      "name": "withdraw",
      "source_mapping": {
        "start": 450,
        "length": 200,
        "filename_relative": "src/Vault.sol",
        "filename_absolute": "/path/to/src/Vault.sol",
        "filename_short": "src/Vault.sol",
        "lines": [25, 26, 27, 28, 29, 30],
        "starting_column": 5,
        "ending_column": 6
      },
      "type_specific_fields": { ... }
    }
  ],
  "additional_fields": {
    "underlying_type": "external_calls_sending_eth"
  }
}
```

### Element Types

| Type | Description |
|------|-------------|
| `contract` | Contract definition |
| `function` | Function definition |
| `variable` | Variable (state or local) |
| `node` | CFG node (specific code location) |
| `pragma` | Pragma directive |
| `enum` | Enum definition |
| `struct` | Struct definition |
| `event` | Event definition |

### Markdown Checklist Output

```bash
slither . --foundry-out-directory out --checklist > audit-checklist.md
slither . --foundry-out-directory out --checklist --checklist-limit 5 > audit-checklist.md
```

---

## 9. Upgradeability Analysis

### `slither-check-upgradeability`

Dedicated tool for proxy/implementation contract analysis.

```bash
slither-check-upgradeability . ContractName
```

### Comparing V1 and V2 Implementations

```bash
slither-check-upgradeability . ContractV1 \
  --new-contract-name ContractV2 \
  --new-contract-filename ./src/ContractV2.sol
```

### With Proxy Contract

```bash
slither-check-upgradeability . Implementation \
  --proxy-name TransparentUpgradeableProxy \
  --proxy-filename ./src/Proxy.sol
```

### Checks Performed (17 Total)

#### High Severity (10 checks)

- State variable becoming constant
- State variable ceasing to be constant
- Incorrect storage variable ordering between versions
- Variables with initialization values (incompatible with `delegatecall` pattern)
- Function ID collision between proxy and implementation
- Function shadowing between proxy and implementation
- Missing `initializer` modifier
- Missing parent `initialize` call
- Multiple calls to same initializer
- Missing `Initializable` inheritance

#### Medium Severity (2 checks)

- Extra state variables in new version
- Missing state variables in new version

#### Informational (5 checks)

- Initialization best practice recommendations

---

## 10. Additional Slither Tools

All registered entry points from `pyproject.toml`:

### `slither-flat` — Flatten Contracts

```bash
slither-flat . --contract ContractName
```

Merges all imports into a single file for Etherscan verification or manual review.

### `slither-check-erc` — ERC Conformance

```bash
slither-check-erc . ContractName
```

Validates compliance with ERC20, ERC721, and other token standards.

### `slither-prop` — Property Generation

```bash
slither-prop . --contract ContractName
slither-prop . --contract ContractName --scenario Transferable
```

Automatically generates unit tests and properties for invariant testing (compatible with Echidna). Use `--contract` to target a specific contract and `--scenario` to select a test scenario.

### `slither-read-storage` — Read Storage Values

```bash
# Retrieve a single variable's value from a deployed contract
slither-read-storage . 0xDeployedAddress --variable-name balances --rpc-url $RPC_URL

# Retrieve full storage layout for a contract
slither-read-storage . 0xDeployedAddress --contract-name ContractName --rpc-url $RPC_URL --json storage_layout.json

# Retrieve storage layout with actual values
slither-read-storage . 0xDeployedAddress --contract-name ContractName --rpc-url $RPC_URL --json storage_layout.json --value
```

Reads actual storage values from deployed contracts by mapping variable names to storage slots. The first positional argument is the project directory (or Etherscan address), the second is the deployed contract address.

### `slither-interface` — Generate Interface

```bash
slither-interface ContractName .
```

Generates a Solidity interface from a contract's external/public functions. Syntax: contract name first, then the source file or project directory.

### `slither-format` — Auto-Fix Findings

```bash
slither-format .
slither-format . --detect naming-convention,external-function
```

Automatically applies fixes for supported detector findings. Only a limited set of detectors are supported: `unused-state`, `solc-version`, `pragma`, `naming-convention`, `external-function`, `constable-states`, `constant-function-asm`, `constant-function-state`.

### `slither-find-paths` — Function Reachability

```bash
slither-find-paths . Contract.functionName
slither-find-paths . Contract.functionA Contract.functionB
```

Finds possible execution paths to the specified target functions. Targets use `Contract.functionName` format.

### `slither-simil` — Code Similarity

```bash
# Train a model on a codebase
slither-simil train model.bin --filename .

# Test similarity of a specific function against the model
slither-simil test model.bin --filename . --fname "Contract.functionName"

# Get info about a trained model
slither-simil info model.bin --filename .
```

Detects similar code patterns across contracts (clone detection, code reuse analysis). Requires a trained model — first `train`, then `test` or `info`.

### `slither-mutate` — Mutation Testing

```bash
slither-mutate . --test-cmd "forge test"
slither-mutate . --test-cmd "forge test" --test-dir test/ --timeout 60
```

Applies mutations to the contract source code and runs the test suite to evaluate its effectiveness. Requires `--test-cmd` to specify how to run the project's tests.

### `slither-check-kspec` — K Specification Coverage

```bash
slither-check-kspec . spec.md
```

Measures coverage of formal K-framework specifications. Takes the project directory and the Klab spec markdown file as positional arguments.

### `slither-doctor` — Diagnose Issues

```bash
slither-doctor .
```

Diagnoses common Slither installation and configuration issues for the given project.

### `slither-documentation` — Generate NatSpec Documentation

```bash
slither-documentation .
slither-documentation . --overwrite
```

Auto-generates NatSpec documentation for all functions using AI. Use `--overwrite` to replace existing files (use with caution).

### GitHub Actions Integration

```yaml
- name: Run Slither
  uses: crytic/slither-action@v0.4.0
  with:
    target: '.'
    slither-args: '--json slither-report.json --exclude-dependencies'
```

---

## 11. Python API for Custom Analysis

### Basic Setup

```python
from slither.slither import Slither

slither = Slither('.')
```

### Core Object Model

#### Slither Instance

| Property | Description |
|----------|-------------|
| `slither.contracts` | All contracts (including inherited) |
| `slither.contracts_derived` | Only non-inherited contracts (avoids duplicates) |
| `slither.get_contract_from_name(name)` | Get specific contract by name |

#### Contract Object

| Property | Description |
|----------|-------------|
| `contract.name` | Contract name |
| `contract.functions` | All functions |
| `contract.modifiers` | All modifiers |
| `contract.state_variables` | All state variables |
| `contract.inheritance` | Parent contracts (linearized) |
| `contract.all_functions_called` | All internally reachable functions |
| `contract.get_function_from_signature(sig)` | Lookup by signature |
| `contract.get_modifier_from_signature(sig)` | Lookup modifier by signature |
| `contract.get_state_variable_from_name(name)` | Lookup state variable |

#### Function Object

| Property | Description |
|----------|-------------|
| `function.name` | Function name |
| `function.full_name` | Name with parameter types |
| `function.visibility` | `public`, `external`, `internal`, `private` |
| `function.view` | Is view function |
| `function.pure` | Is pure function |
| `function.is_constructor` | Is constructor |
| `function.nodes` | Control flow graph nodes |
| `function.entry_point` | CFG entry node |
| `function.variables_read` | All variables read |
| `function.variables_written` | All variables written |
| `function.state_variables_read` | State variables read |
| `function.state_variables_written` | State variables written |
| `function.modifiers` | Applied modifiers |
| `function.parameters` | Input parameters |
| `function.return_type` | Return types |

#### Node Object

| Property | Description |
|----------|-------------|
| `node.type` | `NodeType` enum (ENTRY_POINT, RETURN, IF, etc.) |
| `node.expression` | Code expression at this node |
| `node.variables_read` | Variables read at this node |
| `node.variables_written` | Variables written at this node |
| `node.state_variables_read` | State variables read |
| `node.state_variables_written` | State variables written |
| `node.irs` | SlithIR operations |

#### Variable Object

| Property | Description |
|----------|-------------|
| `variable.name` | Variable name |
| `variable.type` | Solidity type |
| `variable.initialized` | Has initializer |

### Example: Map State Variable Access per Function

```python
from slither.slither import Slither

slither = Slither('.')

for contract in slither.contracts_derived:
    print(f"\n=== {contract.name} ===")
    for function in contract.functions:
        if function.visibility in ['public', 'external']:
            reads = [v.name for v in function.state_variables_read]
            writes = [v.name for v in function.state_variables_written]
            print(f"  {function.visibility} {function.name}()")
            if reads:
                print(f"    Reads:  {reads}")
            if writes:
                print(f"    Writes: {writes}")
```

### Example: Find Functions Without Access Control That Write State

```python
from slither.slither import Slither

slither = Slither('.')

for contract in slither.contracts_derived:
    for function in contract.functions:
        if (function.visibility in ['public', 'external']
            and not function.is_constructor
            and len(function.modifiers) == 0
            and len(function.state_variables_written) > 0):
            print(f"[!] {contract.name}.{function.name}() — "
                  f"writes state with no modifier")
            print(f"    Writes: {[v.name for v in function.state_variables_written]}")
```

### Example: Inheritance Chain Analysis

```python
from slither.slither import Slither

slither = Slither('.')

for contract in slither.contracts_derived:
    print(f"\n{contract.name}")
    print(f"  Inherits: {[p.name for p in contract.inheritance]}")
    for function in contract.functions:
        if hasattr(function, 'overrides') and function.overrides:
            print(f"  Overrides: {function.full_name}")
```

---

## 12. Custom Detector Development

### Detector Skeleton

```python
from slither.detectors.abstract_detector import (
    AbstractDetector,
    DetectorClassification,
)

class MyDetector(AbstractDetector):
    ARGUMENT = 'my-detector'
    HELP = 'Detects [vulnerability description]'
    IMPACT = DetectorClassification.HIGH
    CONFIDENCE = DetectorClassification.HIGH

    WIKI = 'https://example.com/my-detector'
    WIKI_TITLE = 'My Detector'
    WIKI_DESCRIPTION = 'Technical description'
    WIKI_EXPLOIT_SCENARIO = 'Example exploit'
    WIKI_RECOMMENDATION = 'Mitigation steps'

    def _detect(self):
        results = []
        for contract in self.contracts:
            for function in contract.functions:
                if function.view or function.pure:
                    continue
                for node in function.nodes:
                    for ir in node.irs:
                        if self._is_vulnerable(ir):
                            info = [
                                "Vulnerability in ",
                                function,
                                " at ",
                                node,
                                "\n"
                            ]
                            res = self.generate_result(info)
                            results.append(res)
        return results

    def _is_vulnerable(self, ir):
        return False
```

### Classification Levels

| Field | Values |
|-------|--------|
| `IMPACT` | `HIGH`, `MEDIUM`, `LOW`, `INFORMATIONAL`, `OPTIMIZATION` |
| `CONFIDENCE` | `HIGH`, `MEDIUM`, `LOW` |

### Registration Methods

**Method 1:** Add to `slither/detectors/all_detectors.py`

**Method 2:** Create a plugin package with entry points

### Running Custom Detectors

```bash
slither . --detect my-detector
```

---

## 13. Detector-to-PoC Mapping

When writing a PoC for a specific detector finding, use these strategies:

| Detector | PoC Strategy |
|----------|-------------|
| `reentrancy-eth` | Deploy a malicious contract with a `receive()` that re-enters the target. Use `vm.deal` to fund the attacker, call the vulnerable function, assert drained funds. |
| `arbitrary-send-eth` / `arbitrary-send-erc20` | Use `vm.prank` as an unauthorized address, call the function that sends funds, prove funds arrived at attacker's address. |
| `unprotected-upgrade` | Use `vm.prank` as a non-admin, call `initialize()` or `upgradeToAndCall()`, prove ownership was taken. |
| `suicidal` | Use `vm.prank` as an unauthorized address, call the function containing `selfdestruct`, prove the contract was destroyed. |
| `controlled-delegatecall` | Deploy a malicious implementation, call the vulnerable function with the malicious address, prove storage was overwritten. |
| `unchecked-transfer` | Set up a token that returns `false` on `transfer`, call the vulnerable function, prove it succeeds when it should have failed. Use `vm.mockCall` to simulate. |
| `uninitialized-state` / `uninitialized-storage` | Prove the uninitialized variable contains unexpected data that breaks a critical operation. |
| `locked-ether` | Send ETH to the contract, prove there is no way to withdraw it. |
| `divide-before-multiply` | Provide inputs where the precision loss is significant, prove the output is incorrect by comparing with the correct calculation. |
| `incorrect-equality` | Manipulate state to make the strict equality check pass/fail unexpectedly (e.g., send dust amounts). |
| `tx-origin` | Create a scenario with a forwarding contract where `msg.sender != tx.origin` to bypass or exploit the check. |

---

## 14. Known Limitations

Understanding what Slither **cannot** do is as important as knowing what it can. These blind spots define where manual code review and Foundry invariant testing must fill the gap.

### Detection Blind Spots

| Limitation | Why | Workaround |
|-----------|-----|------------|
| **Cross-contract reentrancy** | Slither analyzes contracts individually; it doesn't track state flow through external call chains across multiple contracts | Manual code reading of cross-contract call sequences + Foundry PoC |
| **Read-only reentrancy** | Reentrancy detectors focus on state-writing calls; `view`/`pure` function calls during reentrancy are invisible | Manual review of any protocol that reads external state (LP pricing, oracle views) |
| **Logic bugs** | Slither matches AST patterns — it cannot reason about business logic, economic incentives, or intended vs actual behavior | Adversarial thinking (Phase 4.4), invariant testing |
| **Multi-transaction attack sequences** | Slither analyzes single-function execution paths; attacks requiring a specific sequence of transactions across blocks are invisible | Manual attack path construction + Foundry multi-step PoCs |
| **Inline assembly semantics** | Slither can detect `assembly` usage but doesn't deeply analyze the Yul/EVM opcodes within assembly blocks | Manual review of every `assembly` block for correctness |
| **`delegatecall` side effects** | `function-summary` printer reports variables read/written in the context of the calling contract, but `delegatecall` executes in the caller's storage context — the actual storage writes may differ | Cross-reference `variable-order` of both proxy and implementation; manually verify storage alignment |
| **Token callback reentrancy** | ERC777 `tokensReceived`, ERC721 `onERC721Received`, ERC1155 `onERC1155Received` hooks create reentrant calls inside token transfer, invisible to Slither | Check `checklists/non-standard-tokens.md` §8 for token callback patterns |
| **Economic / game-theoretic exploits** | Flash loan sandwich attacks, MEV extraction, funding rate manipulation — Slither cannot model economic incentives | Manual analysis using domain-specific playbooks |
| **Transient storage (EIP-1153)** | `TSTORE`/`TLOAD` are recent opcodes; Slither support may be incomplete | Manual review of all transient storage usage |

### Confidence Calibration

- **High confidence + High impact** detector finding: Still verify — "High confidence" means the AST pattern is clear, not that exploitation is confirmed. The pattern may be intentional (e.g., documented reentrancy guard elsewhere).
- **Medium confidence** findings: ~30-50% are false positives in typical codebases. Always verify reachability via `call-graph` and access control via `vars-and-auth`.
- **`function-summary` accuracy**: This printer is derived from the AST and is ground truth for what the compiler sees. However, it does not account for: (a) `delegatecall` executing different code, (b) inline assembly performing raw `SSTORE`/`SLOAD`, (c) self-destructing contracts.

### What Slither Does Well (Lean Into These)

- **Structural analysis** — inheritance graphs, storage layouts, function summaries are authoritative
- **Known vulnerability patterns** — reentrancy, unchecked return values, uninitialized state, access control gaps
- **Compliance checking** — ERC20/721 conformance, upgradeability safety
- **Code quality** — naming conventions, dead code, unused variables, optimization opportunities
