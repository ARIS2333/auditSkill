# Smart Contract Auditor Skill

**[English](README.md) | [中文](README_CN.md)**

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) custom skill that turns Claude into a structured smart contract security auditor. It uses **Slither** for static analysis and **Foundry** for Proof-of-Concept exploit development, following a 7-phase workflow (Phases 0-6) designed to minimize AI hallucination.

## Why This Skill?

AI agents hallucinate when auditing smart contracts — they misread inheritance chains, fabricate access control assumptions, and generate broken PoCs. This skill solves that with a **tool-first, code-second** approach: the agent must build a verified structural model of the codebase using Slither's printers (machine-generated ground truth) *before* reading any source code.

## Workflow Overview

| Phase | Name | What Happens |
|-------|------|-------------|
| 0 | Environment Setup | Detect project type, verify compilation, identify audit scenarios (DeFi, proxy, token, cross-chain, staking, transient storage), select submission platform |
| 1 | Structural Reconnaissance | Run 13 Slither printers to map inheritance, entry points, function-to-state-variable relationships, storage layout, access control, and data flow. **No code reading.** |
| 2 | Codebase Documentation | Dive into source code guided by Phase 1 outputs. Produce a comprehensive Mermaid-based codebase document (13 sections) and a ranked attack hypothesis list. |
| 3 | Automated Scanning | Run Slither detectors (full scan, high-impact focused, scenario-specific). Contextualize every finding against Phase 2 documentation. |
| 4 | Targeted Code Reading & Triage | Deep security-focused code reading with a 24-item checklist. Activate domain-specific playbooks. Classify findings as Confirmed / Potential / False Positive. |
| 5 | PoC & Finding Documentation | Write Foundry PoC and finding report together per confirmed/potential finding. Invariant testing when time permits. |
| 6 | Final Report Assembly | Assemble findings into platform-specific submission format. Severity review, dedup, PoC re-verification. |

**Phases 0-3 are strictly sequential.** Phase 1 builds a structural model via printers (no code reading). Phase 2 reads source code using Phase 1 as its guide. Phase 3 runs detectors against a codebase you already understand. Phases 4-6 are the core analysis, PoC, and reporting stages.

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or desktop app installed
- [Foundry](https://getfoundry.sh/) installed (`forge`, `cast`, `anvil`)
- [Slither](https://github.com/crytic/slither) installed (`pip install slither-analyzer` or `uv tool install slither-analyzer`)

Install Foundry:

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

Install Slither:

```bash
pip install slither-analyzer
# or
uv tool install slither-analyzer
```

### Option A: Global Install (Recommended — All Projects)

Install once, use in every project you open with Claude Code:

```bash
# Clone the repo
git clone https://github.com/ARIS2333/smart-contract-auditor.git

# Copy the skill folder to Claude Code's global skills directory
cp -r smart-contract-auditor/smart-contract-auditor ~/.claude/skills/smart-contract-auditor
```

Or symlink it so updates to the repo are reflected automatically:

```bash
ln -s $(pwd)/smart-contract-auditor/smart-contract-auditor ~/.claude/skills/smart-contract-auditor
```

### Option B: Project-Level Install (Single Project Only)

Install into a specific audit project. Useful if you want to commit the skill alongside the project so team members get it automatically:

```bash
# In your audit project root
mkdir -p .claude/skills
cp -r /path/to/smart-contract-auditor/smart-contract-auditor .claude/skills/smart-contract-auditor
```

### Verify Installation

Open Claude Code in any project (for global install) or the target project (for project-level install):

1. Type `/` — you should see `smart-contract-auditor` in the autocomplete menu
2. Or type `/smart-contract-auditor` directly

If it doesn't appear, check that the folder structure is correct:

```
~/.claude/skills/smart-contract-auditor/    # global
# or
.claude/skills/smart-contract-auditor/      # project-level
├── SKILL.md                                # required — skill entry point
└── resources/
    ├── tools/
    │   ├── slither.md                      # 27 printers, 99 detectors, CLI flags, Python API
    │   └── foundry.md                      # Forge CLI, cheatcodes, cast, anvil, PoC workflow
    ├── templates/
    │   ├── structural-summary.md           # Phase 1 output template
    │   ├── codebase-report.md              # Phase 2 output template (13 sections)
    │   ├── poc.md                          # 5 PoC starter templates
    │   ├── code4rena.md                    # Code4rena submission format
    │   └── sherlock.md                     # Sherlock submission format
    ├── checklists/
    │   ├── non-standard-tokens.md          # 14 non-standard ERC20 behaviors
    │   └── domain-playbooks.md             # Attack checklists for 10 protocol types
    └── phases/
        ├── phase-0-setup.md
        ├── phase-1-recon.md
        ├── phase-2-docs.md
        ├── phase-3-scanning.md
        ├── phase-4-analysis.md
        ├── phase-5-findings.md
        └── phase-6-report.md
```

## Usage

### Start an Audit

Open Claude Code in your target smart contract project and invoke the skill:

```
/smart-contract-auditor
```

Or describe what you need in natural language — Claude will auto-detect the skill:

```
Audit this Foundry project for security vulnerabilities
```

```
Run slither on this codebase and write PoCs for any critical findings
```

The agent will walk through all 7 phases: environment setup, structural reconnaissance, codebase documentation, automated scanning, targeted code reading & triage, PoC development, and report assembly.

### Platform Selection

During Phase 0, the agent asks which submission platform to target. Currently supported:

| Platform | Status |
|----------|--------|
| Code4rena | Ready |
| Sherlock | Ready |
| Immunefi | Planned |
| HackerOne | Planned |

Default is Code4rena. To add a new platform, create a template in `resources/templates/` and update Phase 0.6 in `SKILL.md`.

## File Structure

```
smart-contract-auditor/
├── SKILL.md                              # Core skill — 7-phase audit workflow (Phases 0-6)
└── resources/
    ├── tools/
    │   ├── slither.md                    # 27 printers, 99 detectors, CLI flags, Python API, additional tools
    │   └── foundry.md                    # Forge CLI, cheatcodes, forge-std, cast, anvil, PoC workflow
    ├── templates/
    │   ├── structural-summary.md         # Phase 1 output template
    │   ├── codebase-report.md            # Phase 2 output template (13 sections, Mermaid diagrams)
    │   ├── poc.md                        # 5 Foundry PoC starter templates
    │   ├── code4rena.md                  # Code4rena submission format & severity guide
    │   └── sherlock.md                   # Sherlock submission format & dedup model
    ├── checklists/
    │   ├── non-standard-tokens.md        # 14 non-standard ERC20 token behaviors
    │   └── domain-playbooks.md           # Domain-specific attack checklists (10 protocol types)
    └── phases/
        ├── phase-0-setup.md              # Environment setup & scope discovery
        ├── phase-1-recon.md              # Structural reconnaissance (13 printers)
        ├── phase-2-docs.md               # Codebase documentation & hypothesis list
        ├── phase-3-scanning.md           # Automated Slither detector scans
        ├── phase-4-analysis.md           # Targeted code reading & triage (24-item checklist)
        ├── phase-5-findings.md           # PoC & finding documentation
        └── phase-6-report.md             # Final report assembly
```

## Key Features

### Anti-Hallucination Design

The skill enforces strict phase ordering — Phase 0 sets up the environment, Phase 1 uses only Slither printers (AST-derived facts) to build a structural model before the agent reads any code. Phase 2 then reads source code with Phase 1 as its guide. If the agent's code reading later disagrees with printer output, the printer is treated as correct.

### Modular Phase Architecture

Each phase has its own detailed instruction file in `resources/phases/`. Phases reference their inputs (prior phase outputs) and resource files as needed. This keeps the core `SKILL.md` concise while providing deep guidance when the agent needs it.

### Comprehensive Tool References

`resources/tools/slither.md` covers all 27 printers, 99 detectors with severity/confidence, CLI flags, Python API, and known limitations. `resources/tools/foundry.md` covers Forge CLI, all cheatcode signatures, forge-std helpers, mainnet forking, Anvil, Cast, and invariant testing.

### Detector-to-PoC Mapping

For each common Slither detector finding (reentrancy, arbitrary-send, unprotected-upgrade, etc.), the skill provides a specific PoC strategy — what contracts to deploy, which cheatcodes to use, and what to assert.

### Domain-Specific Playbooks

Attack checklists for 10 protocol types: Vault/ERC4626, AMM/DEX, Lending, Proxy/Upgradeable, Governance, Cross-Chain/Bridge, Staking/Restaking, Perpetuals/Derivatives, Token Launch/Bonding Curves, and Yield Aggregators.

### Error Recovery

Three-tier fallback when Slither fails to compile: fix & retry, partial analysis, or Foundry-only degraded mode. When errors persist, the agent is instructed to consult the official GitHub repos.

## Slither Printers Used

These 13 printers are run in Phase 1 to build the structural model before any code reading:

| Printer | Purpose |
|---------|---------|
| `human-summary` | Contract count, SLOC, ERCs, complexity overview |
| `function-summary` | **Anti-hallucination ground truth.** Per-function: visibility, modifiers, state vars read/written |
| `entry-points` | All state-changing external/public functions and their accessed variables |
| `vars-and-auth` | State variables modified + authorization checks per function |
| `inheritance` | Text-based inheritance relationships between contracts |
| `inheritance-graph` | Inheritance hierarchy visualization (DOT) |
| `variable-order` | Storage slot layout — variable ordering per contract |
| `modifiers` | Which modifiers are applied to each function |
| `require` | All `require` and `assert` conditions per function |
| `not-pausable` | Functions missing `whenNotPaused` guard |
| `call-graph` | Cross-function call relationships (DOT) |
| `data-dependency` | How user input flows through to state variables |
| `function-id` | Keccak256 function selectors (check for collisions in proxy patterns) |

## Contributing

Contributions welcome — especially:

- **New platform templates** (`resources/templates/immunefi.md`, etc.)
- **Detector-to-PoC mappings** for additional Slither detectors
- **Domain-specific playbooks** for new protocol types

## References

- [Slither Documentation](https://github.com/crytic/slither/wiki)
- [Foundry Book](https://book.getfoundry.sh/)
- [Code4rena Submission Guidelines](https://docs.code4rena.com/competitions/submission-guidelines)
- [Code4rena Severity Categorization](https://docs.code4rena.com/competitions/severity-categorization)
- [Sherlock Judging Rules](https://docs.sherlock.xyz/audits/judging/judging)

## License

MIT
