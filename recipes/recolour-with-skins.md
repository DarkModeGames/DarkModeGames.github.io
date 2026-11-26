# Recolour one sprite for many actors

**Problem.** You want a grey wolf, a white wolf and a black wolf, but have only one wolf sprite sheet.

**Pattern.** A [`Skin`](/reference/entities.html#skin) recolours a sprite when it is drawn. It is a list of
[`Mask`](/reference/entities.html#mask)s, each swapping one exact colour for another. Every wolf actor uses the same
[`Sprite`](/reference/entities.html#sprite) and picks a different `skin`.

```yaml
Mask:
  greyFurToWhiteMask:
    from_color: '#7A7A7A'
    to_color: '#E8E8E8'
  darkFurToWhiteMask:
    from_color: '#4A4A4A'
    to_color: '#B8B8C0'
  greyFurToBlackMask:
    from_color: '#7A7A7A'
    to_color: '#2A2A2E'
  darkFurToBlackMask:
    from_color: '#4A4A4A'
    to_color: '#141416'

Skin:
  whiteWolfSkin:
    title: White Wolf
    masks: [greyFurToWhiteMask, darkFurToWhiteMask]
  blackWolfSkin:
    title: Black Wolf
    masks: [greyFurToBlackMask, darkFurToBlackMask]

Actor:
  greyWolfActor:
    name: Grey Wolf
    sprite: wolfSprite
  whiteWolfActor:
    name: White Wolf
    sprite: wolfSprite
    skin: whiteWolfSkin
  blackWolfActor:
    name: Black Wolf
    sprite: wolfSprite
    skin: blackWolfSkin
```

**Entities used.** [`Actor`](/reference/entities.html#actor), [`Mask`](/reference/entities.html#mask), [`Skin`](/reference/entities.html#skin)

**Gotchas.**

- Colours must match the sheet's pixels. Pick them from the image, and recolour each shade you care about.
- A Skin holds up to 16 Masks.
- The skin is part of the [Actor](/reference/entities.html#actor). To change colour mid-game, `change_actor_self` into an actor with another skin.
- Pixel art with few colours recolours well. Smoothly shaded art needs many masks.
