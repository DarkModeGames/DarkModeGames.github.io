# Screen Effects and Particles

This page covers two purely visual entities: the [`Overlay`](/reference/entities.html#overlay), a full-screen
animated effect such as rain or fog, and the [`Particle`](/reference/entities.html#particle), a small animated
sprite that rides on one actor, such as a glow or a trail of smoke. Neither changes any game state.

## Prerequisites

- [Sprites and animations](/manual/guides/visuals-sprites-animations.html), since both entities cut frames from a
  sprite sheet the same way.
- [Build a map](/manual/guides/build-a-map.html), since a map can list overlays to show on arrival.

## Overlays: weather on the screen

An Overlay is drawn between the game world and the interface, fixed to the camera. Its fields:

- `texture` — the sprite sheet; `size` — a [`Vector`](/reference/entities.html#vector) giving one frame's size;
  `frames` — the frame indices to play, in order.
- `rate` — playback speed over the base rate (1.0 unchanged); `loop` — repeat, or hold the last frame.
- `opacity` — 0 to 1; `tile` — `true` repeats the texture across the screen (rain, snow), `false` stretches it.

The simplest way to show one is the map's `overlay` list: it is shown on each player's screen when they arrive on
that map, and cleared when they leave.

```yaml
Vector:
  rainFrameVector:
    x: 32
    y: 32
  moorSpawnVector:
    x: 384
    y: 8

Overlay:
  rainOverlay:
    texture: assets/actor.png
    size: rainFrameVector
    frames: [0]
    loop: true
    rate: 1.0
    opacity: 0.6
    tile: true

Map:
  moorMap:
    name: The Moor
    tilemap: groundTilemap
    spawn: moorSpawnVector
    overlay: [rainOverlay]

Parameter:
  rainOverlayParameter:
    key: overlay
    value: rainOverlay
  thinRainParameter:
    key: opacity
    value: '0.3'

Action:
  lightRainAction:
    do: set_overlay_opacity
    parameters: [rainOverlayParameter, thinRainParameter]
  stopRainAction:
    do: hide_overlay
    parameters: [rainOverlayParameter]
```

The actions `show_overlay`, `hide_overlay`, `set_overlay_opacity` and `clear_overlays` change overlays while
playing.

> Note: actions run on the host, so the overlay actions change the host's own screen. In a multiplayer game,
> prefer the map's `overlay` list, which every player's screen applies on arrival.

## Particles: effects that follow an actor

A Particle uses the same sheet fields (`texture`, `size`, `frames`, `rate`, `loop`, `opacity`) plus movement:

- `speed` — drift in pixels per second away from the actor (0 stays on it);
- `direction` — drift bearing in degrees relative to the way the actor faces (0 forward, 90 clockwise);
- `lock` — `true` fixes the bearing when attached; `false` turns with the actor.

Attach one with `attach_particle_self` or `attach_particle_target`. The optional `duration` (seconds) removes it
automatically; without it, it stays until `remove_particle_self` / `remove_particle_target`. Attached particles are
shown to every player.

```yaml
Vector:
  emberFrameVector:
    x: 32
    y: 32

Particle:
  emberGlowParticle:
    texture: assets/actor.png
    size: emberFrameVector
    frames: [0]
    loop: true
    rate: 1.0
    opacity: 0.8
    speed: 0.0

Parameter:
  emberGlowParameter:
    key: particle
    value: emberGlowParticle
  emberSecondsParameter:
    key: duration
    value: '5'

Action:
  kindleAction:
    name: Kindle
    time: 1.0
    do: attach_particle_self
    parameters: [emberGlowParameter, emberSecondsParameter]

Skill:
  kindleSkill:
    name: Kindle
    description: Wreathe yourself in embers for 5 seconds.
    start: kindleAction
```

## See also

- [Audio and backgrounds](/manual/guides/audio-and-backgrounds.html) — [`Sound`](/reference/entities.html#sound)
  and [`Parallax`](/reference/entities.html#parallax).
- [Parallax backgrounds and floor decals](/recipes/backgrounds-and-floors.html).
- [Action reference](/reference/actions.html) — every overlay and particle action.
