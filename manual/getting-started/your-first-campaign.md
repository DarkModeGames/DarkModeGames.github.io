# Your First Campaign

This page walks you through building the smallest campaign that loads: one [Main](/manual/getting-started/glossary.html#terms) entity, one [Actor](/manual/getting-started/glossary.html#terms), and one [Map](/manual/getting-started/glossary.html#terms). You package it, validate it, and boot it headless, using an example campaign as the worked reference.

## Introduction

A [campaign](/manual/getting-started/glossary.html#terms) is a complete game definition written as data. You describe characters, maps, and rules in YAML files, package them as a `.zip`, and the Isometry binary loads them. No code changes are needed to make a game.

By the end of this page you will understand the shape every entity file shares, know which entities a campaign cannot do without, and have run the full build-validate-boot loop. An example campaign is referenced throughout so you can compare your work against a known-good campaign.

## Prerequisites

Before you start, make sure you have:

- Read [What is Isometry](/manual/getting-started/introduction.html) for the big picture.
- Followed [Install and run](/manual/getting-started/install-and-run.html) so the engine binary is downloaded and runnable.

## The entity file shape

Every campaign is a folder of YAML files, and every file follows the same nesting: a **type** name, then one or more **[entity keys](/manual/getting-started/glossary.html#terms)**, then the **fields** for each entity. An [entity](/manual/getting-started/glossary.html#terms) is a pure-data record; there are exactly {{ entity_count }} entity types, and they hold data only, no game logic.

A minimal `Main.yaml` shows the shape at its smallest:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
```

Read this as: a [`Main`](/reference/entities.html#main) entity, keyed `demoMain`, with two fields. The values `"heroActor"` and `"demoMap"` are not actors and maps themselves; they are **[KeyRefs](/manual/getting-started/glossary.html#terms)**, that is, the keys of other entities the engine resolves at load time. Entity keys must be unique across the whole campaign, so a key used for an [Actor](/reference/entities.html#actor) cannot be reused for a Map.

> Note: You may put several entities, even of different types, in one file. The loader merges every YAML file in the campaign into a single set of entities. An example campaign typically splits entities across files for clarity (`Main.yaml`, `heroActor.yaml`, `map.yaml`, and so on) but co-locates helpers, for example a `map.yaml` that also defines the [`Vector`](/reference/entities.html#vector) that the map's spawn point points to.

## Main: the one required entity

Every campaign must contain **exactly one** Main entity. It defines the campaign's starting state, but only one of its fields is required:

| Field | Type | Points to |
| --- | --- | --- |
| `map` | KeyRef | the starting Map (required) |
| `actor` | KeyRef | the player-controlled Actor (optional — omit it and the player starts as an invisible, immovable [null actor](/manual/core-concepts/campaign-structure.html#the-main-entity)) |

If Main is missing or duplicated, validation stops with an error such as `Campaign requires exactly one Main entity (found 0)`.

Main also accepts two more optional fields:

- `notes` — a free-text campaign description, empty by default.
- `plugins` — an ordered list of nested campaign zips to merge in; earlier entries win, and the host campaign always takes priority.

This walkthrough builds a campaign with a controllable Actor, so `Main.map` and `Main.actor` are both the values you need — your next two files just need to exist under the keys Main names.

## Actor: a character in the world

An [Actor](/manual/getting-started/glossary.html#terms) is a character, player-controlled or AI-driven. Every Actor field is optional, so a bare `{}` passes schema validation. In practice you give the player actor a name, a sprite, and a few stats.

Here are the core fields for a hero actor, trimmed to the essentials:

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    base: 8
    speed: 1.0
    perception: 250
    salience: 10
```

Each field is doing one job:

- `name` is the display name. Always write the clean key `name`; the underscore form `name_` is internal and never used in campaign files.
- `sprite` is a KeyRef to a [Sprite](/reference/entities.html#sprite) entity.
- `base` is the actor's footprint size in pixels (minimum 0).
- `speed` is movement speed (minimum 0.0).
- `perception` is the vision range in pixels (minimum 0).
- `salience` is how far away, in pixels, others can notice this actor (minimum 0).

> Warning: `speed` is a Float field, so it must carry a decimal point. Write `1.0`, not `1`. Integer fields like `base` and `perception` accept whole-number floats such as `8.0`, but a Float field rejects a bare integer.

A full hero also references skills, resources, measures, a menu, triggers, and timers through [KeyRefArrays](/manual/getting-started/glossary.html#terms). You can add those later; see [Create an actor](/manual/guides/create-an-actor.html) for the complete field set.

## Map: where play happens

A [Map](/manual/getting-started/glossary.html#terms) is a place actors occupy. Like Actor, all its fields are optional. A typical `map.yaml` references a tilemap, a spawn point, and more:

```yaml
Map:
  demoMap:
    name: Demo Map
    tilemap: demoTileMap
    spawn: demoSpawn
Vector:
  demoSpawn:
    x: 128
    y: 16
```

Note that `spawn` is a KeyRef to a `Vector` entity, not a raw coordinate. The Vector, `demoSpawn`, is defined in the same file. This is the co-location pattern from earlier: the helper entity lives beside the entity that uses it.

For the smallest possible campaign, even `tilemap` and `spawn` may be omitted; a bare Map satisfies schema validation, and cross-reference validation only checks that any keys you *do* reference exist. To build a real, walkable map, see [Build a map](/manual/guides/build-a-map.html).

## Package the campaign

Isometry loads a campaign from a `.zip` whose single root folder matches the campaign name. All asset paths inside the YAML are relative to that root.

Package your campaign folder with a standard zip command, run from the directory that contains it:

```bash
zip -r mycampaign.zip mycampaign/ -x "*.DS_Store"
```

This writes `mycampaign.zip` with the root folder inside the archive named `mycampaign`, and excludes `.DS_Store` files. Keep that root folder name consistent with the campaign name you load.

## Validate and boot it

When a campaign loads, the validator runs in four phases:

1. **Schema** — each entity has valid fields and types.
2. **Cross-reference** — every KeyRef points to a real key, exactly one Main exists, and no keys are duplicated.
3. **Asset paths** — referenced files exist (only when a ZIP archive is supplied).
4. **Dice expressions** — any [dice notation](/manual/core-concepts/dice-expressions.html) parses.

On success the engine logs `Campaign validation passed`.

The fastest check is a quick validate, which runs the validator and exits `0` on success or `1` on failure:

```bash
isometry --validate=mycampaign.zip
```

To run the full load pipeline headless, put the zip in the `campaigns` folder next to the engine (see [Where files go](/manual/getting-started/install-and-run.html#where-files-go)) and boot with `--network=host`:

```bash
isometry --campaign=mycampaign --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

Logs write to `log.txt` in the engine's data folder. A successful boot shows `Campaign validation passed` followed by `Successfully created Entity` lines.

> Note: `--network=none` currently loads no campaign; it is a no-op. Use `--network=host` to exercise the load pipeline. See [CLI reference](/reference/cli.html) for every flag.

If validation fails, the log names the entity and field at fault. Fix the YAML, **zip the campaign again**, and rerun. The engine only ever reads the zip, so an edit to a loose file does nothing until you repackage.

> Tip: Make a one-line script that zips, validates, and boots. You will run it hundreds of times.

## See also

- [Glossary](/manual/getting-started/glossary.html) — definitions of every term used here.
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) — how typed references resolve.
- [Campaign structure](/manual/core-concepts/campaign-structure.html) — file layout and the merge model.
- [Validation](/manual/core-concepts/validation.html) — the four phases and their error messages.
- [Create an actor](/manual/guides/create-an-actor.html) — the full Actor field set.
- [Build a map](/manual/guides/build-a-map.html) — tilemaps, spawns, and deployments.
- [CLI reference](/reference/cli.html) — all command-line flags.
