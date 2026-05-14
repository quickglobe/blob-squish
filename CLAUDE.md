# Blob Squish — Project Context for Claude Code

## What this is
A static single-page web app — a blob-squishing clicker game with evolution mechanics. No build process, no framework, no dependencies. Pure HTML, CSS, and vanilla JS.

## File structure
```
index.html          # The single production file — the one that gets deployed
404.html            # Custom 404 page
favicon.svg         # Browser tab favicon (SVG, scalable)
favicon-32.png      # Browser tab favicon fallback (32×32 PNG)
apple-touch-icon.png # iOS bookmark/home screen icon (180×180 PNG)
icon-192.png        # Android/PWA manifest icon (192×192 PNG, transparent)
icon-512.png        # Android/PWA manifest icon (512×512 PNG, transparent)
og-image.png        # Social share preview image (1200×630 PNG)
site.webmanifest    # PWA manifest
robots.txt          # Search crawler rules
netlify.toml        # Netlify build config (static, no build command)
CLAUDE.md           # This file
```

## Debug panel
`index.html` includes a built-in debug panel (stage picker + generation buttons) hidden behind a **Developer mode** toggle in the info panel (the `i` button, top-left). There is no separate development file.

## Deployment
- Hosted at **https://blob-squish.com** via Netlify
- Netlify is connected to this GitHub repo and auto-deploys on push to `main`
- Netlify is configured with **Auto Publishing Locked** — pushes deploy to preview, publishing to production is manual
- No build step — Netlify serves files directly from repo root

## Game mechanics overview
- **Click/tap** the blob (or press spacebar) to squish it
- **Frenzy**: clicking faster than 150ms builds frenzy levels (!, !!) which speed up and intensify the squish animation
- **Growth**: blob grows 25px every 10 clicks (4 tiers per evolution cycle)
- **Evolution**: every 50 clicks, the blob morphs to a new shape with a celebration animation
- **Progress dots**: 5 dots below the counter show growth progress within the current evolution cycle
- **Generations**: after completing all 10 shapes (500 clicks), a new generation begins — shapes get pointier (×0.5 per gen), colour palette changes

## Shape system
Shapes are defined in `BLOB_SHAPE_RADII` as 16 radii from centre (every 22.5°, starting North, clockwise). The `makeBlob()` function converts these to a smooth SVG path using Catmull-Rom → cubic bezier interpolation. `mutatedRadii()` scales deviations from base radius (85) by `1 + generation × 0.5` — clamped to 38–92.

Current shapes (0–9): circle, organic blob, pear, diagonal, cross +, star ×, crescent, mushroom, alien, eight-point star.

## Colour system
`GEN_PALETTES` has 3 palettes of 6 colours each. Same 6 hue families (rose, teal, amber, sky, lavender, lime) in the same order across all gens, increasing in saturation: Gen 1 = pastels, Gen 2 = rich, Gen 3 = vibrant. `colors()` is a function (not an array) that returns `GEN_PALETTES[generation % GEN_PALETTES.length]`.

**Common mistake**: `colors.length` will throw — always use `colors().length`.

## Dark mode
Fully supported via `prefers-color-scheme: dark`. Each colour theme has a `darkPage` value. A `change` listener on the media query re-applies colour when the system preference changes. Progress dots in dark mode use white with box-shadow outline (unfilled) and solid white (filled) — opacity alone doesn't work on dark backgrounds.

## Mobile considerations
- `touch-action: manipulation` on `body` and `.blob-wrap` prevents double-tap zoom
- `-webkit-user-select: none` prevents text selection
- Blob sits inside `.blob-area` (fixed 290px height) so the counter doesn't move when the blob grows
- `overflow: hidden` on `body` prevents scrolling

## Known gotchas
- `colors` is a **function** — always call it as `colors()`, never reference it as an array
- After any `str_replace` edit, re-view the file before further edits — context goes stale
- `apple-touch-icon.png` must have a **solid** background (Apple fills transparency with black)
- `icon-192.png` and `icon-512.png` have **transparent** backgrounds (Android uses `background_color` from manifest)
- Netlify's free tier blocks unrecognised Git contributors on private repos — repo is public to avoid this
