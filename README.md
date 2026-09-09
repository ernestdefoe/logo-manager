# Logo Manager

**Everything Flarum's logo setting will not do.** Upload an SVG. Make the logo
bigger and let the header grow with it. Centre it. Recolour it, pixelate it,
give it a shadow. Put a santa hat on it in December and take it off again in
January, without touching anything.

MIT licensed. No core files patched, and **no JavaScript loaded on the forum** —
the whole appearance is one stylesheet composed on the server and inlined into
`<head>`, so the header is already the right size in the first paint.

---

## Why this exists

Flarum's logo is one upload field, and it does three things people keep
running into:

| What you hit | Where it comes from |
| --- | --- |
| "My SVG won't upload" | `AbstractImageValidator::getAllowedTypes()` allows `jpeg, jpg, png, bmp, gif, webp`. SVG is rejected twice — once by the MIME allow-list, once by `getimagesizefromstring()`. |
| "My logo looks blurry when I make it bigger" | `UploadLogoController::makeImage()` re-encodes every upload to WebP **scaled to `height: 60`**. The big file you uploaded is not the file being served. |
| "I can't make the logo bigger" | `.Header-logo { max-height: 30px }` and `--header-height: 52px` are in core's compiled CSS with no setting attached to either. |

The usual workaround for the first one is to edit `logo_path` in the database
by hand — which works right up until someone uses the admin uploader again.

## What it does

**Files**
- SVG uploads, sanitised (see below), plus PNG, JPEG, WebP and animated GIF.
- Light and dark-mode logos.
- Raster logos are kept at up to 600px tall instead of core's 60, so they are
  still sharp when displayed large on a 2× screen.

**Size and position**
- Logo height, and a separate height for the phone drawer.
- Header height: grows with the logo automatically, or set it yourself.
- Left or centred alignment. Centring puts the navigation to the left of the
  logo and the account controls to the right, in flow — so a crowded header
  pushes the logo off-centre rather than covering links with it.
- An optional background plate behind the logo, with padding and corner radius.

**Effects** — applied live in the browser, so they work on an SVG and a
photograph alike and nothing is baked into the file.
- Pixelate, grayscale, sepia, invert, saturation, brightness, contrast, hue
  rotate, blur, opacity.
- Flat recolour: keeps the logo's shape and fills it with one colour. This is
  the quick fix for a dark logo on a dark header, with no second file.
- Drop shadow / glow, and a hover behaviour (lift, grow, glow, spin, wobble).
- Six presets to start from.

**Seasons**
- Date ranges (`12-01` → `12-26`), including ranges that wrap the new year.
- Moving dates: Easter, US and Canadian Thanksgiving, Lunar New Year, Mother's
  and Father's Day — with a window of days either side.
- Each season can have its own logo, its own effects, an ornament pinned to a
  corner of the logo, and weather drifting across the header.
- 16 ornaments and 8 weather types, all drawn in code — the extension ships no
  image files, so there is nothing to 404 and nothing to publish.
- Rules are an ordered list you drag to reorder, and the first match wins — so
  Christmas Day can sit above a broad December rule and take precedence for a
  day without either rule knowing about the other.
- 14 starting points included, from Christmas to Pride Month to your forum's
  own birthday.

Anything not in that list is a date range away: the schedule does not need to
know what a holiday is called.

## Installation

```bash
composer require ernestdefoe/logo-manager
php flarum cache:clear
```

Then **Administration → Logo Manager**.

## About SVG uploads

Core refuses SVG for a real reason, and accepting it means taking that job on.
An SVG served from your own domain is a document on your own origin: it can
carry `<script>`, an `onload=` attribute, a `javascript:` link or a
`<foreignObject>` full of HTML. None of that runs in the `<img>` that draws
your header — but the file also sits at a plain URL under `/assets`, and
anything that gets somebody to open that URL directly is running script as
your forum.

So every uploaded SVG is parsed and stripped of everything outside a strict
allow-list before it is written to disk, using
[`enshrined/svg-sanitize`](https://github.com/darylldoyle/svg-sanitizer), with
remote references removed as well — a logo has no business fetching anything,
and a logo that phones home on every page view is both a privacy leak and a
way for its contents to change later.

Uploads are admin-only, capped at 512 KB for SVG and 8 MB for raster, and
raster dimensions are checked from the file header before anything decodes
them, so a small file declaring enormous dimensions cannot exhaust memory.

## Notes for theme authors

Logo Manager stays out of the way until it is configured. With the default
sizes it emits no header rules at all, so installing it does not change a
themed header. Once an administrator actually sets a size, it takes ownership
of `--header-height` and the header's own height together — winning one and
losing the other is what slides page content underneath a fixed header.

Its rules carry one extra unit of specificity so that the outcome does not
depend on which extension's `<style>` block the server emitted last. Your own
custom CSS can still override any of it with `!important`.

Weather animation is disabled under `prefers-reduced-motion: reduce`; the
particles stay, they just stop moving.

## Requirements

- Flarum 2.0+
- PHP 8.3+

## Licence

MIT.
