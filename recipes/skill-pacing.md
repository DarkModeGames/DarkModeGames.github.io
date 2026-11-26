# Pace a skill: lock time and animation speed

**Problem.** A sword swing locks the player for 0.8 seconds, but its animation takes 3 seconds and gets cut off
halfway, or keeps playing after the player can move again.

**Pattern.** Two numbers decide how a skill feels:

- the start action's `time` is how long the caster is locked (and the skill's own global cooldown);
- the [`Animation`](/reference/entities.html#animation)'s `rate` sets its speed. Frames play at 5 per second ×
  `rate`.

Make the clip last as long as the lock: `rate = frames ÷ (5 × time)`. A 12-frame swing with `time: 0.8` wants
`rate: 3.0` (12 ÷ 4).

```yaml
Animation:
  heroSwingAnimation:
    N: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    NE: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    E: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    SE: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    S: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    SW: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    W: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    NW: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
    loop: false
    rate: 3.0
  heroIdleAnimation:
    N: [0]
    NE: [0]
    E: [0]
    SE: [0]
    S: [0]
    SW: [0]
    W: [0]
    NW: [0]

AnimationSet:
  heroAnimationSet:
    animations: [heroIdleAnimation, heroSwingAnimation]
    idle: heroIdleAnimation

Inset:
  heroMarginInset:
    top: 4
    right: 8
    bottom: 2
    left: 8

Vector:
  heroFrameVector:
    x: 32
    y: 32

Sprite:
  heroSprite:
    animation_set: heroAnimationSet
    texture: assets/actor.png
    size: heroFrameVector
    margin: heroMarginInset

Action:
  swingAction:
    name: Swing
    time: 0.8
    animation: heroSwingAnimation
    do: wait

Skill:
  swingSkill:
    name: Swing
    description: A quick cut.
    start: swingAction

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    skills: [swingSkill]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Animation`](/reference/entities.html#animation), [`AnimationSet`](/reference/entities.html#animationset), [`Inset`](/reference/entities.html#inset), [`Skill`](/reference/entities.html#skill), [`Sprite`](/reference/entities.html#sprite), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- `Action.animation` must name a clip that is in **this actor's** [AnimationSet](/reference/entities.html#animationset). Otherwise it falls back to idle.
- `time` is a float. Leaving it out (or 0) locks for 1 second.
- A press refused by its first `if` (before any `do`) ends the lock and the animation at once, so "Not ready"
  costs no time.
- `loop: false` plays once and holds the last frame; use it for attacks and deaths.
- For a monster or a cosmetic flourish with no lock, use `play_keyframe_self` with the clip: it plays one cycle and
  returns to idle.
- Pace a monster's attacks with a cooldown resource, not `time`: a strategy action with `time` roots the monster.
