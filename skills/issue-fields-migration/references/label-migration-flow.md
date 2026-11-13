# Label Migration Flow

### Label Migration Flow

Use this flow when the user wants to convert repo labels into issue field values. Labels can only map to `single_select` issue fields (each label name maps to one option value).

#### Phase L1: Input & Label Discovery

1. Ask the user for: **org name** and **repo(s)** to migrate.
2. Fetch labels from each repo:

```bash
gh label list -R {owner}/{repo} --limit 1000 --json name,color,description
```

3. Fetch org issue fields:

```bash
gh api /orgs/{org}/issue-fields \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  --jq '.[] | {id, name, content_type, options: [.options[]?.name]}'
```

4. **Filtering** (for repos with many labels): if the repo has 50+ labels, group by common prefix (e.g., `priority-*`, `team-*`, `type-*`) or color. Let the user filter with "show labels matching priority" or "show blue labels" before mapping. Never dump 100+ labels at once.

5. Ask the user which labels map to which issue field and option. Support these patterns:
   - **Single label to single field**: e.g., label "bug" → Type field, "Bug" option
   - **Multiple labels to one field** (bulk): e.g., labels p0, p1, p2, p3 → Priority field with matching options
   - **Multiple labels to multiple fields**: e.g., p1 → Priority + frontend → Team. Handle as separate mapping groups.

6. **Auto-suggest mappings**: for each label, attempt to match issue field options using these patterns (in order):
   - **Exact match** (case-insensitive): label `Bug` → option `Bug`
   - **Prefix-number** (`{prefix}-{n}` → `{P}{n}`): label `priority-1` → option `P1`
   - **Strip separators** (hyphens, underscores, spaces): label `good_first_issue` → option `Good First Issue`
   - **Substring containment**: label `type: bug` → option `Bug`

   Present all suggestions at once for the user to confirm, correct, or skip.

**Example output:**

```
Labels in github/my-repo (showing relevant ones):
  p0, p1, p2, p3, bug, enhancement, frontend, backend

Org issue fields (single_select):
  Priority: Critical, P0, P1, P2, P3
  Type: Bug, Feature, Task
  Team: Frontend, Backend, Design

Suggested mappings:
  Label "p0" → Priority "P0"
  Label "p1" → Priority "P1"
  Label "p2" → Priority "P2"
  Label "p3" → Priority "P3"
  Label "bug" → Type "Bug"
  Label "frontend" → Team "Frontend"
  Label "backend" → Team "Backend"
  Label "enhancement" → (no auto-match; skip or map manually)

Confirm, adjust, or add more mappings?
```

#### Phase L2: Conflict Detection

After finalizing the label-to-option mappings, check for conflicts. A conflict occurs when an issue has multiple labels that map to the **same** issue field (since single_select fields can hold only one value).

1. Group label mappings by target issue field.
2. For each field with multiple label sources, note the potential for conflicts.
3. Ask the user for a conflict resolution strategy:
   - **First match**: use the first matching label found (by order of label mapping list)
   - **Skip**: skip issues with conflicting labels and report them
   - **Manual**: present each conflict for the user to decide

**Example:**

```
Potential conflict: labels "p0" and "p1" both map to the Priority field.
If an issue has both labels, which value should win?

Options:
  1. First match (use "p0" since it appears first in the mapping)
  2. Skip conflicting issues
  3. I'll decide case by case
```

#### Phase L3: Pre-flight Checks & Data Scan

1. For each repo, verify write access and cache the `repository_id`:

```bash
gh api /repos/{owner}/{repo} --jq '{full_name, id, permissions: .permissions}'
```

2. For each label in the mapping, fetch matching issues:

```bash
gh issue list -R {owner}/{repo} --label "{label_name}" --state all \
  --json number,title,labels,type --limit 1000
```

   **Warning**: `--limit 1000` silently truncates results. If you expect a label may have more than 1000 issues, paginate manually or verify the total count first (e.g., `gh issue list --label "X" --state all --json number | jq length`).

   **PR filtering**: `gh issue list` returns both issues and PRs. Include `type` in the `--json` output and filter for `type == "Issue"` if the user only wants issues migrated.

3. If **all selected labels return 0 issues**, stop and tell the user. Suggest: try different labels, check spelling, or try a different repository. Do not proceed with an empty migration.

4. For multi-repo migrations, repeat across all specified repos.

5. For each issue found:
   - Check if the issue already has a value for the target issue field (skip if set).
   - Detect multi-label conflicts (issue has two labels for the same field).
   - Apply the conflict resolution strategy chosen in Phase L2.
   - Classify: **migrate**, **skip (already set)**, **skip (conflict)**, or **skip (no matching label)**.

#### Phase L4: Preview / Dry-Run

Present a summary before any writes.

**Example preview:**

```
Label Migration Preview

Source: labels in github/my-repo
Target fields: Priority, Type, Team

| Category                | Count |
|-------------------------|-------|
| Issues to migrate       |   156 |
| Already set (skip)      |    12 |
| Conflicting labels (skip)|    3 |
| Total issues with labels|   171 |

Label breakdown:
  "p1" → Priority "P1": 42 issues
  "p2" → Priority "P2": 67 issues
  "p3" → Priority "P3": 38 issues
  "bug" → Type "Bug": 9 issues

Sample changes (first 5):
  github/my-repo#101: Priority → "P1"
  github/my-repo#203: Priority → "P2", Type → "Bug"
  github/my-repo#44:  Priority → "P3"
  github/my-repo#310: Priority → "P1"
  github/my-repo#7:   Type → "Bug"

After migration, do you also want to remove the migrated labels from issues? (optional)

Estimated time: ~24s (156 API calls at 0.15s each)

Proceed?
```

#### Phase L5: Execution

1. For each issue to migrate, write the issue field value (same endpoint as project field migration):

```bash
echo '{"issue_field_values": [{"field_id": FIELD_ID, "value": "OPTION_NAME"}]}' | \
  gh api /repositories/{repo_id}/issues/{number}/issue-field-values \
    -X POST \
    -H "X-GitHub-Api-Version: 2026-03-10" \
    --input -
```

   Replace `FIELD_ID` with the integer field ID (e.g., `1`) and `OPTION_NAME` with the option name string.

2. If the user opted to remove labels, remove each migrated label after successful field write:

```bash
gh api /repos/{owner}/{repo}/issues/{number}/labels/{label_name} -X DELETE
```

   URL-encode label names that contain spaces or special characters.

3. **Pacing**: 100ms delay between calls. Exponential backoff on HTTP 429 (1s, 2s, 4s, up to 30s).
4. **Progress**: report every 25 items (e.g., "Migrated 75/156 issues...").
5. **Error handling**: log failures but continue. Include label removal failures separately.
6. **Final summary**:

```
Label Migration Complete

| Result                | Count |
|-----------------------|-------|
| Fields set            |   153 |
| Labels removed        |   153 |
| Skipped               |    15 |
| Failed (field write)  |     2 |
| Failed (label remove) |     1 |

Failed items:
  github/my-repo#501: 403 Forbidden (insufficient permissions)
  github/my-repo#88:  422 Validation failed (field not available on repo)
  github/my-repo#120: label removal failed (404, label already removed)
```

---
