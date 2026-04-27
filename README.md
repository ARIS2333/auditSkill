# Smart Contract Auditor Skill

**[English](README.md) | [中文](README_CN.md)**

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) custom skill that turns Claude into a structured smart contract security auditor. It uses **Slither** for static analysis and **Foundry** for Proof-of-Concept exploit development, following a 7-phase workflow designed to minimize AI hallucination.

## Why This Skill?

AI agents hallucinate when auditing smart contracts — they misread inheritance chains, fabricate access control assumptions, and generate broken PoCs. This skill solves that with a **tool-first, code-second** approach: the agent must build a verified structural model of the codebase using Slither's printers (machine-generated ground truth) *before* reading any source code.

## Workflow Overview

| Phase | Name | What Happens |
|-------|------|-------------|
| 0 | Environment Setup | Detect project type, verify compilation, identify audit scenarios (DeFi, proxy, token, cross-chain), select submission platform |
| 1 | Structural Ground Truth | Slither printers map inheritance, entry points, function-to-state-variable relationships, and storage layout. **No code reading yet.** |
| 2 | Attack Hypothesis Generation | More printers map access control, pause gaps, validation conditions, data flow. Produces a ranked target list. **Still no code reading.** |
| 3 | Targeted Code Reading | Now read `.sol` files, guided by Phase 1-2 outputs. Every claim cross-referenced against printer output. |
| 4 | Automated Vulnerability Scanning | Slither detectors (full scan + targeted: reentrancy, access control, oracle/DeFi). Runs in parallel with Phase 3. |
| 5 | Triage & Deep Analysis | Cross-reference findings with code reading. Classify as Confirmed / Potential / False Positive. |
| 6 | PoC Development | Foundry exploit tests for each confirmed High/Medium finding. |
| 7 | Report Generation | Platform-specific submission format (Code4rena by default). |

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
git clone https://github.com/YOUR_USERNAME/smart-contract-auditor.git

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
├── SKILL.md                                # required — must exist
└── resources/                              # reference docs and templates
    ├── slither-audit-guide.md
    ├── foundry-audit-guide.md
    ├── poc-template.md
    └── templates/
        └── code4rena.md
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

The agent will walk through all 7 phases: environment setup, structural analysis, attack surface mapping, code reading, vulnerability scanning, triage, PoC development, and report generation.

### Platform Selection

During Phase 0, the agent asks which submission platform to target. Currently supported:

| Platform | Status |
|----------|--------|
| Code4rena | Ready |
| Sherlock | Planned |
| Immunefi | Planned |
| HackerOne | Planned |

Default is Code4rena. To add a new platform, create a template in `resources/templates/` and update the table in `SKILL.md` Phase 0.6.

## File Structure

```
smart-contract-auditor/
├── SKILL.md                              # Core skill — 7-phase audit workflow
└── resources/
    ├── slither-audit-guide.md            # 28 printers, 103 detectors, CLI flags, Python API
    ├── foundry-audit-guide.md            # Forge CLI, cheatcodes, cast, anvil, PoC development
    ├── poc-template.md                   # Foundry PoC template
    └── templates/
        └── code4rena.md                  # Code4rena submission format & checklist
```

## Key Features

### Anti-Hallucination Design

The skill enforces strict phase ordering — Phases 0-2 use only Slither printers (AST-derived facts) to build a structural model before the agent reads any code. If the agent's code reading later disagrees with printer output, the printer is treated as correct.

### Inline Slither Reference

SKILL.md embeds the most commonly used Slither commands directly in each phase (18 printers, 26 high/medium detectors, preset scan commands) so the agent doesn't need to search the full guide for routine operations.

### Detector-to-PoC Mapping

For each common Slither detector finding (reentrancy, arbitrary-send, unprotected-upgrade, etc.), the skill provides a specific PoC strategy — what contracts to deploy, which cheatcodes to use, and what to assert.

### Error Recovery

Three-tier fallback when Slither fails to compile: fix & retry, partial analysis, or Foundry-only degraded mode. When errors persist, the agent is instructed to consult the official GitHub repos.

## Slither Printers Used

These printers are the backbone of the anti-hallucination workflow:

| Printer | Phase | Purpose |
|---------|-------|---------|
| `human-summary` | 1 | Scope and complexity overview |
| `loc` | 1 | Lines of code breakdown |
| `inheritance-graph` | 1 | Full contract hierarchy (DOT) |
| `inheritance` | 1 | Text inheritance summary |
| `c3-linearization` | 1 | Diamond inheritance resolution |
| `entry-points` | 1 | All state-changing entry points |
| `function-summary` | 1 | Function visibility, modifiers, state var access |
| `variable-order` | 1 | Storage slot layout |
| `vars-and-auth` | 2 | State writes + authorization checks |
| `modifiers` | 2 | Guard coverage per function |
| `not-pausable` | 2 | Missing pause guards |
| `data-dependency` | 2 | User input to state change tracking |
| `call-graph` | 2 | Cross-function call relationships |

## Contributing

Contributions welcome — especially:

- **New platform templates** (`resources/templates/sherlock.md`, etc.)
- **Detector-to-PoC mappings** for additional Slither detectors
- **Scenario-specific checklists** (lending protocols, AMMs, bridges, etc.)

## References

- [Slither Documentation](https://github.com/crytic/slither/wiki)
- [Foundry Book](https://book.getfoundry.sh/)
- [Code4rena Submission Guidelines](https://docs.code4rena.com/competitions/submission-guidelines)
- [Code4rena Severity Categorization](https://docs.code4rena.com/competitions/severity-categorization)

## License

MIT
