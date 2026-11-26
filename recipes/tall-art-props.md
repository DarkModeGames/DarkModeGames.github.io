# Trees, walls and other tall art

**Problem.** A tree drawn into the tile layer looks wrong: the player walks "on top of" it instead of behind it,
and nothing stops them walking through the trunk.

**Pattern.** Tiles are for flat ground. Anything tall is an **actor**, so it is depth-sorted with the characters:
a static actor with `targetable: false`, so it is never picked by clicks, Tab or monsters. Actors don't block each
other, so put a **ghost obstacle tile** under the trunk. It blocks movement but stays invisible and doesn't block
sight.

```yaml
Face:
  roofFace:
    northeast: true
    northwest: true

Tile:
  grassTile:
    symbol: g
    index: 0
    navigation: roofFace
  # Same picture as grass, but nobody can walk on it, and it doesn't block sight.
  trunkTile:
    symbol: t
    index: 0
    obstacle: roofFace
    ghost: true

TileSet:
  groundTileset:
    columns: 1
    texture: assets/ground.png
    tiles: [grassTile, trunkTile]

Actor:
  oakTreeActor:
    name: Oak
    sprite: oakTreeSprite
    targetable: false
    base: 8
    salience: 260

Vector:
  oakVector:
    x: 400
    y: 24

Deployment:
  oakDeployment:
    actor: oakTreeActor
    location: oakVector
```

In the layer grid, write `t` in the cell under the trunk instead of `g`.

**Entities used.** [`Actor`](/reference/entities.html#actor), [`Deployment`](/reference/entities.html#deployment), [`Face`](/reference/entities.html#face), [`Tile`](/reference/entities.html#tile), [`TileSet`](/reference/entities.html#tileset), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- Give props a `salience`. It is how far away they can be seen, so with too small a value the prop pops in late.
- Depth sorting uses one point per actor (its feet). Split a long wall into short pieces, one actor each, so the
  player sorts correctly against every piece.
- Ghost tiles under walls stop walking; an `obstacle` face without `ghost` would also block line of sight.
- Area effects still reach `targetable: false` actors. Filter area effects by tags (see
  [Tags and area filters](/recipes/tags-and-area-filters.html)).
- Many hundreds of props are fine, but obstacle tiles cost load time. Put them only along the edges of solid
  areas.
