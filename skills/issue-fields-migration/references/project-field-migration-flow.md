# Project Field Migration Flow

### Project Field Migration Flow

Use this flow when the user wants to copy values from a GitHub Project V2 field to the corresponding org-level issue field.

Follow these six phases in order. Always preview before executing.

#### Phase P1: Input & Discovery

1. Ask the user for: **org name** and **project number** (or project URL).
2. Fetch project fields:

```bash
# Use MCP tool
mcp__github__projects_list(owner: "{org}", project_number: {n}, method: "list_project_fields")
```

3. Fetch org issue fields:

```bash
gh api /orgs/{org}/issue-fields \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  --jq '.[] | {id, name, content_type, options: [.options[]?.name]}'
```

4. **Filter out proxy fields**: after issue fields are enabled on a project, some project fields appear as "proxy" entries with empty `options: []` for single-select types. These mirror the real issue fields and should be ignored. Only match against project fields that have actual option values.

5. Auto-match fields by name (case-insensitive) with compatible types:

| Project Field Type | Issue Field Type | Compatible? |
|-------------------|-----------------|-------------|
| TEXT | text | Yes, direct copy |
| SINGLE_SELECT | single_select | Yes, option mapping needed |
| NUMBER | number | Yes, direct copy |
| DATE | date | Yes, direct copy |
| ITERATION | (none) | No equivalent; skip with warning |

6. Present the proposed field mappings as a table. Let the user confirm, adjust, or skip fields.

**Example output:**

```
Found 3 potential field mappings:

| # | Project Field      | Type          | Issue Field        | Status     |
|---|-------------------|---------------|--------------------|------------|
| 1 | Priority (renamed) | SINGLE_SELECT | Priority           | Auto-match |
| 2 | Due Date           | DATE          | Due Date           | Auto-match |
| 3 | Sprint             | ITERATION     | (no equivalent)    | Skipped    |

Proceed with fields 1 and 2? You can also add manual mappings.
```

#### Phase P2: Option Mapping (single-select fields only)

For each matched single-select pair:

1. Compare option names between the project field and issue field (case-insensitive).
2. Auto-match options with identical names.
3. For any unmapped project field options, present **all** unmapped options in a single summary and ask the user to provide mappings for all of them at once. Do not prompt one-by-one; batch them into a single exchange.
4. Show the final option mapping table for confirmation.

**Example output:**

```
Option mapping for "Release - Target":

Auto-matched (case-insensitive):
  "GA" → "GA"
  "Private Preview" → "Private Preview"
  "Public Preview" → "Public Preview"

Unmapped project options (need your input):
  1. "Internal Only" → which issue field option? (or skip)
  2. "Retired" → which issue field option? (or skip)
  3. "Beta" → which issue field option? (or skip)
  4. "Deprecated" → which issue field option? (or skip)

Available issue field options not yet mapped: "Internal", "Sunset", "Beta Testing", "End of Life"

Please provide mappings for all 4 options above (e.g., "1→Internal, 2→Sunset, 3→Beta Testing, 4→skip").
```

#### Phase P3: Pre-flight Checks

Before scanning items, verify write access to each repository that may be touched:

1. From the project items (first page), collect the unique set of `{owner}/{repo}` values.
2. For each unique repo, verify the authenticated user has Issues write permission:

```bash
gh api /repos/{owner}/{repo} --jq '{full_name, permissions: .permissions}'
```

3. If any repo shows `push: false` or `triage: false`, warn the user before proceeding. Items in those repos will fail at write time.
4. Cache the `repository_id` (integer) for each repo now; you will need it in Phase 6:

```bash
gh api /repos/{owner}/{repo} --jq .id
```

#### Phase P4: Data Scan

1. Fetch all project items using MCP. **Important**: for projects with more than ~200 items, `gh api graphql --paginate` is unreliable (it concatenates JSON responses without proper separators and can time out). Use the MCP tool which handles pagination internally, or use explicit cursor-based pagination:

```bash
# Preferred: use MCP tool (handles pagination automatically)
mcp__github__projects_list(owner: "{org}", project_number: {n}, method: "list_project_items")

# Fallback for large projects: manual cursor-based pagination
# Fetch 100 items per page, advancing the cursor each time.
# Process each page before fetching the next to avoid memory issues.
# Save progress (page number or last cursor) so you can resume if interrupted.
```

2. For each item:
   - Skip if it is a draft item (not a real issue).
   - Extract the source project field value.
   - Skip if the source value is empty.
   - Check if the issue already has a value for the target issue field:

```bash
gh api /repos/{owner}/{repo}/issues/{number}/issue-field-values \
  -H "X-GitHub-Api-Version: 2026-03-10"
```

   - If the issue field already has a value, skip it (preserve existing data).

3. Classify each item into one of:
   - **Migrate**: has source value, no existing target value
   - **Skip (already set)**: target issue field already has a value
   - **Skip (no source)**: project field is empty for this item
   - **Skip (draft)**: item is a draft, not a real issue
   - **Skip (unmapped option)**: single-select value was not mapped

#### Phase P5: Preview / Dry-Run

Present a summary before any writes.

**If user requested dry-run**: show the full detailed report (every issue, its current value, proposed new value, and skip reason) and stop. Do not execute.

**Otherwise (preview mode)**: show summary counts and a sample of changes, then ask for confirmation.

**Example preview:**

```
Migration Preview for Project #42

Fields to migrate: Priority, Due Date

| Category               | Count |
|------------------------|-------|
| Items to migrate       |   847 |
| Already set (skip)     |    23 |
| No source value (skip) |   130 |
| Draft items (skip)     |    12 |
| Total project items    | 1,012 |

Sample changes (first 5):
  github/repo-a#101: Priority → "High"
  github/repo-a#203: Priority → "Medium", Due Date → "2025-03-15"
  github/repo-b#44:  Priority → "Low"
  github/repo-a#310: Due Date → "2025-04-01"
  github/repo-c#7:   Priority → "Critical"

Estimated time: ~127s (847 API calls at 0.15s each)

Proceed with migration? This will update 847 issues across 3 repositories.
```

#### Phase P6: Execution

1. Use the `repository_id` values cached in Phase 3.

2. For each item to migrate, write the issue field value:

```bash
echo '{"issue_field_values": [{"field_id": FIELD_ID, "value": "VALUE"}]}' | \
  gh api /repositories/{repo_id}/issues/{number}/issue-field-values \
    -X POST \
    -H "X-GitHub-Api-Version: 2026-03-10" \
    --input -
```

   Replace `FIELD_ID` with the integer field ID (e.g., `1`) and `VALUE` with the value string.

3. **Pacing**: add a 100ms delay between API calls. On HTTP 429 responses, use exponential backoff (1s, 2s, 4s, up to 30s).
4. **Progress**: report status every 25 items (e.g., "Migrated 75/847 items...").
5. **Error handling**: log failures but continue processing remaining items.
6. **Final summary**:

```
Migration Complete

| Result  | Count |
|---------|-------|
| Success |   842 |
| Skipped |   165 |
| Failed  |     5 |

Failed items:
  github/repo-a#501: 403 Forbidden (insufficient permissions)
  github/repo-b#88:  422 Validation failed (field not available on repo)
  ...
```
