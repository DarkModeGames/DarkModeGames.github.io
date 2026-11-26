# Parallax backgrounds and floor decals

**Problem.** Beyond the edge of the map there is only flat colour, and you'd like a painted picture under the
ground rather than only repeated tiles.

**Pattern.** A map takes two kinds of extra art:

- `background`: a list of [`Parallax`](/reference/entities.html#parallax) layers drawn behind the world. Each
  scrolls at `effect` times the camera's speed, so a low value (far mountains) moves slowly and gives depth.
- `floor`: a list of [`Floor`](/reference/entities.html#floor) pictures drawn **under the tiles**, with their
  top-left corner at a [`Vector`](/reference/entities.html#vector). They show through wherever the tile layers
  leave cells empty, so a painted backdrop can fill the gaps of a sparse tile layer.

```yaml
Parallax:
  farHillsParallax:
    texture: assets/ground.png
    effect: 0.2
  nearMistParallax:
    texture: assets/ground.png
    effect: 0.5

Floor:
  townSquareFloor:
    location: squareCentreVector
    texture: assets/ground.png

Vector:
  townSpawnVector:
    x: 384
    y: 8
  squareCentreVector:
    x: 400
    y: 16

Map:
  townMap:
    name: Town
    tilemap: groundTilemap
    spawn: townSpawnVector
    background: [farHillsParallax, nearMistParallax]
    floor: [townSquareFloor]
```

**Entities used.** [`Floor`](/reference/entities.html#floor), [`Map`](/reference/entities.html#map), [`Parallax`](/reference/entities.html#parallax), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- List background layers from farthest to nearest, with `effect` rising as they come closer.
- A Floor's `location` is its **top-left corner**, not its centre.
- Floor pictures have no collision and don't affect where anyone can walk; the tiles decide that.
- Parallax textures should tile or be large enough to fill the screen at your camera zoom.
- For weather and screen-wide effects, see the [effects guide](/manual/guides/effects-overlays-particles.html).
