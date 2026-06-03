---
topic: css
status: done
tags: [specificity, selectors, cascade]
---

# specificity

Specificity is three numbers (A, B, C), compared left to right like columns. A higher A beats any B. A higher B beats any C. C only matters when A and B tie.

- A = id selectors. `#nav`
- B = classes, attributes, pseudo-classes. `.btn`, `[type="text"]`, `:hover`
- C = type selectors and pseudo-elements. `div`, `::before`

So `#nav` (1,0,0) beats `.menu .item .link a` (0,3,1). Three classes and a type can't out-rank one id. The columns don't carry: ten classes is (0,10,0), still loses to one id at (1,0,0).

Worked examples:

- `a` -> (0,0,1)
- `.btn` -> (0,1,0)
- `nav a.active` -> (0,1,2)
- `#header .nav a:hover` -> (1,2,1)

Inline style sits above all of these. `!important` is a separate axis the cascade handles, not part of the A,B,C score. The universal selector `*` and combinators (`>`, `+`, `~`, the descendant space) add nothing.

## not all selectors add specificity

Some selectors are structural and contribute zero. The `:not()`, `:is()`, `:where()`, `:has()` pseudo-classes themselves add nothing. What's inside the parens counts, except for `:where`.

- `:not(p)` adds the specificity of `p`, which is (0,0,1). The `:not()` wrapper is free.
- `:is(...)` takes the specificity of its most specific argument. `:is(#id, p)` scores as an id (1,0,0) even when the `p` branch is what matched. This bites you: one heavy argument drags the whole list up.
- `:where(...)` is always zero. Same matching as `:is()`, but it contributes (0,0,0) no matter what's inside. Use it for low-specificity defaults you want easy to override, like in design systems.

```css
:is(h1, #title) span { }   /* scores (1,0,1) because of #title */
:where(h1, #title) span { } /* scores (0,0,1), the #title is ignored */
```

## origin and specificity relationship

Specificity only decides between rules from the same origin and importance level. The cascade checks origin and importance first, and only drops to specificity when those tie. A user-agent rule with high specificity still loses to a low-specificity author rule, because author origin outranks user-agent before specificity is ever compared. Same with `!important`: an `!important` author rule beats any normal author rule regardless of specificity. Specificity is the tiebreaker within a layer, not a global score.

See [[cascade]] for the full origin and importance ordering, and [[nesting-and-specificity]] for how `:is()` behaves under native nesting.
