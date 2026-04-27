# Smart Contract Auditor Skill

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

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- [Foundry](https://getfoundry.sh/) installed (`forge`, `cast`, `anvil`)
- [Slither](https://github.com/crytic/slither) installed (`pip install slither-analyzer` or `uv tool install slither-analyzer`)

### Setup

Copy the `smart-contract-auditor` folder into your Claude Code skills directory:

```bash
cp -r smart-contract-auditor ~/.claude/skills/smart-contract-auditor
```

Or symlink it:

```bash
ln -s /path/to/smart-contract-auditor ~/.claude/skills/smart-contract-auditor
```

The skill will appear as `/smart-contract-auditor` in Claude Code.

## Usage

In Claude Code, invoke the skill on any smart contract project:

```
/smart-contract-auditor
```

Or describe what you need:

```
Audit this Foundry project for security vulnerabilities
```

The agent will walk through all 7 phases, starting with environment setup and ending with a formatted report.

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
