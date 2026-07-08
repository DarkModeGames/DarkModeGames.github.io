# Troubleshooting

This page collects the most common problems you hit while building an {{ product_name }} campaign and gives each one a symptom, a cause, and a fix. Use it as a lookup table: find the line that matches what you see in `log.txt`, then follow the steps.

## Introduction

Most campaign problems fall into a handful of categories: the campaign refuses to load, validation rejects an entity, the map looks wrong, an NPC sits still, or an asset cannot be found. Every error string quoted here is taken from the validator and engine, so you can match it character-for-character against your log. Where a behavior looks like a bug but is not, this page says so and points you at the right fix.

A [campaign](/manual/getting-started/glossary.html) is a folder of YAML [entities](/manual/getting-started/glossary.html) plus assets, packaged as a `.zip`. When something goes wrong, the engine almost always tells you in the log. Read the log first.

## Prerequisites

You should already understand how a campaign is laid out and how it is checked before it loads:

- [Campaign structure](/manual/core-concepts/campaign-structure.html)
- [Validation](/manual/core-concepts/validation.html)

## Read the log first

Nothing is logged until you ask for it. The default log level is `NONE`, so an empty log is normal, not a sign of failure. Pass `--log-level` to turn logging on.

```bash
isometry --headless --campaign=demo --network=host \
  --port=5000 --username=p --secret=p --log-level=INFO
```

The log is written to `log.txt` in the directory next to the executable, the same folder where your campaign `.zip` sits. Levels are `NONE`, `FATAL`, `ERROR`, `WARN`, `INFO`, `DEBUG`, and `TRACE`, and the value is case-insensitive, so `--log-level=info` works too.

> Note: Older documentation pointed at `~/.local/share/isometry/logs/` or an `AppData` path. That is wrong. The log lives beside the binary.

When a campaign loads cleanly you see:

```
Campaign validation passed
```

When it does not, you see a count of problems:

```
Campaign validation failed with 3 error(s)
```

Each error is logged on its own line above that summary. The rest of this page explains what those lines mean.

## The campaign does nothing in single-player

**Symptom:** You run with `--network=none` (or omit `--network`) and nothing loads. No terrain, no actor, no validation output.

**Cause:** `--network=none` is currently a no-op. The startup code only handles the `host`, `server`, and `client` modes; the default branch does nothing. This is a known issue.

**Fix:** Start the game from the launcher to play single-player. To exercise the load pipeline from the command line, use `--network=host`:

```bash
isometry --headless --campaign=demo --network=host \
  --port=5000 --username=p --secret=p --log-level=INFO
```

Success looks like a `Campaign validation passed` line followed by one or more `Successfully created Entity` lines. See the [CLI reference](/reference/cli.html) for the full flag list.

## Validation errors

Validation runs in four phases, and the campaign loads only if all four pass:

1. **Schema** — each entity has the right fields and types.
2. **Cross-reference** — every [KeyRef](/manual/getting-started/glossary.html) (a reference to another entity by its key) resolves, there is exactly one Main entity, no key is duplicated, and action functions exist.
3. **Asset paths** — referenced files exist in the archive. This phase runs only when a `.zip` is loaded (host, server, or client), not for a raw directory load.
4. **Dice expressions** — dice fields contain valid notation.

The sections below group the errors by phase. For the wider picture, read [Validation](/manual/core-concepts/validation.html).

### Schema errors (phase 1)

**`Unknown entity type 'X' - no schema defined`** — A top-level key in your YAML is not one of the {{ entity_count }} entity types. Check the spelling against [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html).

**`Required field 'X' is missing`** — Required fields are rare. The two you are most likely to hit are `Main.actor` and `Main.map`, both of which every campaign needs.

> Warning: An Actor does not require a `sprite`. There is no "Actor missing required field sprite" error. If a guide told you otherwise, it is out of date.

**`Field 'X' not defined in <Type> schema`** — You used a field name the schema does not know, almost always a typo. Compare against [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html).

**Type mismatches** — `Expected <TYPE>, got <type>` for scalars, `Expected KeyRef (String), got <type>`, or `Expected KeyRefArray (Array), got <type>`. You wrote a value of the wrong shape, for example a number where a KeyRef string belongs.

**`Expected INT, got FLOAT with decimal value X`** — A field typed `INT` accepts a whole number written as `8` or `8.0`; only a genuinely fractional value like `8.5` is rejected. Float fields accept whole-looking numbers like `20` without complaint, so do not "fix" them by adding a decimal point.

**Constraint errors** — `Value X is less than minimum Y`, `Value X exceeds maximum Y`, `Array length N exceeds maximum M`, and `String length N exceeds maximum M` (plus the minimum variants). For example, an actor's `skills` array may hold at most nine entries, and `perception`, `base`, `salience`, and `speed` must all be zero or greater.

### Cross-reference errors (phase 2)

**`Campaign requires exactly one Main entity (found 0)`** or **`(found N)`** — Every campaign needs exactly one [Main](/manual/getting-started/glossary.html) entity. Zero means no Main file; more than one means you defined Main twice.

**`Duplicate key 'X' found in entity types: TypeA, TypeB`** — The same [entity key](/manual/getting-started/glossary.html) is used by two different entity types. Keys must be unique across the whole campaign. Rename one.

**`KeyRef 'X' does not exist in campaign`** — A field points at an entity key that no file defines. Check for a typo in the reference or a missing entity file.

**`KeyRef 'X' resolves to type 'A', expected 'B'`** — The reference resolves, but to the wrong kind of entity. For example, you pointed a `sprite` field at an Actor instead of a Sprite.

**`Action function 'X' does not exist in actions.gd`** — An Action's `do` value is not one of the {{ action_count }} built-in action functions. See the [action function reference](/reference/actions.html) for the full list.

**`Action function 'X' requires parameters: a, b`** — The function needs parameters, but the action's `parameters` array is empty.

### How action parameters are shaped

This last error trips people up, so it is worth showing the correct shape. An action's `parameters` is a **flat array of KeyRefs** to Parameter entities. The Parameter entities themselves live in a sibling top-level `Parameter` block, never nested inside the action.

```yaml
Action:
  healAction:
    do: add_resource_self
    parameters:
    - pHeal
    - pAmount
Parameter:
  pHeal:
    value: health
  pAmount:
    value: 1d6
```

The `parameters` array lists only the keys; each key resolves to a Parameter in the sibling block. See [Define actions](/manual/guides/define-actions.html) for the details.

### Asset errors (phase 3)

**`Asset 'path' not found in campaign archive`** — A field that names a file points at a path the `.zip` does not contain. Asset checks run only when a `.zip` is loaded, so this never fires on a raw directory load.

**`Asset 'path' has invalid extension '.x', expected one of: ...`** — The file exists but has the wrong type for that field.

Path matching is suffix-based: an archive entry counts as a match when its path ends with the asset path (after stripping a leading slash). So a small difference in the leading directory is forgiven, but the filename and trailing folders must match. The accepted extensions are:

- Texture and icon fields (`Sprite.texture`, `Resource.icon`, `Measure.icon`, `Skill.icon`, `Waypoint.icon`, `Floor.texture`, `TileSet.texture`, `Parallax.texture`): `png`, `jpg`, `jpeg`.
- `Sound.source`: `mp3`, `wav`, `ogg`.
- `Layer.source`: `tmj`.

### Dice errors (phase 4)

**`Invalid dice expression 'E' - invalid character 'c' at position N`** — A dice field contains a character outside the allowed set `0123456789()*%/+-dD<>`. The fields checked are `Measure.expression`, `Sound.scale`, `Timer.total`, and `Timer.interval`. See [Dice expressions](/manual/core-concepts/dice-expressions.html) for valid notation.

## The map is invisible or gray

A blank or gray play area has two very different causes. Diagnose which one you have before you change anything.

### Ghost tiles versus fog of war

A tile marked `ghost: true` is never drawn at all: the engine skips it when painting the map, and it gets no discovery polygon. Ghost tiles are meant only for invisible navigation helpers. If you set `ghost: true` on a walkable floor tile by mistake, that terrain disappears.

[Fog of war](/manual/getting-started/glossary.html) is different. Fog tiles exist and are drawn, but start hidden until the player discovers them. The player's discovery radius, based on the actor's [perception](/manual/getting-started/glossary.html), reveals tiles as it overlaps them. Terrain that lifts as you walk is fog of war working correctly, not a bug.

### Fix invisible floor tiles

**Symptom:** Floor that should be solid is missing entirely and never appears, even when the player stands on it.

**Cause:** The tile is flagged `ghost: true`.

**Fix:** Remove `ghost` (or set it to `false`) on floor tiles. Reserve `ghost: true` for tiles you genuinely never want rendered.

```yaml
symbol: .
index: 0
navigation: roof
ghost: false
```

### Fix terrain that stays dark

**Symptom:** Terrain is dark until the player walks near it, then fades in.

**Cause:** This is fog of war. The actor's perception circle is small, or the actor spawned away from the area.

**Fix:** Raise the actor's `perception` to widen the reveal radius. The demo hero uses `25`.

```yaml
heroActor:
  name: Hero
  base: 8
  perception: 25
  salience: 1
  speed: 1.0
```

> Tip: If a wrong `Main.map` value loads the wrong (or no) terrain, you get a gray screen too. Confirm `Main.map` points at the map you expect before chasing tile flags.

## An NPC never appears

**Symptom:** An NPC you defined is nowhere on the map.

**Cause:** Defining an Actor and a Deployment is not enough. An NPC spawns only when its Deployment is listed in the Map's `deployments` array. A Deployment that no Map references spawns nothing.

**Fix:** Add the Deployment's key to the Map's `deployments` array. In the demo, `patrollerDeploy` places `patrollerActor` at `patrollerPos`.

```yaml
patrollerDeploy:
  actor: patrollerActor
  location: patrollerPos
```

The Map entity must then reference it:

```yaml
deployments:
- patrollerDeploy
```

See [Build a map](/manual/guides/build-a-map.html) for the full Map entity.

## An NPC appears but never acts

**Symptom:** The NPC is on the map but stands still and does nothing.

**Cause:** AI runs through a chain, and any broken link stops the actor. The Actor needs a `strategy`; the [Strategy](/manual/getting-started/glossary.html) needs `behaviors`; each [Behavior](/manual/getting-started/glossary.html) needs `goals` ([Conditions](/manual/getting-started/glossary.html)) that can be satisfied and an `action`. A missing strategy, an empty behaviors list, or goals that can never be true all freeze the NPC.

**Fix:** Walk the chain from the actor down.

First, confirm the actor has a strategy. The demo patroller uses `followStrategy`:

```yaml
patrollerActor:
  name: Patroller
  speed: 0.8
  strategy: followStrategy
```

Then confirm the strategy lists behaviors and each behavior has goals and an action:

```yaml
patrolBehavior:
  goals:
  - patrolGoal
  action: set_destination_self
```

A Behavior's `action` is special: it may be either an Action entity key or a bare action function name such as `set_destination_self`, which takes a `destination` parameter referencing a Vector. If it is neither, validation reports `KeyRef 'X' does not exist in campaign and is not a valid action function`.

If the behavior still does not fire, check that its goal Conditions can actually be true. An unsatisfiable goal means the behavior never runs. See [Add NPC AI](/manual/guides/add-npc-ai.html) for the full pattern.

## Asset paths and templating

Two smaller traps round out the list.

**Wrong asset path.** When a campaign loads from a `.zip` but an icon or texture does not show, the path field probably does not match an archive entry. Because matching is suffix-based, the failure is usually a wrong filename or trailing folder rather than a wrong leading directory. Check the exact path against the files inside the `.zip`.

**Broken dialog text.** [Dialog](/manual/getting-started/glossary.html) text shows live actor values with template placeholders. Use `{% raw %}{{@resource}}{% endraw %}` for the target actor and `{% raw %}{{$resource}}{% endraw %}` for the caller. Use these tokens only inside a Dialog entity's `text` field.

{% raw %}
```
HP: {{@health}} / Your luck: {{$luck}}
```
{% endraw %}

Here `{% raw %}{{@health}}{% endraw %}` reads the viewed actor's `health` resource and `{% raw %}{{$luck}}{% endraw %}` reads the opening actor's `luck` measure.

## There is no save file to delete

**Symptom:** You read older advice to fix a problem by deleting a save file named with a numeric hash.

**Cause:** That advice does not apply. The engine has no game-save persistence; there is no save file to delete.

**Fix:** Ignore it. A gray or invisible map is explained by ghost floor tiles or fog of war (above), not by a stale save.

## Networking problems

Multiplayer connection failures, credential rules, and campaign version mismatches between client and server are covered on their own page, which is the authoritative source for the exact runtime messages. See [Networking and security](/manual/multiplayer/networking-and-security.html).

## See also

- [Validation](/manual/core-concepts/validation.html)
- [CLI reference](/reference/cli.html)
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Action function reference](/reference/actions.html)
- [Build a map](/manual/guides/build-a-map.html)
- [Add NPC AI](/manual/guides/add-npc-ai.html)
- [Dice expressions](/manual/core-concepts/dice-expressions.html)
- [Networking and security](/manual/multiplayer/networking-and-security.html)
- [Install and run](/manual/getting-started/install-and-run.html)
