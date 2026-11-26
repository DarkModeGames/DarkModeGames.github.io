# Talent trees

**Problem.** Players earn a point each level and spend it in a tree of talents that unlock from the bottom up,
like a WoW talent tree.

**Pattern.** A `Tree` holds `Talent`s on a grid and names the Resource its points come from. Each talent's rank is
its own Resource, so skills, measures and conditions read it like any other value. `requires` opens a talent only
once the talents below it have a rank. `on_learn` runs on the player after each rank, for example to put a new
skill on the hotbar. The campaign grants points itself, here with a Trigger on the level resource. The actor lists
its trees in `trees`; players open them with **P**.

{% raw %}
```yaml
Resource:
  levelResource:
    name: Level
    default: 1
    min: '1'
    max: '10'
  talentPointsResource:
    name: Talent Points
    default: 0
    min: '0'
    max: '10'
    reveal: 1
  toughnessRankResource:
    name: Toughness
    default: 0
    min: '0'
    max: '3'
    reveal: 999999
  cleaveRankResource:
    name: Cleave
    default: 0
    min: '0'
    max: '1'
    reveal: 999999

Condition:
  levelThreeCondition:
    left: '$levelResource'
    operator: '>='
    right: '3'
  aboveLevelOneCondition:
    left: '$levelResource'
    operator: '>'
    right: '1'

Parameter:
  talentPointsParameter:
    key: resource
    value: talentPointsResource
  onePointParameter:
    key: expression
    value: '1'
  slotTwoParameter:
    key: slot
    value: '2'
  cleaveSkillParameter:
    key: skill
    value: cleaveSkill
  cleaveMessageParameter:
    key: message
    value: Cleave swings.

Action:
  grantTalentPointAction:
    if: aboveLevelOneCondition
    do: plus_resource_self
    parameters: [talentPointsParameter, onePointParameter]
  learnCleaveAction:
    do: set_skill_self
    parameters: [slotTwoParameter, cleaveSkillParameter]
  cleaveAction:
    name: Cleave
    time: 1.0
    do: echo
    parameters: [cleaveMessageParameter]

Trigger:
  levelUpPointTrigger:
    resource: levelResource
    action: grantTalentPointAction

Skill:
  cleaveSkill:
    name: Cleave
    description: Strike everything in front of you.
    start: cleaveAction

Talent:
  toughnessTalent:
    name: Toughness
    description: Each rank adds 10 maximum health.
    resource: toughnessRankResource
    max: 3
    column: 1
    row: 0
  cleaveTalent:
    name: Cleave
    description: Learn Cleave. Needs a point in Toughness and level 3.
    resource: cleaveRankResource
    requires: [toughnessTalent]
    if: levelThreeCondition
    on_learn: learnCleaveAction
    column: 1
    row: 1

Tree:
  warriorArmsTree:
    name: Arms
    description: Hit hard and keep standing.
    points: talentPointsResource
    talents: [toughnessTalent, cleaveTalent]

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    trees: [warriorArmsTree]
```
{% endraw %}

**Entities used.** [`Tree`](/reference/entities.html#tree), [`Talent`](/reference/entities.html#talent), [`Trigger`](/reference/entities.html#trigger), [`Condition`](/reference/entities.html#condition), [`Resource`](/reference/entities.html#resource), [`Action`](/reference/entities.html#action), [`Skill`](/reference/entities.html#skill), [`Actor`](/reference/entities.html#actor)

**Gotchas.**

- Rows count from the bottom (row 0 is the first tier). Columns count from the left.
- Give each rank resource `min: '0'` and a `max` of at least the talent's `max`, and `reveal: 999999`.
- A Trigger fires on every change of its resource, so guard it (here: only above level 1). For a full level-up
  chain, see [XP and level-up](/recipes/xp-and-levels.html).
- Read a rank anywhere with `$toughnessRankResource`, for example in a Measure:
  `100+$toughnessRankResource*10`.
- `cost` (default 1) is the points spent per rank.
- For a single choice at a level instead of a tree, see [Talents unlocked at a level](/recipes/talents-at-levels.html).
