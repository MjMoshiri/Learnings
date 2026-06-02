---
topic: css
status: done
tags: [cascade, fundamentals]
---

# cascade

The algorithm that picks a winner when several rules set the same property on one element. Four stages, checked in order:

1. **Importance** — rule type weight. Roughly: normal < `animation` < `!important` < `transition`. Active transitions beat even `!important`.
2. **Origin** — where the rule comes from. User-agent < local user < author < author `!important` < user `!important` < user-agent `!important`.
3. **Specificity** — how precisely the selector matches. `id` > class > element, and points add up. Keep selectors short so they stay overridable.
4. **Position** — last one declared wins. Only used as the tiebreaker when everything above is equal.

```css
button { color: red; }
button { color: blue; }   /* blue wins on position */
```

Handy side effect: declare a property twice for fallbacks. A browser ignores values it can't parse, so the later valid one wins and the unsupported one is silently dropped.

```css
.x {
  font-size: 1.5rem;                          /* fallback */
  font-size: clamp(1.5rem, 1rem + 3vw, 2rem); /* used if supported */
}
```
