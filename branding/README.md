# Branding

Logo, icon, and social preview assets for Ironbridge Health Alliance.

## Contents

- `ironbridge-icon.svg` — square icon (interlocking rings), for avatars and favicons
- `ironbridge-logo-horizontal.svg` — icon + wordmark, for README headers and letterhead-style use
- `beehiiv-thumbnail-preview.svg` — 1200x630 social preview card
- Rasterized PNGs (`*-800.png`, `*-1200x630.png`) — generated from the SVGs above for platforms that require raster uploads (Beehiiv, LinkedIn)

## Palette

- Navy `#14495A` — parent organization
- Teal `#1E8A7A` — allied/acquired site
- Neutral text `#4B5563` / `#9CA3AF`

## Regenerating PNGs

```bash
rsvg-convert -w 800 -h 800 ironbridge-icon.svg -o ironbridge-icon-800.png
rsvg-convert -w 1200 -h 630 beehiiv-thumbnail-preview.svg -o beehiiv-thumbnail-1200x630.png
```
