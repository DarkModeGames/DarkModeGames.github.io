# Credit the people whose work you used

**Problem.** Your campaign uses icons, sounds and sprites under licences that ask for credit (CC BY, for example),
and players should be able to read who made what.

**Pattern.** Put the credits in `Main.credits` as BBCode. Players read them from the System menu's **Credits**
button. Keep a plain `CREDITS.md` beside your campaign source as well, so the licence trail is in your files too.

{% raw %}
```yaml
Main:
  campaignMain:
    map: townMap
    credits: '[b]Wolf Moor[/b][br]A campaign by You.[br][br][b]Icons[/b][br]Wolf head by Lorc, [url=https://game-icons.net]game-icons.net[/url], CC BY 3.0.[br][br][b]Sounds[/b][br]Wind loop by Someone, CC BY 4.0.'
```
{% endraw %}

**Entities used.** [`Main`](/reference/entities.html#main)

**Gotchas.**

- It's one string. Use `[br]` for line breaks, `[b]` for headings and `[url=...]` for links.
- Quote it in YAML (single quotes), because BBCode's brackets would otherwise read as a YAML list.
- Name each work, its author, where it came from and its licence. CC BY needs all four.
