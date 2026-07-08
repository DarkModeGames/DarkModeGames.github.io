# Design Philosophy

This page explains why {{ product_name }} is built the way it is. Understanding the reasoning behind the engine's core decisions helps you predict how it behaves and why your campaigns are structured the way they are.

## Introduction

{{ product_name }} is a single binary that turns YAML into a playable isometric RPG. It has no backend service, no database, and no Docker. Every game is data: you describe what your world contains, and the engine resolves how it plays.

This page is conceptual. It documents no entities and asks you to write no YAML. Instead it covers five decisions that shape everything else: one self-contained binary, data-driven campaigns, command-line launch, multiplayer-first networking, and a small set of deliberate constraints.

## Prerequisites

- [What is Isometry](/manual/getting-started/introduction.html)

The [glossary](/manual/getting-started/glossary.html) defines the core terms used below.

## One self-contained binary

{{ product_name }} ships as a single executable per platform. There is no server you stand up, no container you build, and no service the game phones home to. You download one binary and run it.

A [campaign](/manual/getting-started/glossary.html#terms) is a `.zip` archive that sits beside that binary. At launch the engine reads the archive from the executable's own directory and loads it.

This keeps distribution simple. To ship a game you ship two files: the {{ product_name }} binary and your campaign `.zip`. There is nothing to deploy and nothing to keep running.

## Data-driven campaigns

A whole game is built from YAML files in a campaign folder, with no code changes. The demo campaign built in the [tutorial](/manual/tutorial/build-the-demo.html) is the canonical example.

This is the central idea of {{ product_name }}: [entities](/manual/getting-started/glossary.html#terms) hold data only and contain no game logic. There are exactly {{ entity_count }} entity types, and all game behavior is resolved by the engine through its built-in action functions. You describe *what* exists; the engine decides *how* it runs.

For example, the demo hero is just data:

```yaml
Actor:
  heroActor:
    name: Hero
    speed: 1.0
    base: 8
    perception: 25
    salience: 1
    resources:
    - health
    - mana
    - gold
    skills:
    - attackSkill
    - healSkill
```

Nothing here is procedural. The actor names its parts, and the engine assembles the running character. Because campaigns are declarative, they are also easy to share, diff, and mod: changing the game means editing text, not recompiling the binary.

See [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) for how entities reference one another, and the [campaign structure](/manual/core-concepts/campaign-structure.html) reference for how the files fit together.

### Why this makes campaigns predictable

Because the game is data, the engine can check it before it runs. [Validation](/manual/core-concepts/validation.html) is schema-driven: it cross-checks every reference and every dice expression before a campaign loads. A campaign that loads cleanly logs `Campaign validation passed`, then `Successfully created Entity` for each entity it builds.

This is only possible because logic lives in the engine, not in your files. YAML has no hidden control flow to mis-execute, so most mistakes surface as a clear validation error instead of a runtime surprise.

## Command-line launch

You configure {{ product_name }} through command-line arguments, parsed at startup. The common flags are `--campaign`, `--network`, `--port`, `--username`, `--secret`, and `--log-level`. The [CLI reference](/reference/cli.html) lists them in full.

This is deliberate. A command-line front door means the launcher is a separable layer: you can wrap the binary in your own launcher UI rather than ship the one in the box. {{ product_name }} provides separate launcher and engine builds precisely because the launcher is a front end, not the engine.

## Multiplayer-first

Networking is part of the architecture, not an add-on. Sessions use secure encrypted connections. The network mode is one of four values: `NONE`, `HOST`, `SERVER`, or `CLIENT`.

Designing multiplayer in from the start simplifies the single-player case rather than complicating it. Single-player is just a host session: single-player and multiplayer use the same system underneath, so a solo game runs the same way as a networked one.

> Warning: `--network=none` is currently a no-op and loads no campaign. To run a single-player session, use `--network=host`. See [Networking and security](/manual/multiplayer/networking-and-security.html).

## Deliberate constraints

{{ product_name }} is opinionated where being opinionated keeps campaigns uniform. Two constraints are worth knowing up front.

### Two numeric primitives

Actor stats use two clear systems, not one. A [Resource](/manual/getting-started/glossary.html#terms) is a stored, trackable integer with a `default`, `min`, and `max`. A [Measure](/manual/getting-started/glossary.html#terms) is a value derived from a [dice expression](/manual/getting-started/glossary.html#terms) such as `1d20`, calculated fresh each time it's read.

In the demo, health, mana, and gold are Resources, while `luckMeasure` is a Measure. There is no special "experience" stat built into the engine; any stat like that is simply a campaign-defined Resource. Keeping exactly two primitives makes every stat in every campaign fall into one of two well-understood shapes. See [Add resources and measures](/manual/guides/add-resources-and-measures.html).

### A nine-skill bar

An actor can carry up to nine skills, and the HUD binds those skills to the slots `action_1` through `action_9`.

This is a constraint on the actor, not on a skill. A [Skill](/manual/getting-started/glossary.html#terms) entity itself has no nine-item limit. The demo hero carries only two skills (`attackSkill` and `healSkill`), well under the cap. See [Add skills](/manual/guides/add-skills.html).

> Note: A small, fixed skill bar keeps the player's controls consistent across every campaign, so the HUD never has to grow unbounded.

## See also

- [What is Isometry](/manual/getting-started/introduction.html)
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Campaign structure](/manual/core-concepts/campaign-structure.html)
- [Validation](/manual/core-concepts/validation.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Add skills](/manual/guides/add-skills.html)
- [Networking and security](/manual/multiplayer/networking-and-security.html)
- [CLI reference](/reference/cli.html)
