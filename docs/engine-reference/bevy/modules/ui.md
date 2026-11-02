# Bevy UI — Quick Reference

Last verified: 2026-11-02 | Engine: Bevy 0.20

## What Changed Since ~0.16 (LLM Cutoff)

- `ui::Interaction` and `ui::widgets::Button` deprecated → `Hovered`, `Pressed`, `ui_widgets::Button` (0.20)
- Pointer events flat: `On<PointerPress>` (0.20)
- `Val::Em`, `Val::Rem` with `em()`, `rem()`; `TextFont` default size is `FontSize::Rem(1.)` (0.20)
- `BorderRadius` fields are 2D `CornerRadius` (0.20); `border_radius` is a `Node` field (0.18)
- `bevy_ui_widgets` in `ui` feature, plugins in `DefaultPlugins` (0.19); `Core` prefix removed (0.19)
- Parley text: `TextFont.font: FontSource`, `font_size: FontSize` (0.19)
- `EditableText` and `TextInput` are separate components (0.20)
- `UiSystems` sets ordered weakly (0.20); UI rendering retained (0.20)

## Which Toolkit

| Toolkit | Use for | Notes |
|---------|---------|-------|
| `bevy_ui` + `bevy_ui_widgets` | Game UI: menus, HUD | Headless. You supply all styling. In `default`. The 0.19 guide calls the crate immature and subject to heavy breaking changes |
| Feathers (`bevy_feathers` feature) | Editors, inspectors, debug tools | Crate docs: "deliberately not intended" for game UI. Not in `default` |

## Current API Patterns

### Button

```rust
use bevy::{
    picking::hover::Hovered,
    prelude::*,
    ui::Pressed,
    ui_widgets::{observe, Activate, Button},
};

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Update, style_buttons)
        .run();
}

fn setup(mut commands: Commands, assets: Res<AssetServer>) {
    commands.spawn(Camera2d);
    commands.spawn((
        Button,
        Hovered::default(),
        Node {
            width: px(150),
            height: px(65),
            border: UiRect::all(px(5)),
            justify_content: JustifyContent::Center,
            align_items: AlignItems::Center,
            border_radius: BorderRadius::MAX,
            ..default()
        },
        BorderColor::all(Color::BLACK),
        BackgroundColor(Color::srgb(0.15, 0.15, 0.15)),
        observe(|_activate: On<Activate>| info!("clicked")),
        children![(
            Text::new("Start"),
            TextFont {
                font: assets.load("fonts/FiraSans-Bold.ttf").into(),
                font_size: FontSize::Px(33.0),
                ..default()
            },
        )],
    ));
}

fn style_buttons(
    mut buttons: Query<(&Hovered, Has<Pressed>, &mut BackgroundColor), With<Button>>,
) {
    for (hovered, pressed, mut color) in &mut buttons {
        *color = match (hovered.get(), pressed) {
            (_, true) => Color::srgb(0.35, 0.75, 0.35).into(),
            (true, false) => Color::srgb(0.25, 0.25, 0.25).into(),
            (false, false) => Color::srgb(0.15, 0.15, 0.15).into(),
        };
    }
}
```

Wrong (deprecated; and the prelude still exports the old `Button`, so name the new one
in a `use`):

```rust
// fragment
fn style(q: Query<(&Interaction, &mut BackgroundColor), Changed<Interaction>>) {}
```

Add `ActivateOnPress` to fire on press. `InteractionDisabled` disables a widget.

### Widgets

Headless: `Button`, `Checkbox`, `Slider`, `TextInput`, `RadioGroup`, list, menu, popover,
dialog, modal, scroll area, scrollbar, tabs. State is **external**: the widget emits
`ValueChange<T>` with the proposed value, and the app writes it back (`SliderValue`,
`Checked`), or uses a self-update helper such as `checkbox_self_update`.
Examples: `standard_widgets`, `standard_widgets_observers`, `headless_tabs`.

### Text Input

`TextInput` and `EditableText` are both required. Read with `editable.value()`, change
with `queue_edit` or `clear`. System clipboard needs the `system_clipboard` feature.

```rust
// fragment
use bevy::{text::{EditableText, TextCursorStyle}, ui_widgets::TextInput};

commands.spawn((
    TextInput,
    EditableText { visible_width: Some(10.), allow_newlines: false, ..Default::default() },
    TextLayout::no_wrap(),
    TextFont { font_size: FontSize::Px(24.0), ..default() },
    TextCursorStyle::default(),
    Node::default(),
));
```

### Feathers

Enable the `bevy_feathers` feature, add `FeathersPlugins`, insert
`UiTheme(create_dark_theme())`. Controls are BSN scenes: `@FeathersButton`,
`Children [ @caption("Run") ]`, reacting through `on(|_: On<Activate>| ..)`. Full
version: `examples/ui/widgets/feathers_counter.rs`.

### Units

`px`, `percent`, `vw`, `vh`, `em`, `rem`, `auto`. `RemSize` is a resource (default 20) and
`EmSize` a component that `Node` requires; propagating `EmSize` is the app's job.

## Common Mistakes

- `Interaction`, `On<Pointer<Press>>`, `font_size: 33.0`, `BorderRadius` as a component
- Using Feathers for the game's HUD
- `EditableText` without `TextInput`, or the reverse. Neither works alone
- A dialog's `Escape` handler now fires on the same press that blurs a text field
- Expecting a widget to update its own value. Handle `ValueChange`
- Mixing forms: inside `bsn!` `font_size: px(33.0)`, in Rust `FontSize::Px(33.0)`

## Examples and Sources

`examples/ui/widgets/` (`button`, `feathers_counter`, `feathers_gallery`,
`standard_widgets`), `examples/ui/text/` (`text_input`, `inline_image`),
`examples/ui/navigation/`. Focus: `modules/navigation.md`.
https://bevy.org/news/bevy-0-20/, https://bevy.org/learn/migration-guides/0-19-to-0-20/
