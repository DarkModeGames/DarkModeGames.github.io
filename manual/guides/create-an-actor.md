# Create an Actor

This page shows you how to author an [Actor](/manual/getting-started/glossary.html) entity in campaign YAML, building from a minimal definition up to a full player and NPC actor. An actor is a character in the game world, so getting its fields right is the first step toward a playable campaign.

## Introduction

An Actor entity describes one character: how it looks, how fast it moves, what it can see, what it can do, and which resources it tracks. The same entity type covers both the player and AI-driven enemies; the difference comes from the fields you set.

This page uses two actors from an example campaign. The `heroActor` is the canonical player. The `guardActor` is the canonical NPC. By the end you will understand every Actor field and when to set it.

> Note: Every Actor field is optional. You can spawn a valid actor with no fields at all, then add only what your character needs.

## Prerequisites

Read these pages first so the concepts and YAML shape here are familiar:

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) — how entities reference each other.
- [Your first campaign](/manual/getting-started/your-first-campaign.html) — how a campaign loads.
- [Campaign structure](/manual/core-concepts/campaign-structure.html) — where entity files live.

## A minimal actor

An entity file is YAML keyed by entity type, then by [entity key](/manual/getting-started/glossary.html). The smallest useful actor names itself and points at a sprite.

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
```

The `name` is the human-readable label shown in the UI. The `sprite` is a [KeyRef](/manual/getting-started/glossary.html) to a [Sprite](/manual/guides/visuals-sprites-animations.html) entity that supplies the visuals. Both the hero and the guard reuse `heroSprite`, which shows that assets are shared by key rather than copied.

> Note: In YAML you always write the clean key `name`. The internal alias `name_` exists only inside the engine's GDScript; campaign files never use it.

## Footprint and collision

Two fields control how the actor occupies space on the map.

`base` is the size, in pixels, of the actor's footprint circle — the circle used for placement and selection. The example uses `8` for both actors.

The actor's click/touch area is taken automatically from its sprite (shaped to the visible art, sitting at the feet) — there is no field to author.

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    base: 8
```

The `base` value must be `0` or greater.

## Sight: perception and salience

Two fields govern how actors detect one another.

[perception](/manual/getting-started/glossary.html) is the actor's vision range in pixels for detecting other actors. The hero sees farther than the guard: `25` versus `15`.

[salience](/manual/getting-started/glossary.html) is how easily other actors detect this actor — its detection difficulty, where a higher number means more visible. Both example actors use `1`.

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    perception: 25
    salience: 1
```

Both fields must be `0` or greater. Together they decide whether one actor can see another: a target with high `salience` is spotted from farther away, while an observer with high `perception` spots more.

## Movement: speed and bearing

`speed` is movement speed in pixels per second. The hero moves at `1.0`; the slower guard at `0.5`.

`bearing` is the current facing direction in degrees, from `0` to `360`, where `0` is north. It defaults to `0`, and neither example actor sets it.

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    speed: 1.0
```

The `speed` value must be `0.0` or greater.

## Resources and visibility

An actor's numeric values — health, mana, gold — are [Resource](/manual/guides/add-resources-and-measures.html) entities listed in the `resources` field. The hero tracks three; the guard tracks one.

Two more fields control who sees which resources:

- `public` lists resources shown on the target focus [plate](/manual/getting-started/glossary.html) when another actor views this actor.
- `private` lists resources shown only on the actor's own data plate.

The hero exposes only `health` publicly but sees all three privately.

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    resources:
    - health
    - mana
    - gold
    public:
    - health
    private:
    - health
    - mana
    - gold
```

The guard sets `resources` and `public` to `["health"]` and omits `private`. Omit any of these fields when the actor has nothing to put in it.

You can also attach [Measure](/manual/guides/add-resources-and-measures.html) entities through `measures` — values rolled from a [dice expression](/manual/getting-started/glossary.html). The hero carries `["luckMeasure"]`; the guard carries none.

> Tip: Resources and measures appear on plates. Dialog text reads live values with templating tokens — `@` for the viewed target and `$` for the caller. See [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html).

## Skills, group, and menu

`skills` is a list of [Skill](/manual/guides/add-skills.html) entities the actor can use, up to nine. The hero has `["attackSkill", "healSkill"]`; the guard has none.

`group` is a KeyRef to a [Group](/reference/entities.html#group) entity for faction membership and outline color. The hero belongs to `playerGroup`, the guard to `enemyGroup`.

`menu` is a KeyRef to a [Menu](/manual/guides/menus-dialogs-waypoints.html) entity that supplies custom interaction options. The hero uses `heroMenu`, the guard `guardMenu`.

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    skills:
    - attackSkill
    - healSkill
    group: playerGroup
    menu: heroMenu
```

## Triggers and timers

`triggers` lists [Trigger](/manual/guides/triggers-and-timers.html) entities that watch resource changes and run an action when a condition is met. Both example actors carry `["deathTrigger"]`, which fires when health runs out.

`timers` lists [Timer](/manual/guides/triggers-and-timers.html) entities that run an action on an interval or after a delay. The hero has `["regenTimer"]`; the guard has none.

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    triggers:
    - deathTrigger
    timers:
    - regenTimer
```

## Event hooks

An actor can run an [Action](/manual/guides/define-actions.html) in response to four world events. Each field is a KeyRef to an Action:

- `on_touch` — another actor touches this actor.
- `on_view` — another actor views this actor.
- `on_map_entered` — this actor enters a map.
- `on_map_exited` — this actor exits a map.

Neither example actor sets these, so they are shown here only for completeness. Wire them up when a character should react automatically to contact, sight, or map changes.

> Note: An Action's `parameters` field is a flat array of KeyRefs to Parameter entities, paired with a sibling top-level `Parameter` block. See [Define actions](/manual/guides/define-actions.html) for the exact shape.

## Player versus NPC

The single field that turns an actor into an AI-driven NPC is `strategy` — a KeyRef to a [Strategy](/manual/guides/add-npc-ai.html) entity that decides what the actor does. The player actor has no strategy because the human controls it.

Here is the complete `heroActor`, the player. It has no `strategy`:

```yaml
Actor:
  heroActor:
    name: Hero
    speed: 1.0
    sprite: heroSprite
    base: 8
    public:
    - health
    private:
    - health
    - mana
    - gold
    skills:
    - attackSkill
    - healSkill
    perception: 25
    salience: 1
    menu: heroMenu
    resources:
    - health
    - mana
    - gold
    measures:
    - luckMeasure
    triggers:
    - deathTrigger
    timers:
    - regenTimer
    group: playerGroup
Vector:
  heroStartPos:
    x: 200
    y: 100
```

Note that this file holds a second entity, the `heroStartPos` Vector. A single entity file can define multiple entity types and keys.

And here is the complete `guardActor`, an NPC. It is leaner — no skills, measures, timers, or private resources — and it sets `strategy`:

```yaml
Actor:
  guardActor:
    name: Guard
    speed: 0.5
    sprite: heroSprite
    base: 8
    public:
    - health
    perception: 15
    salience: 1
    menu: guardMenu
    resources:
    - health
    triggers:
    - deathTrigger
    group: enemyGroup
    strategy: guardStrategy
```

The guard sees a shorter distance, moves at half the hero's speed, and is driven entirely by `guardStrategy`.

## See also

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Define actions](/manual/guides/define-actions.html)
- [Add skills](/manual/guides/add-skills.html)
- [Add NPC AI](/manual/guides/add-npc-ai.html)
- [Triggers and timers](/manual/guides/triggers-and-timers.html)
- [Visuals: sprites and animations](/manual/guides/visuals-sprites-animations.html)
- [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html)
- [Entity reference](/reference/entities.html)
