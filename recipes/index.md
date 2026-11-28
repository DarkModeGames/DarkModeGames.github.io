# Recipes

Reusable patterns for building campaigns and plugins with Isometry data. Each recipe states
a **Problem**, the **Pattern** that solves it, a complete YAML example, the entities it uses, and the gotchas to
watch for. Every example validates on its own; sprites and the map's tiles are left for you to supply. For the
fields of each entity, see [All entity types](/reference/index.html#all-entity-types).

> Adding a recipe: drop a new `web/recipes/<slug>.md` file (the filename is the URL slug,
> the first `# heading` is the title), then add one bullet to the list below. Nothing else
> to wire up.

## Actors & stats

- [Give every resource bounds, and hide internal state](/recipes/resource-bounds-and-reveal.html) — `min`, `max` and `reveal`.
- [Set up each actor when it spawns](/recipes/spawn-hook.html) — a spawn hook, because `on_start` is for players only.
- [Tag actors and filter area effects](/recipes/tags-and-area-filters.html) — make an area spell hit only enemies.
- [Derive stats from level and gear with Measures](/recipes/derived-stats-with-measures.html) — always-current attack power.

## Characters & progression

- [Let players pick their hero](/recipes/pick-your-hero.html) — start as a bodiless camera and choose a character.
- [A starter body that picks a class](/recipes/starter-form.html) — walk around first, then choose; the choice sticks.
- [XP and level-up](/recipes/xp-and-levels.html) — a guarded Trigger chain that can't run away.
- [Talents unlocked at a level](/recipes/talents-at-levels.html) — a choice menu that puts a new skill on the hotbar.
- [Talent trees](/recipes/talent-trees.html) — points from level-ups, talents that unlock from the bottom up.
- [A character page with equipment slots](/recipes/character-page.html) — what you wear and your stats on one page.
- [Respawn a player on death](/recipes/respawn-on-death.html) — fall, a countdown, then rise at a graveyard.

## Skills & class resources

- [Give a skill a cooldown](/recipes/skill-cooldowns.html) — a countdown resource, a Timer, and a private "Not ready".
- [Skill tooltips with real numbers, and greyed-out skills](/recipes/skill-tooltips-and-readiness.html) — `{% raw %}{{range:}}{% endraw %}`, `{% raw %}{{dice:}}{% endraw %}`, `ready` and `in_range`.
- [Pace a skill: lock time and animation speed](/recipes/skill-pacing.html) — match `Action.time` and `Animation.rate`.
- [Regenerate mana, decay rage](/recipes/regen-and-decay.html) — repeating Timers for class resources.
- [Energy and combo points](/recipes/energy-and-combo-points.html) — builders and finishers.
- [Health regeneration only out of combat](/recipes/out-of-combat-regen.html) — heal after a few quiet seconds.

## Combat

- [Threat-based targeting and taunt](/recipes/threat-and-taunt.html) — monsters attack the biggest threat; tanks pull them back.
- [Auto-attack: right-click an enemy, and fight back when hit](/recipes/auto-attack.html) — `auto_interact`, `on_interact`, `reaction` and `start_auto_interact`; also for peaceful repeats (tending a crop).
- [Monsters that come back](/recipes/respawning-monsters.html) — `Deployment.respawn` and walking home.
- [Stuns and interrupts with a swing lockout](/recipes/crowd-control.html) — stop a monster attacking for a while.
- [Slows that don't stack](/recipes/slows-that-dont-stack.html) — `temp_speed_target` with a lock.
- [Shields and damage reduction with a hurt Trigger](/recipes/shields-and-mitigation.html) — wards and guard stances.

## Monsters & bosses

- [Monster AI: acquire, chase, attack, leash](/recipes/mob-ai-strategy.html) — how a Strategy picks what to do each tick.
- [Pack call for help](/recipes/pack-call-for-help.html) — idle packmates join the fight.
- [A summoner with a following legion](/recipes/summoner-legion.html) — raise minions that march in formation and fight your fights.
- [Boss phases: a half-health line, adds and an enrage clock](/recipes/boss-phases.html) — a fight that changes as it goes.
- [Scale a boss to the party](/recipes/party-scaled-boss.html) — health by the number of players present.
- [Leave a corpse for a few seconds](/recipes/monster-corpse.html) — a death pose before the body goes.
- [A safe zone monsters won't enter](/recipes/safe-zones.html) — a ward ring at graveyards and arrivals.

## Items & economy

- [Items as resources: equip, use, drop](/recipes/items-as-resources.html) — inventory menus and one weapon at a time.
- [A vendor with a gold check](/recipes/vendor-with-gold-check.html) — buy and sell from an NPC's menu.
- [Drop loot on death and let a player claim it](/recipes/loot-drop-and-pickup.html) — a bag that pays whoever walks onto it.

## Quests & NPCs

- [Quest steps and an NPC that follows them](/recipes/quest-steps.html) — a step resource, menu options and kill credit.
- [Show a quest in the Quests page](/recipes/quest-log.html) — `Quest` and `Step` over the same step resource.
- [Choices that change dialog and shops](/recipes/choices-and-favour.html) — sides, favour and branching dialog.
- [Ambient barks and failure feedback](/recipes/barks-and-feedback.html) — public and private `echo_self`, with `duration`.

## World & maps

- [Travel between maps through a gate](/recipes/map-gates.html) — a Travel / Stay question on touch and on click.
- [Fast travel with waypoints](/recipes/fast-travel-waypoints.html) — discoverable places with a travel menu.
- [Trees, walls and other tall art](/recipes/tall-art-props.html) — untargetable actors over ghost obstacle tiles.
- [Help pathfinding round corners with Nav points](/recipes/corner-pathing-with-nav.html) — doorways and tight gaps.
- [Ranges, sight and area sizes](/recipes/distances-and-areas.html) — isometric pixels, everywhere.

## Look & sound

- [Credit the people whose work you used](/recipes/campaign-credits.html) — `Main.credits`, read from the System menu.
- [Floating combat numbers](/recipes/combat-text.html) — `combat_text` on a Resource.
- [Recolour one sprite for many actors](/recipes/recolour-with-skins.html) — Skins and Masks.
- [Parallax backgrounds and floor decals](/recipes/backgrounds-and-floors.html) — scenery behind and under the map.
- [Sounds on animations and map ambience](/recipes/sounds-and-ambience.html) — one-shot and looping Sounds.
- [Screen effects and particles](/manual/guides/effects-overlays-particles.html) — rain, fog and glows (manual guide).
