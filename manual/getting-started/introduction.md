# Introduction to Isometry

{{ product_name }} is a data-driven isometric RPG framework. This page explains what that means, who the manual is written for, and how the rest of the manual is organized.

## Introduction

Most game engines ask you to write code. Isometry asks you to write data. The engine is a single executable; you describe a whole game as a set of data files, package them into a [campaign](/manual/getting-started/glossary.html), and the binary loads it. You change the game by changing the data, not the engine.

This page gives you the big picture before you install anything or build anything. It defines the core ideas you meet everywhere else: campaigns are data, [entities](/manual/getting-started/glossary.html) describe everything, and multiplayer is driven from the command line. By the end you will know which page to read next.

## Prerequisites

None. This is the first page of the manual.

When a term is unfamiliar, look it up in the [Glossary](/manual/getting-started/glossary.html). The first use of each core term on this page links there.

## What Isometry is

Isometry is one program. There is no separate server to install and nothing else to set up. The program is the whole product.

You do not edit the program to make a game. You write a [campaign](/manual/getting-started/glossary.html) — a folder of data files plus assets — and the program loads it. Building a new game needs no code changes. This is the central idea of the framework, and the rest of the manual builds on it.

> Note: Campaign data is written in YAML in the example campaign. This manual's examples use YAML throughout. See [Campaign structure](/manual/core-concepts/campaign-structure.html) for what a campaign folder contains.

## Who this manual is for

This manual is written for three kinds of reader:

- **Campaign creators** who want to build a game without programming. You write entity files; the engine does the rest.
- **Indie developers** building a game on top of the framework and wanting to understand how the pieces fit.
- **Server operators** who host multiplayer sessions and run the binary from the command line.

You do not need to be a programmer to follow the getting-started pages. Later reference pages go deeper for developers and operators.

## The big picture

Three ideas carry through the entire manual. Read them once here; every later page assumes them.

### Campaigns are data

A [campaign](/manual/getting-started/glossary.html) is a directory of data files plus assets, packaged into a `.zip` that sits next to the executable. When the binary starts, it loads that zip, validates it, and runs it. An example campaign is the canonical reference; you will see its values throughout this manual.

Because a campaign is just data, you build, change, and share a game by editing files and re-zipping them — not by recompiling anything. [Your first campaign](/manual/getting-started/your-first-campaign.html) walks you through this end to end.

### Entities describe everything

An [entity](/manual/getting-started/glossary.html) is a pure-data record loaded from YAML. There are exactly {{ entity_count }} entity types — Main, Map, Actor, Action, Skill, Strategy, Sprite, and so on. Entities hold data only; they contain no game logic.

Every entity file follows the same shape: an entity type, then a unique [entity key](/manual/getting-started/glossary.html), then the fields. Here is an example [Main](/manual/getting-started/glossary.html) entity, the required entry point that names the starting actor and map:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
```

`"Main"` is the entity type, `"demoMain"` is its key, and `actor` and `map` are its two required fields. Every campaign has exactly one Main entity.

Notice that `"actor": "heroActor"` does not embed the actor — it names it. That string is a [KeyRef](/manual/getting-started/glossary.html): a typed reference to another entity by its key. Entities reference each other this way, and the engine resolves every reference when the campaign loads. If a reference points at a key that does not exist, [validation](/manual/getting-started/glossary.html) reports it before the game runs.

The `heroActor` referenced above is defined in its own file. It shows how much one entity can describe with plain fields and KeyRefs:

```yaml
Actor:
  heroActor:
    name: Hero
    speed: 1.0
    base: 8
    perception: 25
    salience: 1
    sprite: heroSprite
    skills:
    - attackSkill
    - healSkill
    resources:
    - health
    - mana
    - gold
```

The plain values (`speed`, `base`, `perception`, `salience`) set the actor's stats. The KeyRef `sprite` points at one other entity. The lists `skills` and `resources` are [KeyRefArrays](/manual/getting-started/glossary.html) — arrays of KeyRefs. An actor may hold at most nine skills.

> Tip: Always write the clean key like `name`; the underscore forms such as `name_` are internal and never used in campaign files.

This is the entire model. To add a behavior, you write an entity that runs an [Action](/manual/getting-started/glossary.html); to give an actor a goal-driven AI, you reference a [Strategy](/manual/getting-started/glossary.html). Actions call one of the {{ action_count }} built-in action functions. [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) covers the data model in depth.

### Multiplayer is CLI-driven

Isometry treats multiplayer as the default shape of the game, not an add-on. You choose a network mode on the command line when you launch the binary. There are four modes: `none`, `host`, `server`, and `client`. Sessions between a server and its clients are authenticated with RSA keys.

Conceptually, single-player is just a server with one connected client. That is why the same engine serves both: there is no separate code path for "offline."

> Warning: The `--network=none` flag currently loads no campaign on its own. To play single-player today, use the launcher to pick a campaign and host or join. For the exact flags, see the [CLI reference](/reference/cli.html) and [Hosting a session](/manual/multiplayer/hosting.html).

## How a campaign loads

You do not do any of this yourself, but knowing the flow helps you understand error messages. When you start the program:

1. You pick a campaign, log in, and host or join.
2. The campaign is read and checked — every KeyRef and every [dice expression](/manual/getting-started/glossary.html) is cross-checked before the game is allowed to run.
3. Once it passes, the game starts.

If validation fails, the campaign does not load and you get a report naming the problem. [Install and run](/manual/getting-started/install-and-run.html) shows how to get the program and confirm a campaign loads.

## Roadmap of this manual

Read the getting-started pages in order, then branch into guides and reference as you need them:

- [Install and run](/manual/getting-started/install-and-run.html) — download the engine and confirm a campaign loads.
- [Your first campaign](/manual/getting-started/your-first-campaign.html) — build a minimal campaign from scratch.
- [Glossary](/manual/getting-started/glossary.html) — concise definitions of every core term.
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) — the data model in depth.
- [Campaign structure](/manual/core-concepts/campaign-structure.html) — what goes in a campaign folder and how it is packaged.
- [CLI reference](/reference/cli.html) — every command-line flag, including the network modes.
- [Hosting a session](/manual/multiplayer/hosting.html) — run the binary as a host or server.

## See also

- [Install and run](/manual/getting-started/install-and-run.html)
- [Your first campaign](/manual/getting-started/your-first-campaign.html)
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Campaign structure](/manual/core-concepts/campaign-structure.html)
- [Glossary](/manual/getting-started/glossary.html)
- [CLI reference](/reference/cli.html)
