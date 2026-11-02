# Bevy Audio — Quick Reference

Last verified: 2026-11-02 | Engine: Bevy 0.20

## What Changed Since ~0.16 (LLM Cutoff)

- Audio is a separate feature: `default` is `2d, 3d, ui, audio`, and `2d`/`3d` do not imply it (0.19)
- Rodio 0.22. Formats are features: `vorbis` on by default; `wav`, `mp3`, `mp4`, `flac`, `aac`; `audio-all-formats` for all. Minimum Android API 26 (0.19)
- `Volume` is an enum, `Volume::Linear(x)` or `Volume::Decibels(x)`, with no `+`/`-` operators

## What Bevy Provides

Playback by entity, looping, volume, speed, pause and mute, seek, stereo spatial panning,
one global volume, pitch tones, custom `Decodable` sources.

**Not provided:** mixer buses, sends, effects such as reverb or filters, an audio editor,
and HRTF. The source says spatial audio is "simple left-right stereo panning". For the
rest see `PLUGINS.md`.

## Current API Patterns

### Play and Control

```rust
use bevy::{audio::Volume, prelude::*};

#[derive(Component)]
struct Music;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Update, toggle_music)
        .run();
}

fn setup(mut commands: Commands, assets: Res<AssetServer>) {
    commands.spawn((
        Music,
        AudioPlayer::new(assets.load("sounds/theme.ogg")),
        PlaybackSettings::LOOP.with_volume(Volume::Linear(0.5)),
    ));
}

// Single skips this system until the sink exists
fn toggle_music(keys: Res<ButtonInput<KeyCode>>, music: Single<&AudioSink, With<Music>>) {
    if keys.just_pressed(KeyCode::KeyM) {
        music.toggle_playback();
    }
}
```

### Playback Settings

| Constant | At the end of the sound |
|----------|-------------------------|
| `PlaybackSettings::ONCE` | Stops; entity and components stay |
| `PlaybackSettings::LOOP` | Repeats |
| `PlaybackSettings::DESPAWN` | Despawns the entity. Right for one-shot effects |
| `PlaybackSettings::REMOVE` | Removes the audio components |

Builders: `.with_volume(v)`, `.with_speed(s)`, `.with_spatial(bool)`,
`.with_spatial_scale(s)`, `.with_start_position(d)`, `.with_duration(d)`, `.paused()`,
`.muted()`.

### Sink Methods (`AudioSinkPlayback`, in the prelude)

`play`, `pause`, `toggle_playback`, `is_paused`, `stop`, `speed`, `set_speed`, `volume`,
`position`, `try_seek`, `empty`, `is_muted`. These need `&mut`: `set_volume`, `mute`,
`unmute`, `toggle_mute`, so query `&mut AudioSink`.

```rust
// fragment
let current = sink.volume();
sink.set_volume(current.increase_by_percentage(10.0));
```

`Volume` also has `decrease_by_percentage`, `scale_to_factor`, `fade_towards`,
`to_linear`, `to_decibels`.

### Spatial

Mark the emitter spatial and spawn one `SpatialListener`. Panning follows the
`Transform` of each.

```rust
// fragment
commands.spawn((
    AudioPlayer::new(asset_server.load("sounds/engine.ogg")),
    PlaybackSettings::LOOP.with_spatial(true),
    Transform::from_xyz(0.0, 0.0, 0.0),
));
commands.spawn((Transform::default(), Visibility::default(), SpatialListener::new(4.0)));
```

`SpatialListener::new(gap)` sets the distance between the ears. 2D games scale with
`SpatialScale::new_2d(scale)`. Spatial entities use `SpatialAudioSink`, not `AudioSink`.

### Global Volume

`GlobalVolume { volume }` is a resource: `app.insert_resource(GlobalVolume::new(Volume::Linear(0.5)))`.
Changing it does not affect audio already playing, and neither does changing
`PlaybackSettings`. Control playing audio through the sink.

## Common Mistakes

- Using `AudioSink` on a spatial emitter. Query `SpatialAudioSink`
- `Volume` is not in the prelude: `use bevy::audio::Volume;`
- Loading `.mp3`, `.wav` or `.flac` with only the default `vorbis` feature
- Disabling default features and losing `audio`
- Effects spawned with the default `ONCE` leave entities behind. Use `DESPAWN`
- Treating `GlobalVolume` as a bus system. There is one, and no groups
- Replaying a finished `AudioPlayer` in place. Remove the audio components and add them again

## Examples and Sources

`examples/audio/`: `audio`, `audio_control`, `decodable`, `pitch`, `play_sound_effect`,
`soundtrack`, `spatial_audio_2d`, `spatial_audio_3d`.
https://bevy.org/learn/migration-guides/0-18-to-0-19/
