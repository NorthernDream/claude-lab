# claude-lab — experimental sandbox

This project exists to prototype Claude Code configuration: skills, hooks, permissions, agents, settings, workflows. The directory name (`experiments/`) is literal.

## Workflow contract

**Core rule: never edit `~/.claude/` directly.** Prove the change inside `claude-lab/` first, get user verification, then write it through to global.

**Bilingual maintenance:** any edit to an agent-facing markdown file in this project (`CLAUDE.md`, `skills/<name>/SKILL.md`, etc.) must update its `.zh.md` sibling in the same flow — English is canonical, `.zh.md` mirrors it. For new files, create the `.zh.md` sibling from scratch in the same flow. JSON / settings files have no translation requirement. This applies to every flow below; if `.zh.md` is out of sync at flow close, the flow is not done.

### Modifying global content via project staging

The pattern: stage the change in project → user verifies → write back to global → clean up. The setup and writeback mechanics depend on what you're touching.

1. **Set up the project copy.**
   - **New content** (file, section, settings entry): write it directly in project — `.claude/skills/<name>/`, `.claude/CLAUDE.md` below the STAGED FOR GLOBAL banner, `.claude/settings.local.json`, etc.
   - **Existing skill at `~/.claude/skills/<name>/`:** `cp -r ~/.claude/skills/<name> .claude/skills/<name>`. For plugin skills (under `~/.claude/plugins/cache/...`), locate via `find ~/.claude/plugins/cache -name '<name>' -type d` and substitute the source path.
   - **Existing section in `~/.claude/CLAUDE.md`:** copy the section text into the project file's banner zone (below STAGED FOR GLOBAL).

2. **Add an in-flight tracker row** — status=testing, scope=global-candidate, File=<project location>. For skill checkouts, set Read-instead = `.claude/skills/<name>/SKILL.md`; otherwise blank.

3. **Edit** the project copy.

4. **Stop and ask the user to verify.**

   For skill checkouts: read the project path with the Read tool. **Do NOT call `Skill(<name>)`** while the row is present — the global copy wins silently. (Sentinel-verified 2026-05-02. Agents under `~/.claude/agents/<name>/` likely behave the same way; run a sentinel before relying on this for agents.)

5. **User confirms** → tracker row status `verified`.

6. **Write back** — single edit, three sub-actions:

   - 6a. **Apply to global:**
     - **New content:** `cp` into `~/.claude/`.
     - **Existing skill:** `rsync -a --delete .claude/skills/<name>/ ~/.claude/skills/<name>/`. The `--delete` is critical — files removed from the project copy must vanish from global. (`cp *` would leave orphans.)
     - **Existing CLAUDE.md section:** replace the corresponding section in `~/.claude/CLAUDE.md` with the project version.
   - 6b. **Remove from project:**
     - **New content / CLAUDE.md section:** remove the file or section text from the project file.
     - **Existing skill:** `rm -rf .claude/skills/<name>`.
   - 6c. **Delete the tracker row.**

7. Snapshot to `claude-archive/`.

### Modifying global permissions in `~/.claude/settings.json`

Permissions are additive — `~/.claude/settings.json` always wins, and `settings.local.json` can only add, not shadow. The project-staging pattern only covers addition.

1. Add an in-flight tracker row (status=testing, scope=global-candidate, File=`.claude/settings.local.json`; Read-instead blank).
2. Add the new/changed permission to `claude-lab/.claude/settings.local.json`.
3. Test in claude-lab.
4. **User confirms** → tracker row status `verified`.
5. Add the permission to `~/.claude/settings.json` first — order matters: doing this before removing local avoids a permission gap where neither file allows the command.
6. Remove the redundant entry from `claude-lab/.claude/settings.local.json`, **delete the tracker row**.
7. Snapshot to `claude-archive/`.

**Revoking an existing global permission**: no project-side preview path (additivity). With explicit user pre-approval, edit `~/.claude/settings.json` directly to remove the entry, then snapshot. No tracker row needed for this single-step direct edit.

### Special cases
- **Plugin-provided skills** (e.g. `superpowers:*`, anything under `~/.claude/plugins/cache/...`) must follow the skill-checkout path above — direct edits get clobbered on plugin update.
- **No project-scoped form** (e.g. `~/.ssh/config` aliases, `~/.claude/settings.json` keys with no `settings.local.json` equivalent like `effortLevel`): say so explicitly and get confirmation before touching global. Note in the tracker as `out-of-scope` with the reason.
- **Escape hatch — trivial doc edits**: pure typo fixes / description tweaks under `~/.claude/`, when the user explicitly approves "skip the dance," can be edited directly. Default is still to checkout.
- **Abandon path**: at any point during the procedure, revert any project edits, delete the tracker row, no archive needed. For skill checkouts, also `rm -rf .claude/skills/<name>` to clean the checkout.

### Archiving
After any procedure above completes — including the final tracker-row removal — snapshot the resulting `.claude/` to `claude-archive/`:
```bash
cp -r .claude claude-archive/$(date +%Y-%m-%d)-<short-tag>
```
Tag should describe what changed in this snapshot (e.g. `promotion-contract`, `redis-cluster-v2`). This is the rollback safety net. Skip only if `.claude/` is byte-identical to the most recent snapshot.

## In-flight tracker

Single worktable for any active change to `.claude/`. Empty = nothing being experimented on. Rows deleted on `promoted` / `accepted` / `abandoned`.

The **`Read-instead`** column is populated only for skill checkouts — it directs the agent to `Read` the project copy directly because `Skill(<name>)` invocation silently loads the stale global copy. Other rows leave it blank.

**Out of scope:** anything outside `.claude/` (e.g. `~/.ssh/config` aliases, `~/.ssh/id_*` keys). Mention in conversation, don't enter here.

**Scope:** `local-only` | `global-candidate`
**Status:** `testing` → `verified` → (`promoted` for global-candidate / `accepted` for local-only / `abandoned` for any ⇒ delete row)

| Entry | File | Scope | Read-instead | Status | Notes |
|---|---|---|---|---|---|
| *(empty — nothing currently in flight)* | | | | | |

<!-- STAGED FOR GLOBAL: everything below this banner — to the end of the file — is destined for
     ~/.claude/CLAUDE.md once user-approved. The banner itself is NOT copied; only the headings
     and bodies below it. File convention: project-only content (workflow contract, in-flight
     tracker, etc.) lives ABOVE this banner; global-candidate content lives BELOW. Promoting an
     existing section to global (or demoting back to project-only) = move it across this banner;
     no other bookkeeping needed. -->
