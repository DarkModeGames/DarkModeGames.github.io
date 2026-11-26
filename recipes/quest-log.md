# Show a quest in the Quests page

**Problem.** Your quest already lives in a step resource (see [Quest steps](/recipes/quest-steps.html)). Players
need to see it in the Quests page (**J**): its name, what to do now, and the steps already done.

**Pattern.** Add one `Quest` that names the step resource, and one `Step` per stage. A step shows while the
resource is at its `value` or past it, until the next step's value is reached. The step with `done: true` moves
the quest to Completed. Nothing else changes: your NPCs and kill counts keep setting the same resource.

{% raw %}
```yaml
Resource:
  wolfHuntStepResource:
    name: The Wolf Hunt
    default: 0
    min: '0'
    max: '3'
    reveal: 999999
  wolfKillsResource:
    name: Wolves Killed
    default: 0
    min: '0'
    max: '3'
    reveal: 999999

Quest:
  wolfHuntQuest:
    name: The Wolf Hunt
    description: Old Bren's sheep keep going missing.
    resource: wolfHuntStepResource
    order: 1
    steps: [wolfHuntKillStep, wolfHuntReturnStep, wolfHuntDoneStep]

Step:
  wolfHuntKillStep:
    value: 1
    text: 'Kill three wolves on the moor. ({{$wolfKillsResource}} of 3)'
  wolfHuntReturnStep:
    value: 2
    text: Go back to Bren in the village.
  wolfHuntDoneStep:
    value: 3
    done: true
    text: Bren paid you. The sheep are safe, for now.
```
{% endraw %}

**Entities used.** [`Quest`](/reference/entities.html#quest), [`Step`](/reference/entities.html#step), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- 0 means "not started": the quest isn't listed until something sets the resource to 1 or more.
- Steps are matched by value, not list order. Leave gaps (10, 20, 30) if you may add stages later.
- `{% raw %}{{$key}}{% endraw %}` in a step's text shows the player's own value, so a kill count updates live.
- Give the step resource `reveal: 999999` so it stays out of the inventory; the Quests page shows it instead.
- `order` sorts the list (lowest first, then by name). Main story quests can use a low number.
