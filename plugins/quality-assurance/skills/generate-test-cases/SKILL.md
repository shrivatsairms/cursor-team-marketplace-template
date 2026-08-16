---
name: generate-test-cases
description: >
  Convert Jira Epics and Stories into exhaustive RFC 4180 manual test case CSVs.
  Use when the user asks for manual test cases, QA CSV, story-to-test mapping,
  generate-tests, or Jira-driven test documentation. Requires Atlassian/Jira MCP.
---

# Generate Test Cases

Generate executable manual test cases from live Jira work items and persist them as an RFC 4180 CSV in the selected GitLab repository.

## What to Do

- Generate/Expand manual test cases from a Jira Epic or Story
- Produce `tests/test-cases/[JIRA_KEY]_test_cases.csv` within the root of the selected Gitlab repository.
- Map acceptance criteria into Happy / Negative / Edge / UI/UX / End-to-End scenarios

## Workflow

1. Identify the primary Jira key and work item type.
2. Create a new branch from the currently checked-out branch before writing the CSV. Name the branch using following convention: `<work-type>/<JIRA_KEY>`, where `<work-type>` is `bug`, `story`, or `feature`.
3. Fetch the Jira work item through Atlassian/Jira MCP.
4. Read the summary, full description, acceptance criteria, comments (only when relevant), links, attachments, and any exposed custom acceptance fields.
5. If the work item is an Epic, fetch all children with the Epic workflow below.
6. Build the scenario matrix before writing the CSV.
7. Write the CSV to `tests/test-cases/[JIRA_KEY]_test_cases.csv`. Create the folder if it does not exist.
8. Reply with only the minimal Jira comment summary described below.

## Epic workflow

1. Fetch the Epic summary and full description.
2. Fetch children with paginated JQL:

   ```text
   "Epic Link" = KEY OR parent = KEY OR "Parent Link" = KEY
   ```

3. Use `maxResults` of at least 100 and paginate until complete.
4. Collect every child Story, Task, Bug, and Spike unless the user excludes a type.
5. Fetch each child's summary and full description; refetch if content appears truncated(try **max 3** times).
6. Include Epic-level cases and meaningful cross-story end-to-end cases.

## Single work item workflow

1. Fetch the full description, summary, links, comments only when relevant, and any spike checklist or acceptance field.
2. Generate scenarios from available Jira content and figma diagram(if provided).
3. If the description is empty, use the summary only and state that coverage is limited.

## Scenario matrix

Create distinct cases across these categories:

- `Positive`: expected flows, each major requirement, each acceptance criterion, and each described workflow step.
- `Negative`: invalid data, wrong sequence, missing permission, failed processing, rejected action, or other Jira-described failure paths.
- `Edge Case`: empty values, maximum/minimum limits, duplicates, timeouts, pagination, missing metadata, and boundary conditions.
- `UI/UX`: visible copy, loading behavior, focus order, validation messages, diagrams/screenshots versus expected behavior.
- `End-to-End`: multi-step business flows, especially Epic flows across multiple child issues.

Stop adding rows only when additional rows would duplicate coverage, invent requirements, or exceed an explicit user cap.

## CSV header (exact)

```text
"Test Case ID","Jira ID","Priority","Scenario","Pre-conditions","Test Data","Steps","Expected Result","Scenario Type"
```

### Column rules

| Column | Values / notes |
|--------|----------------|
| Test Case ID | Stable id, e.g. `TC-001` |
| Jira ID | Default Epic key on every row for Epic packs; child key in Scenario/Test Data unless the user asks otherwise |
| Priority | `High` \| `Medium` \| `Low` |
| Scenario Type | `Positive` \| `Negative` \| `Edge Case` \| `UI/UX` \| `End-to-End` |
| Output path | `tests/test-cases/[PRIMARY_JIRA_KEY]_test_cases.csv` |

## CSV writing

- Quote every cell.
- Escape `"` as `""`.
- Put numbered test steps inside one quoted multiline field.
- Use tester-observable actions and outcomes.
- Do not dump the full CSV in chat after saving it.

## After saving

Print only the minimal Jira comment:

1. `Manual Test Cases Generated`
2. `CSV file:` as a clickable link to the generated CSV file created in the target GitLab repo.
3. Total test cases
4. Priority counts
5. Scenario Type counts

Do not include scope analyzed, issue type details, acceptance criteria notes, sub-task counts, parent Epic details, attachments, linked documents, Figma/design links, comments reviewed, Cloud Agent run URL, or generated-by footer unless explicitly requested.

## Also follow

- `rules/qa-jira-manual-tests.mdc`
- `commands/generate-tests.md`
- Agent: `agents/test-case-generator.md` when invoked as a subagent
