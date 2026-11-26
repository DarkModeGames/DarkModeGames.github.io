# Glossary

Concise definitions of the core terms you meet when building an Isometry campaign.

## Introduction

This page defines the vocabulary used throughout the Isometry developer manual. Read it once to get oriented, then return to it whenever a term in another page is unfamiliar. The terms below are the building blocks of every campaign: the data you write, the engine concepts that load it, and the words the manual uses to describe gameplay.

Terms are listed alphabetically. Each definition is one or two sentences, with links to the pages where you put the concept to work.

## Prerequisites

You do not need to read these pages first, but the terms here make more sense once you have seen them in context:

- [What is Isometry](/manual/getting-started/introduction.html)
- [Installation](/manual/getting-started/install-and-run.html)
- [Your first campaign](/manual/getting-started/your-first-campaign.html)

<a id="terms"></a>
## Terms

**Action**
: An entity that changes the game when it runs, such as dealing damage or moving an actor. Each action names one of the {{ action_count }} built-in action functions in its `do` field and passes [parameters](#terms) to it.

**[Actor](/reference/entities.html#actor)**
: A character in the game world, either player-controlled or AI-driven. An actor entity defines fields such as `name`, `speed`, `base`, `perception`, and `salience`, and references its sprite, skills, resources, and strategy.

**base**
: The size of an actor's footprint, used for placement and to decide what is solid. In the example hero actor, `base` is `8`.

**[Behavior](/reference/entities.html#behavior)**
: A goal-and-action pair used by AI. A behavior lists [conditions](#terms) (its `goals`) and one action. The action keeps running **until** the goals are met; then the strategy moves on to the next behavior.

**campaign**
: A complete game definition: a folder of YAML entity files plus assets, packaged as a `.zip` and placed in the engine's `campaigns` folder.

**[Condition](/reference/entities.html#condition)**
: An entity that compares two values with an operator (`==`, `!=`, `<`, `>`, `<=`, `>=`). Conditions drive an action's `if`/`else` logic and a behavior's goals.

**dice expression**
: A string in tabletop dice notation, such as `1d20` or `(1d6)-1`, that the engine rolls to produce a number. Measures and damage parameters use dice expressions.

**entity**
: A pure-data record loaded from YAML. There are exactly {{ entity_count }} entity types (Main, Actor, Action, [Skill](/reference/entities.html#skill), [Strategy](/reference/entities.html#strategy), and so on); entities hold data only and contain no game logic.

**entity key**
: The unique name you give an entity in its YAML file, for example `heroActor` or `attackAction`. Other entities point to it by this key. Always write the clean key like `name`; the underscore forms such as `name_` are internal and never used in campaign files.

**fog of war**
: The unexplored darkness over the map that lifts as actors discover tiles within their [perception](#terms) range. Tiles marked as ghosts stay hidden.

**KeyRef**
: A field whose value is the key of another entity, forming a typed reference. For example an actor's `sprite` field is a KeyRef to a [Sprite](/reference/entities.html#sprite) entity, resolved at load time.

**KeyRefArray**
: A field holding a list of entity keys, that is, an array of [KeyRefs](#terms). An actor's `skills: ["attackSkill", "healSkill"]` is a KeyRefArray.

**Main**
: The single required entity that defines a campaign's starting state: the starting `map` (required) and, optionally, the player's `actor`. Every campaign has exactly one Main entity.

**[Measure](/reference/entities.html#measure)**
: A value computed from a [dice expression](#terms) every time it is read. Most measures are derived stats, such as `$level*5+20` for maximum health; a measure with a die in it, like `1d20`, rolls fresh each read. A measure can be public, private, and gated by a `reveal` threshold.

**[Parameter](/reference/entities.html#parameter)**
: A key-and-value entity passed to an action function. An action lists its parameters as a flat array of [KeyRefs](#terms), for example `"parameters": ["attackResourceParam", "attackDamageParam"]`, with a sibling top-level `Parameter` block defining each one.

**perception**
: An actor's vision range in pixels (the radius across of its sight ellipse; half that up and down) for detecting other actors and clearing [fog of war](#terms). The example hero's `perception` is `250`: it sees 250 px to either side and 125 px up and down.

**dialog**
: A titled text panel shown to the player, defined by a [Dialog](/reference/entities.html#dialog) entity with `title` and `text`. Dialog text supports templating tokens for live actor values (see below).

**plugin**
: A nested campaign `.zip` merged into the host campaign at load. The Main entity's `plugins` field sets resolution order; earlier entries win, and the host campaign's own files always take priority.

**Repo**
: The engine's internal store of loaded entities. You never touch it directly.

**[Resource](/reference/entities.html#resource)**
: A stored whole number on an actor, such as health, mana, gold, a cooldown, or a quest step. Every actor holds every Resource in the campaign, starting at its `default`. A resource entity defines `default`, `min`, `max`, and visibility fields like `reveal`.

**salience**
: How easily other actors detect this actor; higher means more visible. The example hero's `salience` is `10`. Like perception it is a radius in pixels: another actor notices this one when its sight reaches this ellipse.

**Strategy**
: The AI controller assigned to an actor through its `strategy` field. A strategy holds an ordered list of [behaviors](#terms). Each tick, the first behavior whose goals are **not** yet met runs its action.

<a id="dialog-templating-tokens"></a>
## Dialog templating tokens

Dialog text can show live actor values with two Jinja2-style tokens. `@` reads the target actor (the one being viewed); `$` reads the caller, that is, the actor whose dialog is open.

{% raw %}
```
HP: {{@health}} / Your luck: {{$luck}}
```
{% endraw %}

Here {% raw %}`{{@health}}`{% endraw %} shows the target's `health` resource and {% raw %}`{{$luck}}`{% endraw %} shows the caller's `luck` measure. Use these tokens only inside a Dialog entity's `text` field.

## See also

- [What is Isometry](/manual/getting-started/introduction.html)
- [Your first campaign](/manual/getting-started/your-first-campaign.html)
