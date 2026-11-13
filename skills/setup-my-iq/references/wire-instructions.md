# 10. Wire the files into a custom instructions file

### 10. Wire the files into a custom instructions file

Add pointer entries to one of the user's custom instructions files so other
skills and agents can find the new files. **Recommend user scope, not
workspace scope.** The whole point of this context is that it's available
everywhere, not tied to one repo.

Quick map: 10a detect harnesses, 10b pick the target file, 10c (only if more
than one harness) offer to symlink them to a canonical file, 10d write the
pointers using the bundled template.

On macOS or Linux, every command and path below has a POSIX equivalent.
Substitute `$HOME` for `%USERPROFILE%` in every path, use `ln -s
<canonical-target> <harness-file>` instead of `New-Item -ItemType
SymbolicLink`, and skip the Developer Mode check entirely (symlinks don't
require elevation on macOS or Linux).

**Step 10a. Detect which harnesses the user uses.** Check whether any of these
user-scope paths exist (use the filesystem; no need to ask first):

| Harness | User-scope custom instructions path |
|---------|--------------------------------------|
| VS Code Copilot Chat / Claude Code | `%USERPROFILE%\.claude\CLAUDE.md` |
| GitHub Copilot CLI | `%USERPROFILE%\.copilot\copilot-instructions.md` |

VS Code Copilot Chat does not natively read `%USERPROFILE%\.agents\AGENTS.md`,
but it does read `%USERPROFILE%\.claude\CLAUDE.md`, which is the same file
Claude Code uses. So `.claude\CLAUDE.md` is the shared user-scope file for
those two harnesses.

If `%USERPROFILE%\.agents\AGENTS.md` exists, treat it as the canonical
target. In some setups the harness-specific files above are symlinked to
it, so a single write flows to every wired-up harness.

**Step 10b. Pick the target.** Apply the first rule that matches:

1. If `%USERPROFILE%\.agents\AGENTS.md` exists, default to it. The
   harness-specific files are symlinks to it, so a single write flows to
   every wired-up harness. Confirm with the user before writing.
2. Else if exactly one harness-specific file exists, propose writing to it.
   Confirm. Also mention the canonical-plus-symlink option in step 10c.
3. Else if both harness-specific files exist
   (`copilot-instructions.md` and `CLAUDE.md`), list them and ask which to
   update. Strongly recommend creating `.agents\AGENTS.md` as the canonical
   file and symlinking the others to it so a single edit reaches all
   harnesses (step 10c).
4. Else (no user-scope custom instructions files exist anywhere), ask:
   "Which AI harness do you use most: VS Code Copilot Chat, Claude Code,
   GitHub Copilot CLI, something else?" Then create the matching
   user-scope file (`.claude\CLAUDE.md` for the first two,
   `.copilot\copilot-instructions.md` for Copilot CLI) with a
   `## Personal Context` section.

If the user explicitly asks for workspace scope instead, write to
`<workspace>\AGENTS.md` and tell them the trade-off: it only applies to
that repo.

**Step 10c. Offer to symlink other harnesses to the canonical file.** Only
relevant when the user has (or wants) more than one harness. Ask: "Want me
to make `%USERPROFILE%\.agents\AGENTS.md` the canonical file and symlink
your other harness files to it? That way you edit once and every harness
sees the change."

If yes, for each other harness file the user wants linked:

1. **Prerequisites.** File symlinks on Windows need either Developer Mode
   enabled or an elevated PowerShell. Check Developer Mode with:

   ```powershell
   Get-ItemPropertyValue 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock' AllowDevelopmentWithoutDevLicense -ErrorAction SilentlyContinue
   ```

   Returns `1` if Dev Mode is on. If it isn't, offer the user two options
   and let them pick:

   - **Option A: Enable Developer Mode** (one-time, no restart needed).
     Settings -> Privacy & Security -> For developers -> turn on Developer
     Mode. Or run this in an elevated PowerShell:

     ```powershell
     New-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock' -Name AllowDevelopmentWithoutDevLicense -Value 1 -PropertyType DWord -Force
     ```

   - **Option B: Run the symlink command yourself in an elevated
     PowerShell.** Hand them the exact command (filled in with the real
     paths) and ask them to run it in a PowerShell window opened as
     Administrator. Then continue once they confirm it succeeded:

     ```powershell
     New-Item -ItemType SymbolicLink -Path '<harness-file>' -Target "$env:USERPROFILE\.agents\AGENTS.md"
     ```

   Don't keep going without either Dev Mode on or confirmation that the user
   ran the elevated command.
2. **Back up the existing file** to `<file>.bak-<yyyyMMddHHmmss>` so nothing
   is lost. Never delete.
3. **Create the symlink** pointing to `%USERPROFILE%\.agents\AGENTS.md`
   (skip if the user already ran it themselves under Option B):

   ```powershell
   New-Item -ItemType SymbolicLink -Path '<harness-file>' -Target "$env:USERPROFILE\.agents\AGENTS.md"
   ```

The inline symlink steps above are enough for context wiring.

If the user declines symlinks, proceed with the per-harness write and move
on. Don't push twice.

**Step 10d. Write the pointers.** Use the bundled AGENTS.md template at
`assets/templates/AGENTS.md` (relative to this SKILL.md) as the source of
truth for the `## Personal Context` block structure (header, entry labels,
routing table, rules). Don't reinvent the format.

Behavior:

- **If the target file doesn't exist on disk yet**, create it. Make any
  missing parent directory (e.g., `%USERPROFILE%\.copilot\` or
  `%USERPROFILE%\.claude\`), copy the entire bundled template into the
  target path, replace `<CONTEXT_DIR>` with the absolute path to the
  context directory chosen in step 1, and write it.
- **If the target file exists but has no `## Personal Context` section**,
  append the section to the end of the file (and add the `## Safety`
  section from the template if it's missing too), with `<CONTEXT_DIR>`
  replaced. Don't touch existing content above it.
- **If the `## Personal Context` section already exists**, only add or
  update the specific entries that changed. Do not rewrite the whole
  section. Keep the existing labels and ordering; only swap the `@<path>`
  lines.
- Only wire up files that actually exist on disk. Don't add `@<path>`
  pointers for files the user skipped.
