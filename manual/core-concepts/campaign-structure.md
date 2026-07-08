# Campaign Structure and Packaging

This page explains how a campaign is laid out on disk and packaged for the engine: the ZIP archive, the data files inside it, how those files merge into one entity set, and how nested plugins extend a campaign. Knowing this layout lets you split your work across many small files and reuse content through plugins without guessing how the engine resolves conflicts.

## Prerequisites

This page assumes you understand entities and how they reference each other:

- [Your first campaign](/manual/getting-started/your-first-campaign.html)
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)

If a term here is unfamiliar, check the [glossary](/manual/getting-started/glossary.html).

## What a campaign is

A [campaign](/manual/getting-started/glossary.html#terms) is a complete game definition packaged as a single `.zip` archive. The archive holds two kinds of content:

- **Data files** that define [entities](/manual/getting-started/glossary.html#terms), written as YAML or JSON.
- **Asset files** that the entities reference, such as images, audio, and text tile grids.

The engine reads the archive, merges every data file into one entity set, validates it, and instantiates the entities. You never edit code to build a game; you edit data files and add assets.

This page draws from an example `demo` campaign throughout.

## Archive layout

Every entry in the archive lives under one top-level folder whose name matches the campaign name. The demo archive is built with a command equivalent to `zip -r demo.zip demo/`, so every entry is prefixed `demo/`. That prefix is the **campaign prefix**.

The campaign prefix matters because asset paths inside your YAML are resolved relative to it. A sprite that stores `assets/hero.png` is read from `demo/assets/hero.png` in the archive. Keep the prefix folder name consistent with the campaign name you load.

At runtime the archive must sit beside the Isometry binary. The engine looks for `<binary-dir>/<dir>/<campaign>.zip`.

> Note: `--network=none` is currently a no-op that loads no campaign. To exercise the load pipeline headless, load an example campaign and run with `--network=host`. See [Hosting](/manual/multiplayer/hosting.html) and [Troubleshooting](/reference/troubleshooting.html).

A campaign typically splits its entities across many small, per-purpose files rather than a few large ones. A partial listing:

```
demo/
  Main.yaml
  heroActor.yaml
  guardActor.yaml
  map.yaml
  skills.yaml
  actions.yaml
  triggers.yaml
  assets/
  demoMap/
  polygons/
```

You are free to organize files however you like. The engine merges them all regardless of name, so split by purpose in whatever way keeps your work readable.

## Data file format

Each data file is a top-level dictionary keyed first by entity type, then by [entity key](/manual/getting-started/glossary.html#terms), then by the entity's fields:

```yaml
EntityType:
  entityKey:
    field: value
```

A single file may contain several entity types, and several files may contribute to the same type. A `heroActor.yaml` shows one file holding two types, an `Actor` and a `Vector`:

```yaml
Actor:
  heroActor:
    name: Hero
    speed: 1.0
    sprite: heroSprite
    base: 8
    perception: 25
    salience: 1
Vector:
  heroStartPos:
    x: 200
    y: 100
```

These are typical example values: the hero's `base` is `8` and `speed` is `1.0`. (A full Actor defines more fields than shown here; see [the entity reference](/reference/entities.html) for the full Actor schema.)

> Warning: Data files use **clean keys** only. Always write the clean key like `name`; the underscore forms such as `name_` are internal and never used in campaign files. This applies to every entity type.

### YAML and JSON

You may write data files as YAML (`.yaml` / `.yml`) or JSON (`.json`), and you may mix both in one campaign. The engine chooses the parser by file extension. This manual uses YAML throughout, and both formats are accepted.

## How files merge into one entity set

At load, the engine reads every `.json`, `.yaml`, and `.yml` file in the archive and merges them into one combined set of entities, grouped by type then by key. The merge is per type and per key, so contributions from different files combine into a single set.

This per-key merge is why you can split entities across files freely. Put all your actions in `actions.yaml` and all your triggers in `triggers.yaml`; the engine treats them as one merged whole.

### Key uniqueness

Entity keys must be unique across the **entire** merged dictionary, not just within one type or one file. If two entities share a key, the engine warns about a naming conflict and skips the conflicting entity, which leaves the campaign missing data. The [validation](/manual/core-concepts/validation.html) phase also rejects duplicate keys outright.

When two files contribute the same key, a later-loaded entity overwrites an earlier one. Do not rely on file load order for correctness; give every entity a distinct key.

## The Main entity

Every campaign requires exactly one [Main](/manual/getting-started/glossary.html#terms) entity. Main names where the game starts: the player-controlled actor and the starting map. An example `Main.yaml`:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
```

Here `actor` is a [KeyRef](/manual/getting-started/glossary.html#terms) to the `heroActor` entity and `map` is a KeyRef to `demoMap`. Both fields are required.

Main also accepts two optional fields:

- `notes` — a free-form description string.
- `plugins` — an ordered list of plugin names that controls precedence. Covered in [Plugins](#plugins) below.

> Note: Having more than one Main entity is an error. Define exactly one, with any key you like; the example uses `demoMain`.

## Assets

Asset files travel in the archive alongside your data files and are loaded on demand by the entities that reference them. The loader recognizes these extensions:

- `.png`, `.jpg`, `.jpeg` — loaded as image textures.
- `.wav` — loaded as audio. Audio must use a 44,100 Hz sample rate.
- `.mp3` — loaded as audio.
- `.txt` — loaded as text, used for layer tile grids and similar data.

Reference an asset by its path relative to the campaign prefix, for example `assets/hero.png`. The engine prepends the prefix when it reads the file.

<a id="plugins"></a>
## Plugins

A [plugin](/manual/getting-started/glossary.html#terms) is a nested `.zip` file placed inside the campaign archive. Plugins let you package reusable content, such as a set of enemies or a shared item library, and drop it into any campaign. The engine treats the archive as containing plugins if any entry ends in `.zip`.

You can place plugin zips anywhere in the archive; a common convention is a `plugins/` folder. A plugin's **name** is its zip basename with the `.zip` extension removed, so `plugins/enemies.zip` has the name `enemies`. That name is what you list in `Main.plugins`.

### How plugins flatten

When the engine detects plugins, it expands and re-roots each one under the campaign prefix, then writes a single merged archive for the session. The plugin's own top-level folder segment is stripped and replaced by the prefix. A plugin `enemies.zip` containing `enemies/goblins.yaml` and `enemies/sprites/goblin.png` becomes `<prefix>/goblins.yaml` and `<prefix>/sprites/goblin.png` in the merged archive.

After flattening, plugin data files merge into the campaign exactly like the campaign's own files. There is no separate plugin namespace; everything lands in one entity set.

Plugins expand recursively. A plugin may itself contain plugin zips to any depth. Within a single plugin, its nested sub-plugins are applied first and the plugin's own files last, so a plugin's own content beats the sub-plugins it bundles.

### Precedence

When the same path or entity key appears in more than one place, precedence decides which wins, from highest to lowest:

1. **The campaign's own top-level files** always win. Your campaign can override anything a plugin provides.
2. **Plugins listed in `Main.plugins`**, in list order. Earlier in the list means higher priority.
3. **Plugins not listed in `Main.plugins`**, ranked lowest, in archive order.

To set plugin order, list their names in the `Main.plugins` array. Suppose two plugins both define a `goblinActor`, and you want `enemies` to win over `monsters`:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
    plugins:
    - enemies
    - monsters
```

Because `enemies` comes first, its `goblinActor` takes priority over the one in `monsters`. Your campaign's own `goblinActor`, if any, would still beat both.

> Note: `Main.plugins` lists plugin **names**, not entity keys, so its entries are not cross-reference validated against entities. If you list a name with no matching plugin zip, the engine logs a warning and ignores the entry.

The merged archive is a build detail used only for loading. For [multiplayer](/manual/multiplayer/networking-and-security.html) integrity, the engine checksums the **original** archive, which already contains the nested plugin zips, so every player verifies identical content including plugins.

## See also

- [Your first campaign](/manual/getting-started/your-first-campaign.html)
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Validation](/manual/core-concepts/validation.html)
- [Hosting](/manual/multiplayer/hosting.html)
- [Networking and security](/manual/multiplayer/networking-and-security.html)
- [Entity reference](/reference/entities.html)
- [Tutorial: build the demo](/manual/tutorial/build-the-demo.html)
