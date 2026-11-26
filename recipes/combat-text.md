# Floating combat numbers

**Problem.** You want damage and healing to float up as numbers over the actor that took them, without writing an
echo into every attack.

**Pattern.** Turn on `combat_text` on the [Resource](/reference/entities.html#resource). Every change to it then floats up as a number over the actor,
in `gain_color` when it rises and `loss_color` when it falls. Players see it on their own character, on actors
they hurt or healed, and on actors they are targeting.

```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
    combat_text: true
    gain_color: '#5ec96a'
    loss_color: '#d04040'
  manaResource:
    name: Mana
    default: 50
    min: '0'
    max: '50'
    combat_text: true
    gain_color: '#5aa0e0'
```

**Entities used.** [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- It is off by default. Turn it on only for numbers players care about. Mana ticking up every two seconds can be
  noisy.
- The colours are optional (green for gains and red for losses by default). They take a hex code or a colour
  name.
- A refill at a respawn floats too ("+100"). That is usually fine.
- For words rather than numbers, use [`echo_self`](/recipes/barks-and-feedback.html).
