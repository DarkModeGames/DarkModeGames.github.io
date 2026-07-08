# Audio and Backgrounds

This page shows you how to give a map atmosphere: a soundtrack with the Sound entity and a scrolling backdrop with the Parallax entity. You attach both to a map, so the music plays and the background drifts as soon as the player loads it.

## Prerequisites

You should be comfortable with these pages before you start:

- [Build a map](/manual/guides/build-a-map.html) — you attach audio and backgrounds to a Map entity.
- [Visuals: sprites and animations](/manual/guides/visuals-sprites-animations.html) — Sound can also be triggered from an animation.
- [Dice expressions](/manual/core-concepts/dice-expressions.html) — a Sound's pitch is set by a [dice expression](/manual/getting-started/glossary.html#terms).

This page builds on an example campaign's sound, parallax, and map entities.

## How sound and background attach to a map

A map carries its presentation through two list fields. Both are [KeyRefArrays](/manual/getting-started/glossary.html#terms) — arrays of [entity keys](/manual/getting-started/glossary.html#terms) that point at other entities.

- `Map.audio` references Sound entities. The engine plays each one when the map loads.
- `Map.background` references Parallax entities. Each one becomes a scrolling layer behind the world.

The example map wires up one sound and two background layers:

```yaml
Map:
  demoMap:
    name: Demo Map
    background:
    - skyParallax
    - starsParallax
    audio:
    - bgMusic
```

The map names the keys; the next sections define the entities behind those keys.

## Add sound with the Sound entity

A Sound entity is an audio file plus playback settings. Use it for background music, ambience, or a one-shot effect.

The Sound entity has three fields:

- `source` — the path to the audio file inside your campaign archive.
- `scale` — a [dice expression](/manual/getting-started/glossary.html#terms) that sets the playback pitch.
- `loop` — whether the stream repeats continuously.

> Note: The schema marks every Sound field optional, but a Sound with no `source` has nothing to play. Always set `source`.

### Define background music

Here is a looping soundtrack:

```yaml
Sound:
  bgMusic:
    source: /assets/Audio/Crystal caverns Chimes 1.mp3
    scale: 1d200
    loop: true
```

The `source` path is relative to the root of the campaign archive. The engine picks the audio format from the file extension, so an `.mp3`, `.ogg`, or `.wav` all work. With `loop` set to `true`, the track restarts the moment it ends, which is what you want for background music.

To attach it, list `bgMusic` in the map's `audio` array, as shown above.

### Control pitch with scale

The `scale` field is a [dice expression](/manual/getting-started/glossary.html#terms). The engine rolls it, divides the result by `100.0`, and applies that number as the audio player's pitch scale.

> Warning: `scale` controls **pitch**, not loudness. A value of `1d200` rolls a number from 1 to 200, giving a pitch between `0.01` and `2.0` — anywhere from very low and slow to twice normal speed. Loudness is handled separately by the engine's fade, which you do not set on the entity.

The expression is re-rolled every time playback finishes, so a random `scale` shifts the pitch on each repeat. Any valid dice notation works, including a plain constant for a fixed pitch. To keep the pitch unchanged, set `scale` to `100`.

### Trigger a sound from an animation

A Sound does not have to live on a map. The `attackSound` is referenced from an animation instead, so it fires when that animation plays:

```yaml
Sound:
  attackSound:
    source: /assets/Audio/fx.mp3
    scale: 50+(1d100)
```

This sound omits `loop`, so it plays once. Its `scale` of `50+(1d100)` rolls 51 to 150, giving a pitch of `0.51` to `1.5` — a small random variation that keeps a repeated effect from sounding identical each time. See [Visuals: sprites and animations](/manual/guides/visuals-sprites-animations.html) for how an animation points at a sound.

## Add a background with the Parallax entity

A Parallax entity is one scrolling background layer. [Parallax](https://en.wikipedia.org/wiki/Parallax) is the effect where distant objects appear to move slower than near ones as the camera pans — stacking layers at different speeds gives the world depth.

The Parallax entity has two fields:

- `texture` — the path to the background image inside your campaign archive.
- `effect` — how fast the layer scrolls relative to the camera.

> Note: Both fields are optional in the schema, but a layer with no `texture` has nothing to draw. Always set `texture`.

### Define a layer

Here are two layers:

```yaml
Parallax:
  skyParallax:
    texture: /assets/SpaceBg/Backgrounds/Blue1.png
    effect: 10.0
  starsParallax:
    texture: /assets/SpaceBg/Backgrounds/BlueStars.png
    effect: 20.0
```

As with audio, the `texture` path is relative to the root of the campaign archive. List both keys in the map's `background` array to stack them.

### Set the scroll speed with effect

The `effect` field is a scroll-speed factor. The engine divides it by `100` to get the layer's motion ratio against the camera:

- `effect` of `10.0` gives a ratio of `0.1` — the layer moves at 10% of camera speed.
- `effect` of `20.0` gives a ratio of `0.2` — the layer moves at 20% of camera speed.

A lower `effect` makes a layer move slower, which reads as farther away. The distant sky (`10.0`) drifts behind the nearer stars (`20.0`), creating depth. The minimum is `0.0`, a layer that does not scroll at all.

> Warning: `effect` is a float. Write it with a decimal point, such as `10.0`, not `10`. The whole-number form `10` fails validation.

## See also

- [Build a map](/manual/guides/build-a-map.html)
- [Visuals: sprites and animations](/manual/guides/visuals-sprites-animations.html)
- [Dice expressions](/manual/core-concepts/dice-expressions.html)
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Entity reference](/reference/entities.html)
