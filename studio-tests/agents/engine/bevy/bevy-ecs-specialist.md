# Agent Test Spec: bevy-ecs-specialist

## Agent Summary
Domain: Bevy ECS in Rust: components and resources, schedules and system ordering including weak ordering, observers, messages, states, queries, exclusive systems, fallible systems, and ECS test patterns.
Does NOT own: materials, shaders and render-world code (bevy-render-specialist), UI layout and widgets (bevy-ui-specialist), asset loading and data files (bevy-assets-specialist), Cargo features, new crates and version decisions (bevy-specialist), game mechanics (design docs).
No gate IDs assigned.

---

## Static Assertions (Structural)

- [ ] `description:` field is present and domain-specific (references schedules / chain_weak / observers / states / queries)
- [ ] `tools:` list includes Read, Write, Edit, Bash, Glob, Grep
- [ ] Model tier is `sonnet` — frontmatter `model:` reads exactly `sonnet` (tiers: `.claude/docs/model-tiers.md`)
- [ ] Agent definition references `docs/engine-reference/bevy/VERSION.md`, `deprecated-apis.md` and `breaking-changes.md`, and sends uncertain APIs to docs.rs or the crate source
- [ ] Agent definition uses `On<Add<A>>` for lifecycle observers, and shows `On<Add, A>` only as the old form
- [ ] Agent definition explains when `.chain_weak()` does not hold (interior mutability, channels, atomics, `NonSend`)
- [ ] Agent definition does not claim authority over rendering, UI or asset loading
- [ ] Agent definition routes third-party crate questions to `docs/engine-reference/bevy/PLUGINS.md`

---

## Test Cases

### Case 1: In-domain request — appropriate output
**Input:** "Implement the damage pipeline: hits are detected, damage is applied, then dead enemies are removed. It only runs while the game is in the Playing state."
**Expected behavior:**
- Writes separate systems for each step, ordered with `.chain()` or sets, and gated with `.run_if(in_state(GameState::Playing))`
- Carries hits between systems as a `#[derive(Message)]` with `MessageWriter`, `MessageReader` and `app.add_message()`, or justifies an observer instead
- Reads damage values from data or a resource, never from constants in a system
- Returns `Result` from fallible systems and uses `?` rather than `unwrap()`
- Writes a test in `tests/unit/[system].rs` declared with `mod [system];` in `tests/unit/main.rs`, with a function named `[scenario]_[expected]`, and adds `StatesPlugin` to the headless test app
- Does NOT decide the damage formula: it comes from the design docs

### Case 2: Out-of-domain redirect
**Input:** "Write the WESL shader that flashes the enemy white when it takes damage."
**Expected behavior:**
- Does NOT write the shader
- Redirects the shader and material work to `bevy-render-specialist`
- Offers the ECS half it does own: the system or observer that starts and ends the flash

### Case 3: Version awareness — pasted 0.19 code
**Input:** "This compiled on 0.19 and now errors. Fix it:
```rust
app.add_observer(|on: On<Add, Health>, mut commands: Commands| { commands.entity(on.entity).insert(HealthBar); });
fn sync(list: Res<Targets>, q: Query<&Health>, mut next: ResMut<NextState<Phase>>) {
    for health in q.iter_many(&list.0) { if health.0 <= 0 { next.set_if_neq(Phase::Over); } }
}
```"
**Expected behavior:**
- Checks `deprecated-apis.md` and `breaking-changes.md` and states that the code is 0.19
- Rewrites the observer as `On<Add<Health>>`
- States that `iter_many` now yields `Result`, and either handles the `Err` or calls `.matched()` deliberately, saying which and why
- Replaces `set_if_neq`, a deprecated wrapper, with `set_if_different`
- Does NOT reintroduce `On<Add, A>`, and does NOT treat the deprecated call as acceptable because it still compiles

### Case 4: Weak ordering with a hidden dependency
**Input:** "These three systems talk through a crossbeam channel. Can I swap `.chain()` for `.chain_weak()` to get more parallelism?"
**Expected behavior:**
- Says no: weak ordering holds only between tracked data accesses, and a channel is not tracked
- Keeps `.chain()`, or offers to express the dependency as `ResMut<T>` in the producer and `Res<T>` in the consumer so `.chain_weak()` works
- Suggests `ScheduleBuildSettings::shuffle_seed` (needs the `debug` feature) to expose the bug, and logs the seed
- Does NOT present `.chain_weak()` as a free speed-up

### Case 5: Exclusive system conflict
**Input:** "`&mut World` is a normal parameter in 0.20, right? So I can write `fn refresh(time: Res<Time>, world: &mut World)` and run it every frame."
**Expected behavior:**
- Confirms that `&mut World` is an ordinary `SystemParam` and need not come first
- States that it still must not conflict with other parameters, and that `Res<Time>` conflicts with it
- Offers a form that does not conflict, such as reading `Time` from the `world`, or a non-exclusive system
- Notes the system blocks everything else in its schedule while it runs, and asks whether it needs to be exclusive at all

### Case 6: State transition reruns
**Input:** "Clicking Pause twice replays my pause menu spawn in `OnEnter(Paused)`."
**Expected behavior:**
- Explains that `next.set(s)` runs `OnExit` and `OnEnter` even for the same state
- Replaces it with `next.set_if_different(s)`, and notes that a `set` with that state in the same frame still runs the transition
- Uses `DespawnOnExit(Paused)` for cleanup rather than a manual query loop

---

## Protocol Compliance

- [ ] Stays within declared domain (systems, schedules, observers, messages, states, queries, Rust patterns)
- [ ] Redirects render, UI and asset work to the matching Bevy sub-specialist
- [ ] Reads the Bevy reference files before suggesting version-qualified APIs, and marks anything unverified rather than asserting it from memory
- [ ] Produces code in 0.20 form, not `On<Add, A>`, `set_if_neq` or a bare `iter_many` loop
- [ ] Places tests where they compile, with the `mod` declaration
- [ ] Asks "May I write this to [filepath]?" naming the file before writing

---

## Coverage Notes
- Case 1 should have a unit test declared in `tests/unit/main.rs`; a test file no `mod` reaches does not count
- Case 3 is the primary version-awareness check, and covers three separate 0.20 changes in one paste
- Case 4 verifies the agent understands weak ordering rather than only knowing its name
- `&mut World` conflict (Case 5) can be confirmed by compiling the example, which only the project's build owner runs
