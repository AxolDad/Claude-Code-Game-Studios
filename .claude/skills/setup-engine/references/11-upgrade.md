# /setup-engine — Section 11: Upgrade Subcommand

> Part of `/setup-engine`. `SKILL.md` names when to read this file; the settings it
> resolved and the rules under its later headings still apply here.

## 11. Upgrade Subcommand

**In this file:**

- Step 1 — Read Current Version State
- Step 2 — Fetch Migration Guide
- Step 3 — Pre-Upgrade Audit
- Step 4 — Confirm Before Updating
- Step 5 — Update VERSION.md
- Step 6 — Post-Upgrade Reminder

If invoked as `/setup-engine upgrade [old-version] [new-version]`:

### Step 1 — Read Current Version State

Read `docs/engine-reference/<engine>/VERSION.md` to confirm the current pinned
version, risk level, and any migration note URLs already recorded. If
`old-version` was not provided as an argument, use the pinned version from this
file.

### Step 2 — Fetch Migration Guide

Use WebSearch and WebFetch to locate the official migration guide between
`old-version` and `new-version`:

- Search: `"[engine] [old-version] to [new-version] migration guide"`
- Search: `"[engine] [new-version] breaking changes changelog"`
- Fetch the migration guide URL from VERSION.md if one is already recorded,
  or use the URL found via search.
- **Bevy** publishes one guide per minor release, at a fixed address:
  `https://bevy.org/learn/migration-guides/<old>-to-<new>/`, such as `0-19-to-0-20`.
  Fetch it directly, and fetch each one in turn when the upgrade crosses more than
  one minor. Look for a patch upgrade's changes (`0.20.0` → `0.20.1`) with the
  searches above.

Extract: renamed APIs, removed APIs, changed defaults, behavior changes, and
any "must migrate" items.

### Step 3 — Pre-Upgrade Audit

Scan the **code root** (resolve per `.claude/docs/code-root-resolution.md`) for code that uses APIs known to be deprecated or changed in the
target version:

- Use Grep to search for deprecated API names extracted from the migration
  guide (e.g., old function names, removed node types, changed property names)
- List each file that matches, with the specific API reference found
- On Bevy, scan `tests/` as well: tests are Rust that compiles against the same
  crate, so a renamed API breaks them too

Present the audit results as a table:

```
Pre-Upgrade Audit: [engine] [old-version] → [new-version]
==========================================================

Files requiring changes:
  File                              | Deprecated API Found       | Effort
  --------------------------------- | -------------------------- | ------
  src/gameplay/player_movement.gd   | old_api_name               | Low
  src/ui/hud.gd                     | removed_node_type          | Medium

Breaking changes to watch for:
  - [change description from migration guide]
  - [change description from migration guide]

Recommended migration order (dependency-sorted):
  1. [system/layer with fewest dependencies first]
  2. [next system]
  ...
```

If no deprecated APIs are found in the code root, report: "No deprecated API usage
found in the code root — upgrade may be low-risk."

### Step 4 — Confirm Before Updating

Ask the user before making any changes:

> "Pre-upgrade audit complete. Found [N] files using deprecated APIs.
> Proceed with upgrading VERSION.md to [new-version]?
> (This will update the pinned version and add migration notes — it does NOT
> change any source files. Source migration is done manually or via stories.)"

On Bevy, add that Step 5 also moves the crate, which changes `Cargo.toml` or
`Cargo.lock` but no source file, so across a minor the code will not compile until
the audit's files are migrated.

Wait for explicit confirmation before continuing.

### Step 5 — Update VERSION.md

After confirmation:

1. Update `docs/engine-reference/<engine>/VERSION.md`:
   - `Engine Version` → `[new-version]`
   - `Project Pinned` → today's date
   - `Last Docs Verified` → today's date
   - Re-evaluate and update the `Risk Level` and `Post-Cutoff Version Timeline`
     table if the new version falls beyond the LLM knowledge cutoff
   - Add a `## Migration Notes — [old-version] → [new-version]` section
     containing: migration guide URL, key breaking changes, deprecated APIs
     found in this project, and recommended migration order from the audit

2. If `breaking-changes.md` or `deprecated-apis.md` exist in the engine
   reference directory, append the new version's changes to those files.

3. If `project.yaml` exists at the repo root and has an `engine` block, Read it
   first, then update `engine.version` to `[new-version]` so the primary config
   store stays in sync with VERSION.md.

4. **On Bevy, move the crate — the version pin alone changes nothing that builds.**
   - Within a minor (`0.20.0` → `0.20.1`): `cargo update`.
   - Across minors: `cargo update` will not cross one, because Cargo treats each `0.x`
     as breaking. Ask "May I change `bevy = "<old>"` to `bevy = "<new>"` in
     `Cargo.toml`?", Edit that line, then run `cargo update`. Any third-party Bevy crate
     must move to a release for the new minor too: check them before editing.
     Declined: report `Cargo.toml still on [old]`, and say that
     `project-coherence.sh` reports a `[DIFFERS]` line until it moves.
   - Re-run Section 3's probe for the new version: its minimum Rust version may have
     risen. Update the `Installed at pin time` row with the result.

### Step 6 — Post-Upgrade Reminder

After updating VERSION.md, output:

```
VERSION.md updated: [engine] [old-version] → [new-version]

Next steps:
1. Migrate deprecated API usages in the [N] files listed above
2. Run /setup-engine refresh after upgrading the actual engine binary to
   verify no new deprecations were missed
3. Run /architecture-review — the engine upgrade may invalidate ADRs that
   reference specific APIs or engine capabilities
4. If any ADRs are invalidated, run /propagate-design-change to update
   downstream stories
```

On Bevy, step 2 reads instead: `Run cargo check and cargo check --tests, which report
what still needs migrating, then /setup-engine refresh` — there is no engine binary to
upgrade.

---
