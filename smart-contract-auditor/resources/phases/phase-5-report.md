# Phase 5: Final Report Assembly

**Purpose:** Assemble all Phase 4 finding write-ups into the final submission format, review for consistency, and verify completeness.

**Gate:** All findings assembled, severity reviewed, PoCs verified, format validated.

**Inputs:**
- `audit-output/phase-4-findings/` (all finding folders with PoCs and write-ups)
- `audit-output/phase-2-docs/codebase-overview.md` (for cross-checking claims)
- `audit-output/phase-3-planning/attack-plan.md` (for prior-audit-overlap verdicts and privilege risk coverage)
- Platform template: `resources/templates/code4rena.md` or `resources/templates/sherlock.md`

**Outputs:** Write to `audit-output/phase-5-report/`:
- `final-report.md` — assembled report (all findings)
- `qa-report.md` — consolidated Low + Centralization findings

---

## Checklist

1. **Collect all findings** from `audit-output/phase-4-findings/` — High/Medium individual submissions, Low/Centralization consolidated into QA report.
2. **Review severity consistency** — are similar-impact findings rated the same? Would any finding be better at a different severity?
3. **Check for duplicates** — findings with the same root cause should be consolidated.
4. **Verify all PoCs still pass** — run `forge test --match-path test/audit/ -vvv` one final time.
5. **Cross-check known issues** — for each finding tagged `prior-audit-overlap`, verify the Phase 4 verdict: true duplicates are excluded; findings with higher demonstrated severity or a distinct attack path remain with an explicit note referencing the prior finding.
6. **Cross-check against Phase 2 document** — does the codebase documentation support every finding's root cause and impact claim? If a finding contradicts something in the doc, one of them is wrong.
7. **Verify privilege risk coverage** — compare the attack plan §4 (Privilege & Management Risk) against C-XX findings in the QA report. Every critical privilege risk should be documented.
8. **Final format check** — titles under 255 chars, root cause links point to exact line numbers, all mandatory fields populated.

---

## Key Reminders

- **PoC is mandatory** for all High/Medium code vulnerability submissions.
- **No PoC needed** for Centralization/privilege risk findings — written analysis is sufficient.
- **Do not overstate severity** — QA findings submitted as Medium/High are penalized.
- High/Medium: individual submissions. Low/Centralization: consolidated QA report.

## Platform-Specific Notes

### Code4rena
- High/Medium submitted individually.
- Low + Centralization consolidated into one QA report using L-XX and C-XX labels.
- Do NOT use R- (refactor) or I- (informational) labels.

### Sherlock
- High/Medium findings submitted individually (Low/Info not accepted unless contest specifies otherwise).
- Findings are deduplicated by root cause — ensure each submission has a distinct root cause.
- Pre-conditions are explicitly separated into Internal and External.
- Check contest README for admin trust assumptions (TRUSTED vs RESTRICTED).
