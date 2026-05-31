---
topic: css
status: done
tags: [nesting, specificity, selectors]
---

# nesting and specificity

## native CSS nesting

Nest rules inside rules now. No Sass, no preprocessor. A child selector inside a parent block scopes to that parent.

```css
.card {
  padding: 1rem;

  p {
    color: gray;
  }

  &:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  }
}
```

The nested `p` compiles to `.card p`. A bare nested selector gets an implicit `&` (descendant) in front of it, so `.card { p {} }` is exactly `.card p {}`.

## the `&` selector

`&` is the current parent selector. Use it for pseudo-classes and compound selectors.

The space matters, and this is the one I tripped on:

```css
.btn {
  &:hover { }    /* the button itself, hovered */
  & :hover { }   /* a hovered descendant of the button */
}
```

`&:hover` (no space) is the element itself. `& :hover` (with space) is a descendant. Same rule as any combinator, but easy to miss when it's glued to the `&`.

`&` also lets you reference the parent from the other direction, to flip context:

```css
.card {
  background: white;

  .dark-theme & {
    background: black;
  }
}
```

That compiles to `.dark-theme .card`. Handy for theming without leaving the `.card` block.

## `:is()`

Takes a selector list, matches if any of them match. Cuts repetition.

```css
:is(h1, h2, h3) a { color: blue; }
```

Instead of writing the `a` rule three times.

Gotcha worth stating: `:is()` takes the specificity of its most specific argument. `:is(#id, p)` is as specific as an id, even when it's the `p` branch that matched. `:where()` is the same matching behavior but always zero specificity.

## specificity, most specific wins

When multiple rules set the same property on the same element, the highest specificity wins. Source order doesn't matter unless specificity ties.

Rough ranking, high to low:

- inline style
- id
- class / attribute / pseudo-class
- type / element

```css
.foo { color: red; }
#bar { color: green; }
```

```html
<p class="foo" id="bar">text</p>
```

Text is green. `#bar` is an id, `.foo` is a class, so `#bar` wins even though `.foo` comes later in the file. Order only breaks ties between equal specificity, and then the last one wins.
