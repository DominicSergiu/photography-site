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
images/                 photos uploaded via the CMS
SETUP.md                end-user guide for the GitHub + Netlify + CMS setup
```

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
`file://` fails. Serve the folder root with anything static, e.g.:

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000. (The site itself suggests `npx serve` in its
load-error message, but python3 needs nothing installed.)

## Known issues / fragilities

- **`index.html` grid hover scales the image** (`transform: scale(1.02)`, 0.45s).
  This contradicts the opacity-only / no-scaling / 180–300ms motion rule above.
  Flagged, not yet changed.
- Captions and titles are injected into HTML via template strings without
  escaping (`alt="${p.cap}"`, chip text). A caption containing `"` or `<` would
  break markup. Content is CMS-authored so low-risk, but worth knowing.
- The footer/about **Instagram link points to `https://instagram.com`**, not a
  real profile.
