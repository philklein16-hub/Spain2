# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A static HTML travel itinerary website for Phil & Sarah Klein's 20th anniversary trip to Spain (July 26 – August 9, 2026). Pure HTML/CSS/JS — no build toolchain, no package manager, no framework.

## Viewing / Development

Open any `.html` file directly in a browser. There is no dev server or build step. The site is deployed via GitHub Pages with Jekyll (`.github/workflows/jekyll-docker.yml` runs `jekyll build --future` on push to `main`).

To preview locally with Jekyll:
```bash
docker run -v $(pwd):/srv/jekyll -v $(pwd)/_site:/srv/jekyll/_site \
  jekyll/builder:latest /bin/bash -c "chmod -R 777 /srv/jekyll && jekyll build --future"
```

## Architecture

### Files
- `index.html` — Homepage: hero, city-card grid, journey stats, quick links
- `map.html` — Interactive Leaflet.js map (v1.9.4 via CDN) with all trip locations
- `styles.css` — Shared stylesheet imported by all pages
- City detail pages (`madrid.html`, `toledo.html`, `granada.html`, `sevilla.html`, `barcelona.html`, `girona.html`) and utility pages (`flights.html`, `bookings.html`) are referenced in the nav but may not yet exist

### CSS Architecture
`styles.css` defines the shared design system used across all city pages. Each page may also contain a page-specific `<style>` block in `<head>` for hero backgrounds and unique per-page layout.

**CSS custom properties (defined in `:root`):**
| Variable | Color | Usage |
|---|---|---|
| `--navy` / `--navy-light` | `#1a1f2e` / `#252c3f` | Backgrounds, headers |
| `--cream` / `--cream-dark` | `#f5f0e8` / `#ede7d8` | Page backgrounds |
| `--gold` / `--gold-light` | `#c9954a` / `#d4a96a` | Accents, borders, tags |
| `--burgundy` | `#7a1c2e` | Dining, links |
| `--slate` | `#3a4a5c` | Transit elements |
| `--confirmed-green` | `#2e6b3a` | Confirmed bookings |
| `--alert-red` | `#8b1a2e` | Action-required alerts |

### Activity Card System
City detail pages use these CSS classes on `.activity` elements:
- `.activity.confirmed` — green left border (confirmed/paid bookings)
- `.activity.alert` — red left border (action required)
- `.activity.anniversary` — gold left border (special moments)
- `.activity.dining` — burgundy left border (restaurants)
- `.activity.transit-info` — slate left border (transport)

Tags inside activity cards use `.activity-tag` plus a color modifier: `.tag-confirmed`, `.tag-alert`, `.tag-anniversary`, `.tag-dining`, `.tag-info`, `.tag-michelin`.

### Map Data Structure (`map.html`)
All trip locations live in `const cities` — a plain JS object keyed by city slug (`madrid`, `toledo`, `granada`, `sevilla`, `barcelona`, `girona`). Each city has:
```js
{
  name, dates, hotel,
  center: [lat, lng],
  zoom: number,
  locations: [{ type, name, coords, address, meta, notes, conf? }]
}
```

Location `type` values and their marker/tag colors:
| Type | Color | Meaning |
|---|---|---|
| `hotel` | gold | Accommodation |
| `confirmed` | confirmed-green | Paid/confirmed activity |
| `restaurant` | burgundy | Dining |
| `site` | navy | Attraction/museum |
| `experience` | purple `#6b4ba0` | Unique experiences |
| `transit` | slate | Transport hubs |

The map tab bar, marker icons, popup tags, and legend all derive their colors from these types — keep them consistent when adding locations.

### City Hero Images
City hero backgrounds are set via CSS classes like `.city-img-madrid`, `.city-img-toledo`, etc. Each uses a three-layer `background-image` with an Unsplash URL as the middle layer plus a gradient overlay and a CSS fallback gradient.

## Key Conventions

- **No duplication of CSS variables**: the color palette is defined once in `styles.css` `:root`. `map.html` duplicates these locally in its own `<style>` block (since it doesn't use the same layout) — keep both in sync if colors change.
- **Booking confirmation codes** appear as `.booking-ref` (inline monospace navy/gold pill) in city pages and as `.conf-code` in map popups — same visual treatment, different class names.
- **All map location data** is maintained in `map.html`'s `const cities` object. When adding a new location, add it there; do not create separate data files.
- **Leaflet.js** is loaded from CDN with SRI integrity hashes — do not change the version without updating the hashes.
- **Deep-linking** on `map.html` works via URL hash (`map.html#barcelona`). The `hashchange` event re-renders the city.
