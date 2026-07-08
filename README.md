# Isometry Documentation

The source for the [Isometry](https://dark-mode-games.itch.io/isometry) documentation website.

**Live site:** https://darkmodegames.github.io/

Isometry is a data-driven isometric RPG framework by **Dark Mode Games** — build a
multiplayer game by writing YAML, not code. This repository holds the public docs: the
player guide, developer manual, reference, recipes, and updates.

## Working on the docs

The site is a static site built with **swb**. From this folder:

```bash
swb run .      # preview locally (http://localhost:8080)
swb build .    # build the static site into build/
swb deploy .   # publish the built site to GitHub Pages
```

Pages are Markdown under `manual/`, `guide/`, `reference/`, `recipes/`, `platform/`,
`updates/`, and `about/`; the layout and theme live in `templates/` and `assets/`. The
entity and action reference pages are generated, so they should be regenerated upstream
rather than edited by hand.

## Download Isometry

The engine (Linux, Windows, macOS) is available on itch.io:
https://dark-mode-games.itch.io/isometry

## License

Documentation © Dark Mode Games, licensed **CC BY-NC 4.0**.
