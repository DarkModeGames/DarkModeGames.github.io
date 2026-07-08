# Entities and KeyRefs

This page explains Isometry's central data model: every part of your game is a typed data record, addressed by a campaign-unique key, and entities reference one another by key through KeyRefs. Understanding this shape is what lets you build an entire game in YAML without writing code.

## Introduction

In Isometry, you do not write game logic. You write *data*. A hero, a map, a healing spell, a patrol route, and a damage number are all the same kind of thing under the hood: an [entity](/manual/getting-started/glossary.html#terms), a pure-data record loaded from YAML.

There are exactly {{ entity_count }} entity types. Each entity carries fields that describe one piece of your game, and many of those fields point at *other* entities. A pointer from one entity to another is a [KeyRef](/manual/getting-started/glossary.html#terms): the key of the target entity, stored as a plain string. The engine resolves those references when the campaign loads.

Once this clicks, every other page in the manual reads the same way: define entities, wire them together with KeyRefs.

## Prerequisites

- [Your first campaign](/manual/getting-started/your-first-campaign.html) — you have loaded a campaign and seen its files.
- [Glossary](/manual/getting-started/glossary.html) — for any term used here without a full definition.

## The shape of an entity

Every entity file follows one shape, with no exceptions:

```yaml
<Type>:
  <key>:
    <field>: <value>
```

The outer key is the entity **type**, one of the {{ entity_count }} valid type names. The next key is the **entity key**, a name you choose that must be unique across the whole campaign. The inner dictionary holds the entity's fields.

Here is an example [Main](/manual/getting-started/glossary.html#terms) entity, the single record that defines a campaign's starting state:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
```

`Main` is the type. `demoMain` is the key. The two fields, `actor` and `map`, are both KeyRefs — covered in the next section.

> Note: The entity key is the top-level YAML key, not a field inside the entity. You never write a `"key"` field; the key is the name you nest the entity under.

### Entities are pure data

An entity holds data and nothing else. There is no behavior inside an Actor or a Map record. Each entity type only declares its fields. Logic lives elsewhere — actions, strategies, and the engine itself act *on* entity data. This separation is why the same loader handles every type.

### Many entities in one file

You are not limited to one entity per file. A single YAML file can contain several types and several keys; the loader merges every YAML and JSON file in the campaign into one dataset before validating. A `heroActor.yaml` can do exactly this — holding an `Actor` and a `Vector` side by side:

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
Vector:
  heroStartPos:
    x: 200
    y: 100
```

Organize files however suits you. The engine sees the merged whole, so file boundaries are for your convenience, not the engine's.

## Referencing other entities with KeyRefs

Entities rarely stand alone. An actor needs a sprite; a campaign needs a starting map. A [KeyRef](/manual/getting-started/glossary.html#terms) is how one entity points at another: you write the target entity's key as a string.

In the Main entity above, `"actor": "heroActor"` is a KeyRef to the Actor whose key is `heroActor`. `"map": "demoMap"` is a KeyRef to the Map whose key is `demoMap`. The field value *is* the target key — nothing more.

A KeyRef resolves to a single entity. The field is just a string.

```yaml
sprite: heroSprite
```

This says: the actor's sprite is the Sprite entity keyed `heroSprite`.

### Pointing at many entities with a KeyRefArray

When a field needs to reference several entities, use a [KeyRefArray](/manual/getting-started/glossary.html#terms): an array of keys. Each element is itself a KeyRef.

A hero's skills are a KeyRefArray:

```yaml
skills: [attackSkill, healSkill]
```

This references two Skill entities by their keys. KeyRefArrays appear all over the Actor type — `resources`, `measures`, `triggers`, and `timers` are all lists of keys.

> Tip: A KeyRef holds one key string; a KeyRefArray holds an array of key strings. If a field can point at more than one entity, it is a KeyRefArray.

## A worked example: the hero actor

The Actor type ties the most concepts together. Here is an example `heroActor`, the player-controlled character:

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
```

Read the fields by kind:

- **Plain values** describe the actor directly. `name` is the display name `"Hero"`. `speed` is the movement rate `1.0`. `base` is the `8`-pixel footprint. `perception` is the `25`-pixel vision range. `salience` is `1`, how easily others detect this actor.
- **Single KeyRefs** point at one entity each. `sprite` references `heroSprite`, `menu` references `heroMenu`, and `group` references the Group `playerGroup`.
- **KeyRefArrays** point at lists. `skills`, `resources`, `measures`, `triggers`, and `timers` each name several entities by key. `public` and `private` list which [Resource](/manual/getting-started/glossary.html#terms) entities show on which data plate.

Every key in this file — `heroSprite`, `attackSkill`, `health`, `playerGroup`, and the rest — is defined as its own entity elsewhere in the campaign. The Actor record only points; it does not contain those entities.

Each [guide](/manual/guides/create-an-actor.html) covers the fields of the types it builds.

## Keys must be unique

An entity key must be unique within its type. If two entities in the same type share a key, the loader keeps the first and warns about the conflict, dropping the duplicate's data. Choose distinct keys.

### References are checked before the campaign loads

KeyRefs are checked up front. Before any entity is created, the [campaign validator](/manual/core-concepts/validation.html) cross-checks that every referenced key actually exists. A KeyRef pointing at a key that no entity defines is a validation error, and the campaign refuses to load with a clear message. You catch broken references at load time, not mid-game.

## Clean keys versus internal aliases

Always write the clean key like `name`; the underscore forms such as `name_` are internal and never used in campaign files. A resource's bounds are `min` and `max`, never `min_` or `max_`.

```yaml
name: Hero
```

## See also

- [Campaign structure](/manual/core-concepts/campaign-structure.html) — how entity files and assets are organized into a campaign.
- [Validation](/manual/core-concepts/validation.html) — how the engine checks entities and KeyRefs before load.
- [Create an actor](/manual/guides/create-an-actor.html) — build the Actor used as the example here.
- [Define actions](/manual/guides/define-actions.html) — wire actions and their Parameter entities together.
- [Glossary](/manual/getting-started/glossary.html) — definitions for the terms on this page.
- [Philosophy](/reference/philosophy.html) — why Isometry is data-driven.
