# CLAUDE.md

Context for working on this repo. Read before making changes.

## What this is

A single-page photography portfolio for Dominic Lucaciu. Live at
**dominiclucaciu.com**, hosted on **Netlify**. Deploys automatically on every
push to `main` — there is **no build step**, files are served as-is.

The stack is deliberately minimal: one HTML file with inline CSS and vanilla JS.
**No framework, no bundler, no dependencies, no npm.** Keep it that way — do not
introduce a build tool, package manager, or JS framework unless explicitly asked.

## File structure

```
index.html              the entire site — HTML, CSS, JS all inline
admin/index.html        Sveltia CMS entry point (loads from unpkg)
admin/config.yml        CMS schema — field names here must match the JSON shape
content/series.json     array of series slugs the site loads, e.g. ["street","travel","portraits"]
content/series/*.json   one file per series, written by the CMS
content/about.json      About-page photo + bio, edited via the CMS "Pages" collection
images/                 photos uploaded via the CMS
favicon.svg             source logo mark (monochrome, rounded-square)
favicon-16/32.png       raster favicon fallbacks
apple-touch-icon.png    180px iOS home-screen icon
SETUP.md                end-user guide for the GitHub + Netlify + CMS setup
```

The PNG icons were rasterized from `favicon.svg` on macOS with no extra tools:
`qlmanage -t -s 1024 -o <dir> favicon.svg` then `sips -z <n> <n>` to downscale.
Re-run that if the logo changes. The social-share image (`og:image`/`twitter:image`
in `<head>`) points at the **About-page photo**; those tags are static HTML, so if
the About photo changes in the CMS the URL must be updated in `index.html` by hand.

## How content works

1. On load, the site fetches `content/series.json` — just a list of slugs.
2. For each slug it fetches `content/series/<slug>.json`.
3. Each series file has `title`, `slug`, `order`, and a `photos` array. Each photo
   has `image`, `caption`, and `featured`.
4. Navigation builds itself from the series files, sorted by `order`.
5. The homepage grid is composed from every photo marked `featured: true`,
   interleaved across series (round-robin) so one series doesn't clump.

**Adding a new series is two steps:** the CMS creates
`content/series/<slug>.json`, AND the slug must be added to `content/series.json`.
Miss the second step and the series won't appear.

Series are surfaced in the UI as **projects** (that's just the label — the data
is still `content/series/*.json`; the CMS collection is labelled "Projects" but
its folder is unchanged).

The **About page** is CMS-managed too: `content/about.json` holds `image` (photo)
and `bio` (paragraphs separated by blank lines), edited via the "Pages → About"
file collection. `index.html` fetches it at load and falls back to the hardcoded
`ABOUT` array / no photo if the file is missing or a field is empty.

## Navigation, routes & lightbox

- **Nav is `Work` + `About`.** Hovering/focusing `Work` opens a preview panel:
  one column per project (title + a small thumbnail **carousel** with prev/next
  arrows). Clicking `Work` goes to the home view-all wall. On mobile the panel
  expands inside the hamburger menu (arrows hidden, strip swipe-scrolls).
- **Routes** (hash-based): `#/` home wall · `#/project/<slug>` a project grid ·
  `#/about`. On a project page the header logo becomes a back arrow + the project
  title.
- **Lightbox operates on a "sequence"** (`{src, cap, key, title}[]`), not on a
  single folder. Clicking a photo on the **home** wall flips through the whole
  interleaved featured sequence; clicking inside a **project** flips through that
  project. The caption shows the photo's project as a link into `#/project/<slug>`.

## Content vs code — where changes belong

- The `content/**` JSON files are **written by the CMS** at dominiclucaciu.com/admin.
  Do **not** hand-edit them or restructure their shape without saying so first.
- The photo/series field names (`image`, `caption`, `featured`, `title`, `slug`,
  `order`) are mirrored in `admin/config.yml`. **Changing a field name means
  changing `config.yml` to match, or the CMS breaks.**
- If unsure whether a request is a content change (belongs in the CMS) or a code
  change (belongs in `index.html`), **ask.**

## Design constraints — do not drift

Arrived at over a lot of iteration:

- **Type:** Inter only. 16px, 24px line-height, 0.7px letter-spacing. Regular
  weight everywhere except the logo, which is medium (500).
- **No uppercase text. No wide letter-spacing. No bold headings.**
- **Monochrome.** Near-black `#161616` for text, `#8f8f8f` for secondary. No
  accent colours.
- **12px padding** from the viewport edge, everywhere, including the header — the
  nav aligns with the content. (`--pad: 12px`)
- **8px** border radius on images, **8px** gaps in the grid. (`--radius`, `--gap`)
- **Three-column masonry** via CSS `column-count`: 3 columns, → 2 under 900px,
  → 1 under 600px. Photos keep their own aspect ratio.
- **Motion is opacity-only**, 180–300ms, on `cubic-bezier(0.4, 0, 0.2, 1)`
  (`--ease`). No scaling, no sliding, no scroll-triggered reveals. The lightbox
  crossfades between two stacked `<img>` layers so there is never a blank frame.
  - **Deliberate exceptions (leave them — both explicitly requested):** the
    Work-menu thumbnail carousel slides horizontally (`translateX`, 260ms; can be
    a crossfade instead); and the mobile lightbox has **pinch / double-tap / pan
    zoom** (`transform: scale` on `.stage`), which is user-driven, not decorative.
- **Copy:** never mention camera gear, film stock, or specific cities other than
  London.

Reference points: **zegzulka.com** and **works.pm** — restrained, image-first,
the design gets out of the way.

## Workflow

When asked for a change:

1. Make it.
2. Show what changed before committing.
3. Commit with a clear message and push to `main`.

Netlify picks it up from there.

## Running locally

`fetch()` of the content JSON needs a server — opening `index.html` over
`file://` fails (browsers block `fetch` on `file://`). Serve the folder root over
`http://`, e.g.:

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000. **Caching gotcha:** the plain server doesn't
send no-cache headers, so the browser may keep serving a stale `index.html` and
your edits won't show — hard-reload (⌘⇧R) or run a no-cache server. Only a
local-dev concern; Netlify handles cache correctly per deploy.

## Known issues / fragilities

- **Secondary text contrast:** `--muted: #8f8f8f` on white is ~2.5:1, below the
  WCAG AA threshold. Darkening toward `#767676` would fix legibility while keeping
  the monochrome look — not yet done (flagged, awaiting a design call).
- **First-load layout shift:** grid figures reserve space from a per-session
  aspect-ratio cache (`sessionStorage`), so navigation/return visits don't reflow.
  The very first view of an image still settles once, because the CMS image widget
  doesn't record dimensions (would need a manual field to fully fix).
