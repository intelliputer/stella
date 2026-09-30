# Intelliputer Stella fork notes

This checkout is a local Stella fork used by the sibling `mi` Atari 2600
assembly project. Its purpose is deterministic, non-interactive emulation for
agent and regression experiments. It is not an upstream Stella feature set.

## Local additions

The following command-line options were added in `src/common/main.cxx` and
`src/emucore/OSystem.*`:

| Option | Purpose |
| --- | --- |
| `-inputscript FILE` | Apply a JSON array of frame-indexed Stella input events. |
| `-snapshotframes N` | Save a snapshot and quit after `N` emulation frames. |
| `-assertmemory ADDRESS=VALUE` | At final snapshot time, require a hexadecimal emulated-memory byte and exit non-zero on mismatch. |
| `-telemetry FILE` | Write one JSON Lines state record per emulation frame. |
| `-keyframeinterval N` | Save numbered PNG keyframes every `N` emulation frames. |

The input-script format is intentionally simple:

```json
[
  { "frame": 30, "event": "ConsoleReset", "value": 1 },
  { "frame": 31, "event": "ConsoleReset", "value": 0 },
  { "frame": 37, "event": "LeftJoystickLeft", "value": 1 }
]
```

`event` must be an existing Stella `Event::Type` name. Inputs are delivered to
the normal event handler before the matching emulation frame is dispatched.

Each telemetry line is a JSON object such as:

```json
{"frame":190,"room":17,"player_x":74,"player_y":47,"carried_object":162}
```

The fields are deliberately generic RAM observations for the Adventure ROM
experiment. Address semantics belong to that ROM: `$8A` is the room, `$8B/$8C`
are the player coordinates, and `$9D` is the carried object. If this fork is
used for another ROM, either interpret these raw fields differently or extend
the feature with a configurable watch list.

## Experiment bundle convention

The sibling project stores a run as a directory containing the source action
trace, a ROM/build manifest, `telemetry.jsonl`, selected keyframe PNGs, and
the terminal result. The action trace is the reproducible source of truth;
telemetry provides searchable state history; images provide later human-facing
visualization.

## Build and use

On this machine:

```sh
./configure
make -j32
```

The build needs the SDL 3 development package. Scenario runs can use a
temporary base directory and dummy SDL audio, for example:

```sh
SDL_AUDIODRIVER=dummy ./stella -basedir /tmp/stella-run \
  -audio.enabled 0 -inputscript actions.json \
  -telemetry telemetry.jsonl -keyframeinterval 60 \
  -snapshotframes 600 -assertmemory 9d=bf game.bin
```

The SDL video path still needs access to a graphical session. Dummy audio makes
agent runs independent of PipeWire/PulseAudio output.

## Upstreaming and maintenance

Keep these changes in clearly scoped commits. They are useful locally, but
they currently make Adventure-specific telemetry assumptions and should not be
upstreamed unchanged. A possible upstream-quality follow-up would replace the
fixed addresses with configurable memory watches, provide a documented result
schema, and add automated tests for input timing and output files.
