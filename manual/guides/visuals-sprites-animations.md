# Sprites and Animations

This page shows how an actor gets its on-screen appearance from three entities: a [Sprite](/reference/entities.html#sprite) that names a sprite sheet and frame size, an [AnimationSet](/reference/entities.html#animationset) that groups animations, and [Animation](/reference/entities.html#animation) entities that list the frames to play for each of the eight directions an actor can face.

## Introduction

An actor in the game world is drawn from a single image file, a *sprite sheet*, that packs many small pictures into one grid. A Sprite entity tells the engine which sheet to use and how big one frame is. An Animation entity then lists, per facing direction, which frames to show in sequence so the actor appears to walk, run, or swing a tool.

You connect these pieces by key. The actor references a Sprite, the Sprite references an AnimationSet, and the AnimationSet references its Animations. At runtime the engine combines the current animation state (such as `idle` or `run`) with the actor's heading (such as `S` for south) to pick the exact frames to play. Getting this wiring right is what makes a static image come alive in eight directions.

This page uses an example hero throughout: `heroSprite`, `heroAnimSet`, and the `idle`, `run`, and `tool` animations.

## Prerequisites

Read these pages first:

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) — how entities reference each other by key.
- [Create an actor](/manual/guides/create-an-actor.html) — the actor's `sprite` field points at the Sprite you build here.

If a term is unfamiliar, see the [Glossary](/manual/getting-started/glossary.html).

## How a sprite sheet becomes frames

A sprite sheet is one image divided into a grid of equal cells. Each cell is a *frame*, and every frame has an index. Indices count left to right, then top to bottom, starting at `0`.

The engine slices the image into frames by the frame size you give it, then numbers them. So if a sheet is `512` pixels wide and each frame is `64` pixels wide, there are `8` columns across. Frame `0` is the top-left cell, frame `7` is the top-right cell, and frame `8` is the first cell of the second row.

> Note: You never write pixel coordinates in an Animation. You write frame indices, and the engine converts each index into a rectangle for you.

## Define the Sprite

A Sprite entity names the sheet, the frame size, the rendering margin, and the AnimationSet to build from. The hero's sprite:

```yaml
Sprite:
  heroSprite:
    animation_set: heroAnimSet
    texture: /assets/astronaut_sprite.png
    size: spriteSize
    margin: spriteMargin
Vector:
  spriteSize:
    x: 64
    y: 64
Inset:
  spriteMargin:
    top: 16
    right: 8
    bottom: 16
    left: 8
```

The fields:

- `texture` is the path to the sheet image inside the campaign archive.
- `size` is a [KeyRef](/manual/getting-started/glossary.html) to a [Vector](/reference/entities.html#vector) giving one frame's width and height in pixels. Here each frame is `64` by `64`. This is the number the engine uses to slice the grid.
- `margin` is a KeyRef to an [Inset](/reference/entities.html#inset) giving the empty padding around the visible art on each edge (`top`/`right`/`bottom`/`left`). The bottom edge seats the art's feet on the tile, the left/right center it, and all four edges size the actor's footprint so it hugs the body instead of the empty frame.
- `animation_set` is a KeyRef to the AnimationSet whose animations are built into this sprite. Without it, the sprite has no directional animations.

All four fields are optional in the schema, but a sprite with no `texture`, `size`, or `animation_set` produces nothing visible. Always set them.

> Tip: Define `spriteSize` (a Vector) and `spriteMargin` (an Inset) in the same file as the sprite. It keeps the frame geometry next to the entity that uses it.

The `size` Vector does double duty. The engine uses it both to slice the sheet into frames and as the full frame size. The `margin` Inset's edges are subtracted from that full size so a frame with empty padding sits correctly on its tile and the footprint matches the art.

## Group animations with an AnimationSet

An AnimationSet is a named collection of Animations:

```yaml
AnimationSet:
  heroAnimSet:
    name: Hero Animations
    animations:
    - idle
    - run
    - tool
```

The fields:

- `name` is a human-readable label for the set.
- `animations` is an array of KeyRefs to Animation entities.

Each Animation's *key* matters: the key becomes the animation state name at runtime. The set lists `idle`, `run`, and `tool`, so those three keys are the states this actor can play.

> Warning: In YAML, write the clean key `name`. The internal GDScript backing field is `name_`, but that alias is internal only and must never appear in a campaign file.

## Define an Animation

An Animation holds eight arrays, one per direction, plus an optional sound. Each array lists the frame indices to play in order for that facing.

The eight directions are `N`, `NE`, `E`, `SE`, `S`, `SW`, `W`, and `NW` — the eight compass headings an isometric actor can face.

### A single-frame animation: idle

The hero's `idle` animation shows one frame per direction. The actor stands still, so each array holds a single index:

```yaml
Animation:
  idle:
    N:
    - 35
    NE:
    - 20
    E:
    - 5
    SE:
    - 25
    S:
    - 30
    SW:
    - 15
    W:
    - 0
    NW:
    - 10
```

Each integer is one frame index into the sheet. When the hero faces south while idle, the engine shows frame `30`.

### A looping animation: run

The `run` animation cycles four frames per direction to animate the legs:

```yaml
Animation:
  run:
    N:
    - 37
    - 38
    - 39
    - 38
    NE:
    - 32
    - 33
    - 34
    - 33
    E:
    - 7
    - 8
    - 9
    - 8
    SE:
    - 27
    - 28
    - 29
    - 28
    S:
    - 22
    - 23
    - 24
    - 23
    SW:
    - 17
    - 18
    - 19
    - 18
    W:
    - 12
    - 13
    - 14
    - 13
    NW:
    - 2
    - 3
    - 4
    - 3
```

Facing south, the engine plays frames `22`, `23`, `24`, `23` in sequence, producing a four-step running cycle. Repeating the middle frame (`23`) gives a back-and-forth leg motion.

### An animation with sound: tool

The `tool` animation is a single-frame action pose with an attached sound:

```yaml
Animation:
  tool:
    sound: attackSound
    N:
    - 36
    NE:
    - 31
    E:
    - 6
    SE:
    - 26
    S:
    - 21
    SW:
    - 16
    W:
    - 11
    NW:
    - 1
```

The `sound` field is a KeyRef to a [Sound](/manual/guides/audio-and-backgrounds.html) entity. The engine builds an audio node named after the animation key and plays it when the animation starts. When the animation's state ends, a looping sound fades out and a one-shot plays to its end. See [Audio and backgrounds](/manual/guides/audio-and-backgrounds.html) for how Sound entities are defined.

An `Animation` also accepts an optional `loop` boolean, which defaults to `true`. Leave it out and the animation repeats, as `idle`, `walk` and `run` should. Set `loop: false` for a clip that plays once and holds its last frame, such as a death pose. A skill whose action uses a one-shot clip holds that last frame until the action's `time` runs out, then the actor returns to idle.

## How states and headings choose what plays

At runtime the engine names each playable animation by combining the state with the heading, in the form `<state>:<heading>`. For example, `run:S` is the running animation facing south, and `idle:N` is the idle pose facing north. The engine builds one of these for every animation-and-direction pair in the set.

Two inputs drive the choice:

1. **The state.** Movement uses the built-in states `idle`, `walk`, `run`, `fly`, and `hover`. On the ground, the engine selects `run` if the actor's speed is high enough, otherwise `walk`; when it stops, it returns to `idle`. While flying (toggled with the `fly_self`/`land_self` actions), the same movement/stopped split uses `fly` while moving and `hover` while stationary instead. A flier only lands over walkable ground: `land_self` and `land_target` do nothing over a wall, water or the edge of the map, and `$can_land` reads `1` when landing would work. Check `$can_land` before a `change_actor` into a body that can't fly, which isn't stopped this way. A skill can also set a state from its action's animation (the example attack uses `tool`).
2. **The heading.** The actor's facing maps to one of the eight directions.

Because of this, an AnimationSet should normally include `idle` plus `walk` and/or `run` so ground movement has frames to play, and `fly`/`hover` if the actor can ever fly. The example provides `idle` and `run`; a flying actor would add `fly` and `hover` the same way.

> Note: If a requested `<state>:<heading>` animation does not exist, the actor falls back to `idle:<heading>` for that direction. A missing run animation simply shows the idle pose while moving, so define the states you actually use.

### Driving an animation from a skill

A skill's start action can name an animation. When the skill runs, the engine sets that animation as the state for the action's duration, then returns the actor to `idle` when the action ends. The example attack skill uses the `tool` animation this way, which is why `tool` is in `heroAnimSet`. See [Add skills](/manual/guides/add-skills.html) and [Define actions](/manual/guides/define-actions.html) for how an action references an animation.

## Wire the sprite to an actor

The actor connects to everything through its `sprite` field. The hero sets:

```yaml
Actor:
  heroActor:
    sprite: heroSprite
```

That single KeyRef pulls in the whole chain: `heroSprite` names the sheet and frame size and points at `heroAnimSet`, which lists `idle`, `run`, and `tool`. At load time the engine slices the sheet into frames and builds a directional animation for each state-and-heading pair. See [Create an actor](/manual/guides/create-an-actor.html) for the rest of the actor's fields.

## Reusing one sprite with skins

A Skin recolors an actor's sprite at render time, so many [Actor](/reference/entities.html#actor) entities can share one Sprite (and one texture) while each showing up as a different color — no duplicate art or duplicate Sprite entities needed.

A Skin is a named list of Masks, and each Mask maps one hex color to another:

```yaml
Mask:
  redToBlue:
    from_color: "#B23A2E"
    to_color: "#2E5DB2"
  redToYellow:
    from_color: "#B23A2E"
    to_color: "#D9C23A"
Skin:
  blueTeam:
    title: Blue Team
    masks:
    - redToBlue
  yellowTeam:
    title: Yellow Team
    masks:
    - redToYellow
```

Point an actor's `skin` field at one:

```yaml
Actor:
  heroBlue:
    sprite: heroSprite
    skin: blueTeam
  heroYellow:
    sprite: heroSprite
    skin: yellowTeam
```

Both actors above use the exact same `heroSprite` texture. `heroBlue` and `heroYellow` each swap the sprite's red for a different color, entirely on the GPU — the underlying image is never duplicated or re-exported. `skin` is optional: an actor with no `skin` renders the sprite's original, unmodified colors.

`skin` is fixed campaign data — set it when you define the Actor, not something changed mid-game. A Skin holds up to 16 Masks; colors are matched exactly (small anti-aliasing differences are tolerated automatically).

## Checklist for a new actor sprite

1. Add the sheet image to your campaign's `assets` folder.
2. Define a Vector for the frame `size` and an Inset for `margin`.
3. Define a Sprite with `texture`, `size`, `margin`, and `animation_set`.
4. Define an AnimationSet listing your Animation keys.
5. Define `idle` plus `walk` and/or `run` Animations, with all eight directions filled in.
6. Point the actor's `sprite` field at your Sprite.
7. Optional: to reuse the same Sprite with a different coloring, define a Skin (and its Masks) and point the actor's `skin` field at it.

## See also

- [Create an actor](/manual/guides/create-an-actor.html)
- [Audio and backgrounds](/manual/guides/audio-and-backgrounds.html)
- [Define actions](/manual/guides/define-actions.html)
- [Add skills](/manual/guides/add-skills.html)
- [Entity reference](/reference/entities.html)
