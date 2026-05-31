---
topic: css
status: done
tags: [layout, fundamentals]
---

# box model

## everything is a box

Every element the browser renders is a rectangular box. Block boxes and inline boxes behave differently, but they're both boxes. Layout is just the browser sizing and positioning these boxes.

## the four layers

From inside out: content, padding, border, margin.

- **content**: the actual text or image. Sized by `width`/`height`.
- **padding**: space between content and border. Sits inside the background.
- **border**: the line around the padding.
- **margin**: transparent space outside the border. Pushes other boxes away.

Watch out: margins collapse vertically between block boxes. Two stacked blocks with `margin: 20px` give you 20px of gap, not 40px. The larger of the two wins.

## extrinsic vs intrinsic sizing

**Extrinsic**: you set the size explicitly. The box takes the size you gave it and ignores its content, so content can overflow.

**Intrinsic**: the box sizes itself to fit its content. This is the default for the height of block elements, and what you get with `max-content`, `min-content`, or `fit-content`.

```css
.extrinsic { width: 300px; }          /* fixed, content may overflow */
.intrinsic { width: max-content; }    /* shrinks/grows to fit content */
```

## box-sizing

`content-box` is the CSS default. `width`/`height` apply to the content box only; padding and border get added on top.

```css
.content-box {
  box-sizing: content-box;            /* default */
  width: 200px;
  padding: 20px;
  border: 5px solid;
}
/* renders 250px wide: 200 + 20*2 + 5*2 */

.border-box {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid;
}
/* renders 200px wide total; content shrinks to fit */
```

`content-box` makes layout math annoying because the rendered width isn't the width you typed. With `border-box`, `width` means the actual rendered width and padding/border eat into it. This is why most projects set it on everything at the top of the stylesheet:

```css
*, *::before, *::after { box-sizing: border-box; }
```

## user agent stylesheet

The browser ships its own default stylesheet, the user agent stylesheet. It's why an unstyled `<h1>` is big and bold, `<ul>` has bullets and indent, `body` has an 8px margin, and links are blue and underlined. Your CSS layers on top of it.

This is why CSS resets and normalize.css exist: wipe or even out those defaults so you start from a known baseline. The default `box-sizing: content-box` comes from here too.
