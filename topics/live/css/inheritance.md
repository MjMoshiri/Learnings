---
topic: css
status: done
tags: [inheritance, fundamentals]
---

# inheritance

## what inherits

Some properties pass from parent to child on their own. Mostly text stuff: `color`, `font-family`, `font-size`, `line-height`, `letter-spacing`, `list-style`. Box stuff doesn't: `width`, `margin`, `padding`, `border`, `background`.

Rule of thumb: if it'd be weird for every child to copy it (a border on everything), it doesn't inherit. Set `font-family` on `body` once and the whole page picks it up. That's inheritance doing the work.

## the five keywords

Every property accepts these, whether or not it inherits by default.

- `inherit`: take the parent's computed value, explicitly. Forces inheritance even on properties that don't inherit, like `border: inherit`.
- `initial`: reset to the property's spec default. Ignores both the parent and the browser's user agent stylesheet. `display: initial` gives `inline`, since that's the spec initial value, even on a div.
- `unset`: smart reset. Acts like `inherit` if the property normally inherits, like `initial` if it doesn't.
- `revert`: roll back to what the user agent stylesheet (or user styles) would have applied, as if your CSS never set it. Not the same as `initial`: `display: revert` on a div gives `block` (UA default), while `initial` gives `inline` (spec default).
- `revert-layer`: same idea, scoped to cascade layers (`@layer`). Rolls back to the value from the previous layer instead of all the way to the UA stylesheet. Handy when you stack layers and want to undo only the current one's override.

The `display` case shows the gap between `initial` and `revert`:

```css
div.a { display: initial; } /* inline: spec default */
div.b { display: revert; }  /* block: UA stylesheet default */
```

`revert-layer` rolls back one layer at a time:

```css
@layer base {
  p { color: blue; }
}
@layer theme {
  p { color: red; }
  p.plain { color: revert-layer; } /* falls back to blue from base */
}
```

UA stylesheet basics live in box-model.md.
