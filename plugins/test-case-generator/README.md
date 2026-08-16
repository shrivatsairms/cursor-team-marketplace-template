# Manual Test Case Generator

Cursor plugin that turns **Jira Epics / Stories** into exhaustive **manual test case CSVs** using **Atlassian (Jira) MCP**.

Plugin id: **`test-case-generator`**

## What it includes

| Component | Path | Purpose |
|-----------|------|---------|
| Rule | `rules/qa-jira-manual-tests.mdc` | Always-on QA + Jira CSV standards |
| Skill | `skills/senior-qa-engineer/` | CSV format, coverage, forbidden terms |
| Command | `commands/generate-tests.md` | Slash / prompt workflow for Epics |
| Agent | `agents/test-case-generator.md` | Dedicated generator subagent |
| MCP | `mcp.json` | Atlassian Rovo MCP (Jira) via OAuth |

## How to use

1. Install this marketplace plugin in Cursor (team import or local).
2. Authenticate the **atlassian** MCP server when prompted (browser OAuth).
3. Run `/generate-tests` or ask: `Generate manual test cases for Epic DXP-12345`.
4. Find output at `tests/DXP-12345_test_cases.csv`.

## Output format

RFC 4180 CSV with header:

```text
"Test Case ID","Jira ID","Priority","Scenario","Pre-conditions","Test Data","Steps","Expected Result","Scenario Type"
```

## Notes

- Manual tests only — no Playwright / Selenium / Robot.
- Does not commit Jira tokens; OAuth is handled by Cursor + Atlassian.
