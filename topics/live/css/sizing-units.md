---
topic: css
status: done
tags: [sizing, units, fundamentals]
---

# sizing units

Source: [The CSS Podcast 008 — Sizing Units](https://thecsspodcast.libsyn.com/tcp-css-podcast-episode-008) · [web.dev/learn/css/sizing](https://web.dev/learn/css/sizing)

## numbers (unitless)

A bare number's meaning depends on context. The big one: **`line-height` should be unitless** — `1.5` means 1.5× the font size and stays relative as `font-size` is inherited. `line-height: 15px` won't adapt. Also used for `opacity`, `rgb()` channels, `transform: scale()`.

## percentages

Calculated against a base that depends on the property:

- `width` → % of the parent's available width.
- `margin` / `padding` → % of the parent's **width**, even vertically (`margin-top: 50%` of a 300px parent = 150px).
- `transform: translateX(10%)` → % of the **element's own** size.

## absolute vs relative lengths

- **Absolute** (`px`, `cm`, `in`, `pt`) resolve against one fixed base — predictable, best for print. Physical units don't map reliably to screens.
- **Relative** resolve against a changing base — adaptive, ideal for the responsive web.

## the relative units worth knowing

- **`em`** — relative to the element's font size (inherited base).
- **`rem`** — relative to the root font size (default 16px). Prefer for text so it respects user font-size preferences (`px` ignores them).
- **`ch`** — width of the "0" glyph. Great for capping line length for readability: `max-width: 60ch`.
- **`vw` / `vh`** — 1% of viewport width / height. `dvh`/`svh`/`lvh` variants handle mobile browser UI showing/hiding.
- **`cqw` / `cqi`** etc. — 1% of the container's size; for container queries.

## misc

- Angles: `deg`, `rad`, `turn` (`1turn = 360deg`) for `rotate()` and `hsl()` hue.
- Resolution: `dpi` for targeting high-density screens in media queries.

Resources: [Values & Units L4](https://www.w3.org/TR/css-values-4) · [All About Ems](https://learn.scannerlicker.net/2014/07/31/so-how-much-is-an-em/) · [Percentages explainer](https://2019.wattenberger.com/blog/css-percents)
