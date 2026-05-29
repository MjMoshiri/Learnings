---
topic: css
status: done
tags: [selectors, fundamentals]
---

# selectors

## the basic selectors

- type / element: matches by tag name. `p { }`, `div { }`
- class: `.btn { }`
- id: `#header { }`
- attribute: `[type="text"] { }`. Drop the value to match any element that has the attribute at all: `[href]`

## attribute selector operators

Match on part of an attribute value, not the whole thing:

```css
/* href contains "example.com" */
[href*='example.com'] { color: red; }

/* href starts with https */
[href^='https'] { color: green; }

/* href ends with .com */
[href$='.com'] { color: blue; }
```

Quick reference: `*=` contains, `^=` starts with, `$=` ends with.

Case sensitivity: whether the value match is case-sensitive depends on the attribute and HTML rules, so don't assume. Force it with a flag before the closing bracket. `s` for case-sensitive, `i` for case-insensitive.

```css
[href$='.COM' s] { }   /* only matches uppercase .COM */
```

## grouping

One rule block can serve many selectors. Separate them with commas.

```css
h1, h2, h3 { margin: 0; }
```

## combinators

- descendant (space): matches an element nested anywhere inside another, any depth.

  ```css
  .top div { }   /* every div inside .top, any depth */
  ```

- child (`>`): matches direct children only, one level down.

  ```css
  .top > div { }  /* only divs that are immediate children of .top */
  ```

  The one I tripped on: `.top div` vs `.top > div`. The space is any descendant at any depth. The `>` is direct children only. Easy to mix up.

- adjacent sibling (`+`): matches the element immediately after another. Same parent, the very next one.

  ```css
  h2 + p { }   /* the p right after an h2 */
  ```

- general sibling (`~`): matches every following sibling at the same level, not just the next.

  ```css
  h2 ~ p { }   /* every p after an h2, same parent */
  ```
