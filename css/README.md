# ProAV site CSS

Plain CSS, no build step. Each page links the files it needs, in this order.

| File | What lives here |
|---|---|
| `tokens.css` | Colors, line colors, type families, the signal-line gradient. Change a value here and it changes site-wide. |
| `base.css` | Page reset, body, type helpers (`.display`, `.sr-only`), the section scaffold (`.section-eyebrow`, `.section-title`), `.band-plain`, scroll reveal. |
| `layout.css` | Chrome: nav, search, mobile menu, footer, corner brackets. |
| `components.css` | Buttons, chips, tiles, video cards, merch cards, image modal. |
| `home.css` | Homepage sections: hero + logo video, show, who's on, listen, watch. |
| `signal-flow.css` | Signal Flow Daily: briefing, vendor tabs, tiles. |
| `contact.css` | Contact page form only. |

Pages: `index.html` links tokens, base, layout, components, home, signal-flow. `contact.html` links tokens, base, layout, contact.

## Rules of the road
- Colors and fonts come from `tokens.css` (`var(--tungsten-amber)`, `var(--font-display)`). Don't type hex values into components.
- Signal colors are amber, white, cyan. The signal line is `var(--signal-gradient)`.
- Tiles are flat with square corners; pills (chips, tabs, buttons) stay round. Tile hover is the top-edge accent (amber, cyan on every second tile).
- Responsive rules sit at the bottom of the same file as the component, widest breakpoint first (1280, 1024, 720, 640).
- Within a file, rules are grouped by component under a `/* name */` comment. Add new rules to the matching group.
- Same selector set twice means one of them is dead. Edit the existing rule instead of appending a patch.
