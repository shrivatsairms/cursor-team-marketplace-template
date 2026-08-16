---
name: generate-tests
description: Generate exhaustive manual test case CSV from a Jira Epic or Story via Atlassian MCP
---

# Jira Epic / Story → Manual Test Cases (CSV)

**Canonical enforcement:** Apply this command together with [qa-jira-manual-tests.mdc](../rules/qa-jira-manual-tests.mdc) rule and the [senior-qa-engineer](../skills/senior-qa-engineer/SKILL.md) skill.

Output: `tests/[JIRA_KEY]_test_cases.csv` (RFC 4180).

---

## What the agent must do

### 1) Identify scope

- Fetch the issue and read **issue type** (Epic, Story, Task, Bug, …).

### 2) Deep fetch (Epic)

If the issue is an **Epic** (or user asked for full Epic scope):

1. Load Epic **summary** and **full description** (tables, links, embedded MIME matrices, Google Sheets).
2. Run **JQL**, e.g.  
   `jql`: `"Epic Link" = {KEY} OR parent = {KEY} OR "Parent Link" = {KEY}`  
   `maxResults`: `100` (or higher if needed).
3. **Paginate** until the result set reports **no more pages**.
4. Collect **every** child: Story, Task, Bug, Spike—unless the user excludes types.
5. For **each** child, ensure **summary + full description**. Refetch if truncated.
6. Treat **description** as the main source of acceptance-style content; pull custom Acceptance Criteria fields if exposed.
7. **Attachments / images:** note diagram and screenshot names; testers verify in Jira UI.

### 3) Deep fetch (single Story/Task/Bug)

- Full description, summary, links, and any spike checklists.

### 4) Build the scenario matrix (before writing CSV)

Cover **all** of:

| Bucket | Examples |
|--------|----------|
| Happy | Each major requirement, each AC bullet, each workflow step in description |
| Negative | Invalid input, failed scan, unauthorized, wrong order, infected files (lab policy) |
| Edge | Empty files, max-size files, long scan, duplicate rescan, pagination of large lists |
| UI/UX | Loading states, delayed preview, focus, messages, diagram vs. actual flow |
| End-to-End | Epic-only: chains that touch **multiple child stories** |

**Exhaustive scale (Epic — not a number target):**

- Add **every** manual scenario **implied** by Jira.
- **Every child issue** gets **multiple** rows with that child key in **Scenario** or **Test Data** when the ticket has enough content.
- **Do not** stop because a multiplier was hit—**do** stop when new rows would **duplicate** coverage or **require inventing** requirements. Respect an explicit user **cap**.

### 5) Approval (optional)

- If the user wants a **review gate**, list categorized scenarios and wait.
- If the user says **generate now** or uses the prompt template below, **write the CSV immediately**.

### 6) Write the CSV

- Path: `tests/test-cases/{JIRA_KEY}_test_cases.csv` within the root of the projects's Gitlab repo.
- **Jira ID column:** default = **Epic key on every row** for Epic packs; child key in **Scenario**.
- Strict RFC 4180: quoted fields, `""` for inner quotes, multiline **Steps** inside one cell.

### 7) Finish in Jira/comment

Print only:

- "Manual Test Cases Generated"
- `CSV file:` as a clickable link to the generated CSV file in the Gitlab repo.
- Total test cases  
- Counts by **Priority** (High / Medium / Low)  
- Counts by **Scenario Type** (Positive / Negative / Edge Case / UI/UX / End-to-End)  

Do not dump the full CSV to the terminal.
Do not include scope analyzed, issue type, acceptance criteria notes, sub-task count, parent Epic, attachments, linked documents, Figma/design links, comments reviewed, Cloud Agent run URL, or generated-by footer unless explicitly requested.

---

## Copy-paste prompt (use for every Epic)

Replace `{JIRA_EPIC_KEY}` (e.g. `DXP-17149`):

```text
Generate manual test cases for Epic {JIRA_EPIC_KEY}.

Use Jira MCP: fetch the Epic (full description, tables, links, embedded images/diagram references) and ALL child issues with JQL ("Epic Link" = {JIRA_EPIC_KEY} OR parent = {JIRA_EPIC_KEY} OR "Parent Link" = {JIRA_EPIC_KEY}); paginate until complete. Fetch full description for every child (Stories, Tasks, Bugs, Spikes); refetch any issue if the response was truncated.

Coverage: generate the maximum practical set of manual test cases **implied by Jira**—every distinct requirement, bullet, table row, workflow step, error path, attachment/diagram note, and meaningful cross-story End-to-End flow. For each child issue, include multiple rows that name the child key in Scenario or Test Data when the ticket is content-rich; do **not** use a fixed multiplier as a stopping rule—expand until further cases would duplicate coverage or require inventing requirements. Include dedicated Epic-level rows for the Epic summary/description and multi-story E2E cases.

Categories: Happy, Negative, Edge, UI/UX, End-to-End. Do not invent requirements; note if AC exist only in description or comments.

Output: tests/{JIRA_EPIC_KEY}_test_cases.csv (RFC 4180; Epic key in Jira ID column unless I say otherwise). No automated test code. After saving, print only the minimal Jira comment: CSV clickable link/download entry, total test cases, priority breakdown, and scenario-type breakdown.

Generate the CSV now (no scenario approval wait).
```

**Optional add-ons:**

- `Use child issue keys in the Jira ID column instead of the Epic key.`  
- `Cap total cases at {N}.`  
- `Exclude Bugs / Include only Stories.`  
- `List a short scenario outline first, then wait for my OK before CSV.`

---

## JQL reference

```text
"Epic Link" = DXP-17149 OR parent = DXP-17149 OR "Parent Link" = DXP-17149
```
