---
topic: css
status: done
tags: [layout, flexbox, grid, positioning, fundamentals]
---

# layout

Source: [web.dev/learn/css/layout](https://web.dev/learn/css/layout)

CSS has many ways to do layout on the horizontal axis, the vertical axis, or both. Most real designs need more than one. The old way was `<table>` elements; CSS split visual style from HTML and it grew from there ([CSS Zen Garden](http://www.csszengarden.com) was the demo that sold people on it).

## the `display` property does two things

**1. Sets the box's own outer behavior: inline or block.**

- **inline**: sits next to siblings in the inline direction, like words in a sentence. `<span>`, `<strong>` are inline by default. You can't set explicit `width`/`height`, and surrounding elements ignore block-level margin and padding.
- **block**: takes its own line and expands to fill the inline dimension (full width in horizontal writing mode). Margin on all sides is respected.

**2. Sets how the children behave.** `display: flex` makes the box block-level *and* turns its children into flex items, unlocking alignment, ordering, and flow rules. `display: grid` does the same for grid.

## flexbox: one axis

```css
.parent { display: flex; }
```

Lays children out along a single axis (row or column). By default: children sit next to each other in the inline direction, stretched to equal height in the block direction, and **don't wrap**. They squash onto one line when space runs out. Change that with `align-items`, `justify-content`, `flex-wrap`.

Children become flex items you can size individually:

```css
.item { flex: 1 0 auto; }   /* flex-grow flex-shrink flex-basis */
```

These are low-level hints telling the browser how to react to content and viewport changes. Good for responsive design. Flexbox mostly treats items as a group.

## grid: two axes

```css
.parent {
  display: grid;
  grid-template-columns: repeat(12, 1fr);   /* 12-col grid */
  gap: 1rem;
}
```

Multi-axis layout with precise placement in two dimensions. New primitives: `repeat()`, `minmax()`, and the `fr` unit (a fraction of leftover space). Place an item across rows and columns explicitly:

```css
.parent :first-child {
  grid-row: 1 / 3;      /* row 1 to start of row 3 */
  grid-column: 1 / 4;   /* col 1 to start of col 4 */
}
```

Grid gives control over placement; flexbox groups. That's the split.

## normal flow (no grid/flex)

- **`inline-block`**: inline element that flows with text but respects `width`/`height`/margin/padding like a block.
- **`float: left | right`**: pulls an element aside so text wraps around it, newspaper-style. Following elements may reflow; clear it with `clear: both` on a following element, or `display: flow-root` on the parent ([the end of the clearfix hack](https://rachelandrew.co.uk/archives/2017/01/24/the-end-of-the-clearfix-hack/)).
- **multicolumn**: flow long content across columns. `column-count: 2` splits into a fixed number; `column-width: 260px` sets a minimum width and lets the browser add/remove columns as space changes (responsive). Content flows column to column like a magazine.

## positioning

`position` changes how a box sits in the flow. Default is `static`.

- **`relative`**: nudged by `top`/`left`/etc. from its normal spot; still occupies its original space. Also becomes the containing block for any `absolute` children.
- **`absolute`**: out of flow. Positioned by `top`/`right`/`bottom`/`left` against the nearest positioned ancestor; surrounding content reflows to fill the gap it left.
- **`fixed`**: like absolute but anchored to the viewport (`<html>`); stays put while scrolling.
- **`sticky`**: hybrid. Honors normal flow until the viewport scrolls to its offset, then sticks at the `top`/etc. value you set.

## gotchas

- **out of flow** means positioned by coordinates against a containing block, not by sibling order. Adding `top`/`left` alone doesn't pull a box out of flow; `position: absolute` (or `fixed`) does.
- **flexbox and grid don't wrap by default.** Opt in with `flex-wrap: wrap` or `repeat(auto-fit, ...)`.
- `display` decides inline/block/none and how children behave, not scrolling (that's `overflow`).

Next: [Flexbox](https://web.dev/learn/css/flexbox) and [Grid](https://web.dev/learn/css/grid) in depth.
