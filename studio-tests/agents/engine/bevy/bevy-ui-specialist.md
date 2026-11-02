# Agent Test Spec: bevy-ui-specialist

## Agent Summary
Domain: Bevy 0.20 UI: `bevy_ui` layout and `Val` units, headless `bevy_ui_widgets`, Feathers for tools, UI picking, input focus and keyboard and gamepad navigation.
Does NOT own: UX flow design (ux-designer), visual art style (art-director), `bsn!` scene structure and asset loading (bevy-assets-specialist), custom shaders beyond coordination on `UiMaterial` (bevy-render-specialist), gameplay code (gameplay-programmer).
No gate IDs assigned.

---

## Static Assertions (Structural)

- [ ] `description:` field is present and domain-specific (references bevy_ui / bevy_ui_widgets / Feathers / picking / focus)
- [ ] `tools:` list includes Read, Write, Edit, Bash, Glob, Grep
- [ ] Model tier is `sonnet` — frontmatter `model:` reads exactly `sonnet` (tiers: `.claude/docs/model-tiers.md`)
- [ ] Agent definition references `docs/engine-reference/bevy/VERSION.md`, `deprecated-apis.md` and `breaking-changes.md`, and sends uncertain APIs to docs.rs or the crate source
- [ ] Agent definition uses `Hovered` and `Pressed`, and names `Interaction` and `ui::widgets::Button` as deprecated
- [ ] Agent definition states that Feathers is for tools and not intended for game UI
- [ ] Agent definition states that `TabNavigationPlugin` is not in `DefaultPlugins` and that propagating `EmSize` is the app's job
- [ ] Agent definition does not claim authority over UX flow design or visual art direction
- [ ] Agent definition states there is no built-in data binding, screen stack or localization crate, and routes third-party crates to `docs/engine-reference/bevy/PLUGINS.md`

---

## Test Cases

### Case 1: In-domain request — appropriate output
**Input:** "Implement the main menu: Start, Options and Quit buttons, working with mouse and keyboard."
**Expected behavior:**
- Imports `Button` by explicit path from `bevy::ui_widgets`, because the deprecated one is still in the `ui` prelude
- Spawns each button with `Hovered::default()` and an `observe(|_: On<Activate>| ...)`, and styles from `&Hovered` and `Has<Pressed>`
- Adds `TabNavigationPlugin`, wraps the UI in a `TabGroup`, and gives each control a `TabIndex`
- Sizes text and spacing in `rem` or `em`, not fixed pixels
- Has the actions trigger a state change or message: the UI does not own game state
- Implements the flow it is given, without designing it
- Names a retained screenshot in `production/qa/evidence/` as the evidence

### Case 2: Out-of-domain redirect
**Input:** "Design the UX flow for the pause menu: what should happen when the player presses Back from Options?"
**Expected behavior:**
- Does NOT produce the flow design
- Explicitly states that interaction flow design belongs to `ux-designer`
- Redirects the request to `ux-designer`
- Notes it will implement whatever flow `ux-designer` specifies

### Case 3: Version awareness — pasted 0.19 code
**Input:** "Update this to 0.20. Also Bevy's own button.rs says to use `On<Insert, Pressed>`, so I used that:
```rust
fn style(q: Query<(&Interaction, &mut BackgroundColor), (Changed<Interaction>, With<Button>)>) { /* ... */ }
fn clicked(press: On<Pointer<Press>>) { /* ... */ }
fn pressed(on: On<Insert, Pressed>) { /* ... */ }
```"
**Expected behavior:**
- Checks `deprecated-apis.md` and `breaking-changes.md` and states that `Interaction` and `ui::widgets::Button` are deprecated
- Replaces the query with `&Hovered` and `Has<Pressed>`, and imports `bevy::ui_widgets::Button`
- Replaces `On<Pointer<Press>>` with `On<PointerPress>`, or `On<Activate>` if it means a button activation
- Rewrites `On<Insert, Pressed>` as `On<Insert<Pressed>>`, and explains that the comment in that example is stale and the types are the authority
- Does NOT keep the deprecated form because it still compiles

### Case 4: Feathers for game UI
**Input:** "Feathers looks great, let's use it for the player HUD and inventory."
**Expected behavior:**
- States, before writing anything, that Feathers' crate docs say it is deliberately not intended for game UI
- Offers `bevy_ui_widgets` with game-specific styling instead, and mentions copying Feathers' code as a starting point
- Leaves the decision to the user and `art-director`, rather than refusing or silently switching
- Is willing to use Feathers for a debug panel or tool, with `FeathersPlugins` and the `bevy_feathers` feature

### Case 5: Val::Em trap
**Input:** "I set a bigger font on the panel, but the children sized with `Val::Em` did not change."
**Expected behavior:**
- Explains that `Val::Em` resolves against the node's own `EmSize`, recomputed only from a `TextFont` on that entity
- States that `bevy_ui` does not propagate `EmSize`, so that is the app's job
- Offers a propagating system, or `rem` with the `RemSize` resource if everything should scale together
- Does NOT claim that `EmSize` inherits as in CSS

### Case 6: No built-in solution
**Input:** "How do I bind the health bar to the Health component, and what is the built-in screen stack?"
**Expected behavior:**
- States that Bevy has no data binding and no screen stack
- Uses change detection or an observer to update the bar, and states with `DespawnOnExit` or a small stack resource for screens
- Does NOT invent a binding API
- Keeps the UI reading game state and never writing it

---

## Protocol Compliance

- [ ] Stays within declared domain (`bevy_ui`, widgets, Feathers, picking, focus)
- [ ] Redirects UX flow design to ux-designer and visual style to art-director
- [ ] Reads the Bevy reference files before suggesting version-qualified APIs, and marks anything unverified rather than asserting it from memory
- [ ] Produces code in 0.20 form, not `Interaction`, `ui::widgets::Button` or `Pointer<Press>`
- [ ] Every interactive element works without a pointer
- [ ] Asks "May I write this to [filepath]?" naming the file before writing

---

## Coverage Notes
- Main menu (Case 1) should have a retained screenshot of each screen touched in `production/qa/evidence/`
- Case 3 is the primary version-awareness check, and includes a trap where Bevy's own comment is out of date
- Keyboard and gamepad navigation (Case 1) need a manual check; a screenshot does not show focus order
