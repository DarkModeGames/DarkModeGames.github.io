# Validation

Before a campaign loads, the engine checks every entity you wrote and refuses to start the game if anything is wrong. This page explains the four checks that run, how to read the error each one produces, and how to fix the most common failures.

## Introduction

Validation is the engine's safety net. Because a [campaign](/manual/getting-started/glossary.html#terms) is pure data, a typo in a [KeyRef](/manual/getting-started/glossary.html#terms) or a misspelled action function would otherwise surface as a confusing crash deep in gameplay. Instead, the validator inspects the whole campaign up front, gathers every problem at once, and shows you a single list. The campaign loads only when that list is empty.

Validation runs as a pipeline of four phases, in this order: schema, cross-reference, asset, and dice. Each phase covers a different class of mistake. All four run when a campaign actually loads, that is, in `--network=host`, `--network=server`, or `--network=client`.

> Note: Validation does not run in `--network=none`. That mode is currently a no-op and loads no campaign at all (see the [CLI reference](/reference/cli.html) and [Troubleshooting](/reference/troubleshooting.html)). To exercise validation headless, load the campaign with `--network=host`.

## Prerequisites

This page assumes you understand entities and how they reference each other:

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Campaign structure](/manual/core-concepts/campaign-structure.html)

## When validation runs

The engine first merges every `.json`, `.yaml`, and `.yml` file in the campaign into one combined record, grouped by [entity](/manual/getting-started/glossary.html#terms) type. It then validates that combined record as a whole. Because the checks see the entire campaign at once, rules like "exactly one Main" and "no duplicate keys" span all of your files, not just one.

If any phase reports an error, the load is aborted, the loading screen is hidden, and a validation error modal appears listing every problem found. The modal's title reads `Found N error(s)`, and its Close button quits the game. On success, the engine logs `Campaign validation passed` and the campaign loads.

Validation never stops at the first error. It collects every problem across all four phases, so one load attempt tells you everything to fix.

## How to read a validation error

Every error line follows one of two formats.

When the problem concerns a specific field of an entity:

```
[CATEGORY] EntityType.entityKey.fieldName: message
```

When the problem concerns the entity as a whole (or the campaign as a whole):

```
[CATEGORY] EntityType.entityKey: message
```

Read each part left to right:

- **CATEGORY** names the kind of failure, such as `MISSING_REQUIRED_FIELD` or `KEYREF_UNRESOLVED`. There are 13 categories in total, listed at the end of this page.
- **EntityType.entityKey** locates the entity. For example, `Action.attackAction` points at the `attackAction` entity in your [`Action`](/reference/entities.html#action) block.
- **fieldName**, when present, names the exact field at fault.
- **message** explains what is wrong and often what was expected.

For example, this line means the [`Main`](/reference/entities.html#main) entity named `demoMain` points its `actor` field at a key that does not exist:

```
[KEYREF_UNRESOLVED] Main.demoMain.actor: KeyRef heroActor does not exist in campaign
```

To fix it, you either correct the key in `actor` or add the missing `heroActor` entity.

## Phase 1: schema checks

The schema phase checks each entity against the rules for its type: which fields exist, which are required, what type each field holds, and any value constraints. Every one of the {{ entity_count }} entity types has a schema.

Schema checks produce these categories:

- **UNKNOWN_TYPE** — you used an entity type that has no schema, for example a typo like `Acter` instead of [`Actor`](/reference/entities.html#actor). The message reads `Unknown entity type X - no schema defined`.
- **MISSING_REQUIRED_FIELD** — a required field is absent.
- **EXTRA_FIELD** — you included a field the schema does not define. The message reads `Field X not defined in <Type> schema`. This usually means a misspelled field name.
- **INVALID_TYPE** — a field holds the wrong type of value.
- **CONSTRAINT_VIOLATION** — a value breaks a `min`/`max` or length limit.

The [Main](/manual/getting-started/glossary.html#terms) entity is the smallest example. It requires `map`; `actor` is optional:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
```

Omit `map` and the schema phase reports:

```
[MISSING_REQUIRED_FIELD] Main.demoMain.map: ...
```

### A note on numbers

A field typed `INT` accepts a whole number written as `8` or `8.0`; only a genuinely fractional value like `8.5` is rejected. A float with a decimal part on an `INT` field fails:

```
[INVALID_TYPE] Actor.heroActor.base: Expected INT, got FLOAT with decimal value 8.5
```

For the full list of every type's fields, see [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html).

## Phase 2: cross-reference checks

The cross-reference phase verifies the relationships between entities. It runs four checks in order: the required single Main, no duplicate keys, all KeyRefs resolve, and all action functions exist.

### Exactly one Main

Every campaign needs exactly one Main entity, because Main names the starting [actor](/manual/getting-started/glossary.html#terms) and starting map. Zero Mains reports:

```
[MAIN_ENTITY_MISSING] ...: Campaign requires exactly one Main entity (found 0)
```

More than one reports `MAIN_ENTITY_DUPLICATE` with the count found. This commonly happens when two campaign files each define a Main, since validation spans all files.

### No duplicate keys

Every [entity key](/manual/getting-started/glossary.html#terms) must be unique across the entire campaign, not just within its own type. If you reuse a key in two different entity types, the engine cannot tell which one a reference means:

```
[DUPLICATE_KEY] ...: Duplicate key heal found in entity types: Action, Skill
```

Rename one of them so every key is globally unique.

### KeyRefs resolve

A KeyRef check confirms two things: that the referenced key exists, and that it resolves to the type the field expects. A missing key reports:

```
[KEYREF_UNRESOLVED] Action.attackAction.parameters: KeyRef attackDamageParam does not exist in campaign
```

A key that exists but points at the wrong type also reports `KEYREF_UNRESOLVED`, with a message that names both types:

```
[KEYREF_UNRESOLVED] Main.demoMain.actor: KeyRef demoMap resolves to type Map, expected Actor
```

Empty-string KeyRefs are skipped, so an optional reference you leave blank does not error.

### Action functions exist

Each [Action](/manual/getting-started/glossary.html#terms) names a built-in action function in its `do` field. There are exactly {{ action_count }} valid action functions. If `do` names one that does not exist, you get:

```
[INVALID_ACTION_NAME] Action.attackAction.do: Action function attck_resource_target does not exist
```

Some action functions require parameters. An `attackAction` that uses `minus_resource_target` supplies its parameters as a flat array of KeyRefs to sibling [Parameter](/manual/getting-started/glossary.html#terms) entities:

```yaml
Action:
  attackAction:
    name: Attack
    do: minus_resource_target
    parameters:
    - attackResourceParam
    - attackDamageParam
Parameter:
  attackResourceParam:
    key: resource
    value: health
  attackDamageParam:
    key: expression
    value: (1d6)-1
```

The `parameters` array holds KeyRefs only; the [`Parameter`](/reference/entities.html#parameter) entities live in their own top-level block. If an action function needs parameters and the `parameters` array is empty, you get:

```
[INVALID_ACTION_PARAM] Action.attackAction.parameters: Action function minus_resource_target requires parameters: ...
```

> Note: This check confirms only that `parameters` is non-empty when the function requires parameters. It does not yet verify that each specific parameter name is present, so a wrong parameter name still passes Phase 2. As an example, `set_destination_self` requires a `destination` parameter, a KeyRef to a [Vector](/reference/entities.html#vector).

For the full list of functions and their parameters, see [Define actions](/manual/guides/define-actions.html).

## Phase 3: asset checks

The asset phase runs only when the campaign is loaded from a `.zip` archive. It confirms that every file an entity points to actually exists inside the archive and has an allowed extension.

These fields are checked, and the allowed extensions:

- Image (`png`, `jpg`, `jpeg`): `Sprite.texture`, `Resource.icon`, `Measure.icon`, `Skill.icon`, `Waypoint.icon`, `Floor.texture`, `TileSet.texture`, `Parallax.texture`.
- Sound (`mp3`, `wav`, `ogg`): `Sound.source`. Use `mp3`; the other two pass validation but are unreliable at runtime.
- Layer grids: `Layer.source`. A grid file has **no extension** (for example `/maps/town/ground`), and extensionless files are not extension-checked.

Path matching is forgiving: a leading `/` is stripped, then a match succeeds if any file in the archive ends with the path you gave. So `/assets/icons/sun.png` matches a file stored at that path inside the archive.

A missing file reports:

```
[ASSET_NOT_FOUND] Sprite.heroSprite.texture: Asset /assets/heroSprite.png not found in campaign archive
```

A file with the wrong extension also reports `ASSET_NOT_FOUND`, with a message that lists the allowed extensions:

```
[ASSET_NOT_FOUND] Sound.theme.source: Asset /assets/theme.txt has invalid extension .txt, expected one of: ...
```

## Phase 4: dice checks

The dice phase validates every [dice expression](/manual/getting-started/glossary.html#terms) field: `Measure.expression`, `Resource.min`/`max`, `Condition.left`/`right`, `Timer.total`/`interval`, and `Sound.scale`. Dice-typed action parameters are checked in phase 2.

Each expression must be well-formed, use only `0123456789()*%/+-dD<>`, and every `$key`/`@key` must name a real [Resource](/reference/entities.html#resource), [Measure](/reference/entities.html#measure), or built-in value. Empty strings are skipped. A `luckMeasure` that uses a valid expression, `1d20`, and a damage parameter that uses `(1d6)-1` both pass. Any character outside the set fails:

```
[INVALID_DICE] Measure.luckMeasure.expression: Invalid dice expression 1k20 - invalid character k at position 1
```

For the notation itself, see [Dice expressions](/manual/core-concepts/dice-expressions.html).

## The error categories

The 13 categories you may see, grouped by the phase that produces them:

- Schema: `UNKNOWN_TYPE`, `MISSING_REQUIRED_FIELD`, `EXTRA_FIELD`, `INVALID_TYPE`, `CONSTRAINT_VIOLATION`.
- Cross-reference: `MAIN_ENTITY_MISSING`, `MAIN_ENTITY_DUPLICATE`, `DUPLICATE_KEY`, `KEYREF_UNRESOLVED`, `INVALID_ACTION_NAME`, `INVALID_ACTION_PARAM`.
- Asset: `ASSET_NOT_FOUND`.
- Dice: `INVALID_DICE`.

## See also

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Campaign structure](/manual/core-concepts/campaign-structure.html)
- [Dice expressions](/manual/core-concepts/dice-expressions.html)
- [Define actions](/manual/guides/define-actions.html)
- [CLI reference](/reference/cli.html)
