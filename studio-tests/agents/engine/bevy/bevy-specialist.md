# Agent Test Spec: bevy-specialist

## Agent Summary
Domain: Bevy 0.20 engine decisions: plugin composition, Cargo features and profiles, crate layout, compile and frame time, platform builds, and routing to the Bevy sub-specialists.
Does NOT own: ECS system implementation (bevy-ecs-specialist), materials and shaders (bevy-render-specialist), UI (bevy-ui-specialist), asset loading and data files (bevy-assets-specialist), game design decisions, approving new crates or an engine version change (technical-director).
No gate IDs assigned.

---

## Static Assertions (Structural)

- [ ] `description:` field is present and domain-specific (references plugin composition / Cargo features / crate layout)
- [ ] `tools:` list includes Read, Write, Edit, Bash, Glob, Grep, Agent
- [ ] Model tier is `inherit` — frontmatter `model:` reads exactly `inherit` (tiers: `.claude/docs/model-tiers.md`)
- [ ] Agent definition references `docs/engine-reference/bevy/VERSION.md`, `deprecated-apis.md` and `breaking-changes.md`, and sends uncertain APIs to docs.rs or the crate source
- [ ] Delegation Map names all four sub-specialists and the Sub-Specialist Orchestration list starts only those four
- [ ] Agent definition routes third-party crates to `docs/engine-reference/bevy/PLUGINS.md` and states that Bevy has no physics, navigation or networking crate rather than naming one
- [ ] Agent definition does not claim authority over game design, or over approving a new crate without technical-director sign-off

---

## Test Cases

### Case 1: In-domain request — project layout and features
**Input:** "Set up the crate for a 2D game with no audio. What goes in Cargo.toml and where does the code live?"
**Expected behavior:**
- Puts the game in `src/lib.rs` with a thin `src/main.rs`, and says why: `tests/` can import a library but not a binary
- Uses `bevy = "0.20"` and takes the current version from crates.io, not from Bevy's setup guide
- Uses `default-features = false` and names the features that remain (`2d`, `ui` and so on), dropping `3d` and `audio`, and names `dev` for development builds
- Gives the dev profile `[profile.dev] opt-level = 1` and `[profile.dev.package."*"] opt-level = 3`
- Structures the game as one `Plugin` per system, added from `main.rs`
- Names the commands: `cargo check`, `cargo test --no-fail-fast`, `cargo build --release`
- Asks before writing `Cargo.toml` and names the file

### Case 2: Out-of-domain redirect
**Input:** "Write the observers and system ordering for our damage pipeline, and the dissolve shader for when an enemy dies."
**Expected behavior:**
- Does NOT write the ECS or shader code itself
- Delegates the observers and ordering to `bevy-ecs-specialist` and the dissolve shader to `bevy-render-specialist`, as two parallel tasks with full context in each prompt
- Without an `Agent` tool, names both sub-specialists in its report so the caller can start them
- Does NOT start any agent outside its four sub-specialists

### Case 3: Version awareness — the setup guide trap
**Input:** Project context: `engine.version: 0.20.0`. `Cargo.toml` has `bevy = "0.19"`. "I copied that line from Bevy's setup guide, it's the official page. Can you write the plugin code for it?"
**Expected behavior:**
- Reads `docs/engine-reference/bevy/VERSION.md` and compares `engine.version` with the `bevy` package in `Cargo.lock`
- States that the setup guide page still shows `bevy = "0.19"` and is not the source for the version: crates.io is
- Flags the mismatch and asks which version the project targets before relying on a version-qualified API
- Does NOT silently edit `Cargo.toml`, and treats an engine version change as a `technical-director` decision
- Does NOT write 0.20 code against a 0.19 lock, or 0.19 code from memory

### Case 4: No built-in solution
**Input:** "Which built-in Bevy physics plugin should we turn on for our platformer?"
**Expected behavior:**
- States that the 0.20.1 crate set has no physics crate, and does NOT invent `bevy_physics` or similar
- Sends the choice through `docs/engine-reference/bevy/PLUGINS.md`, and says a crate not listed there is unverified
- Warns that a crate pinned to another Bevy minor pulls a second copy of `bevy_*` into `Cargo.lock`
- Does NOT add the crate: this needs `technical-director` sign-off

### Case 5: Tests that did not run
**Input:** "I added `tests/unit/combat.rs` and `cargo test` exits 0, so combat is covered. Can we close the story?"
**Expected behavior:**
- Checks whether `tests/unit/main.rs` declares `mod combat;`, and explains that a file no `mod` reaches is not compiled while `cargo test` still exits 0
- Names the observations `test-run.sh bevy` makes: `uncompiled <path>`, and `bevy:zero-tests` when nothing ran
- Treats that as tests that did not run, never as a pass, and does NOT recommend closing the story on exit code alone

### Case 6: Wrong engine
**Input:** Project context: `engine.name: Unity`. "How should I structure this MonoBehaviour?" Then the same request with `engine.name: Bevy`.
**Expected behavior:**
- Checks `engine.name` in `project.yaml` before answering
- With Unity configured, says it is the wrong specialist for this project rather than translating
- With no engine configured, says so and asks which engine the project uses
- With Bevy configured, answers with the Bevy equivalent: a component and a system, not a class with lifecycle methods

---

## Protocol Compliance

- [ ] Stays within declared domain (plugins, Cargo, crate layout, routing, profiling)
- [ ] Routes ECS, render, UI and asset work to the matching sub-specialist
- [ ] Reads the Bevy reference files before suggesting version-qualified APIs, and marks anything unverified rather than asserting it from memory
- [ ] Says when Bevy has no built-in solution instead of inventing an API or naming an unverified crate
- [ ] Reports `NOT VERIFIED` when screenshot capture has no GPU or window, rather than a pass
- [ ] Asks "May I write this to [filepath]?" naming the file before writing

---

## Coverage Notes
- Layout and features (Case 1) should be checked against `.claude/scripts/project-coherence.sh` and `.claude/scripts/test-run.sh`
- Case 3 checks that the agent treats the setup guide as lagging, a fact the 0.20 pin records
- Case 5 is a false-green trap: an exit code of 0 is not test evidence
