---
topic: book-summaries
status: wip
tags: [design, ux]
---

# Chapter 5: Human Error? No, Bad Design

"Human error" is usually the last step in a chain the design set up to fail. Blaming the person who made the final move ignores every earlier point where the system could have caught it.

## Slips vs. mistakes

- Slip: the goal was right, the execution went wrong. You meant to hit "save" and hit "delete" instead, because the buttons sit next to each other. Fix with layout, spacing, confirmation.
- Mistake: the goal itself was wrong, built on a faulty read of the situation. You confidently took the wrong exit because your mental model of the highway was wrong. Fix with better information, not better buttons.

Slips happen to experts on autopilot. Mistakes happen when someone reasons their way to the wrong conclusion. A single confirmation dialog helps slips and does nothing for mistakes, since the person is confident they're right.

## Kinds of slips

- Capture error: a more practiced routine hijacks a similar one. Drive to the store, end up at the office, because the office route is worn deeper into habit.
- Description error: the right action, aimed at the wrong target, because the two look alike from where you're standing. Pouring orange juice into the coffee mug sitting next to the coffee cup.
- Mode error: the same action means something different depending on system state, and you acted on the wrong assumption of which state you were in. Typing a password into the username field.
- Memory-lapse error: you forget a step partway through a sequence. Photocopying page one, forgetting to grab the original off the glass.

## Kinds of mistakes

- Rule-based: applying a rule that doesn't fit the situation, or misjudging which rule applies.
- Knowledge-based: reasoning from an incomplete or wrong model of how the system works.
- Memory-lapse mistake: forgetting the goal itself, not just a step toward it.

## The Swiss cheese model

Reason's image for accidents: every layer of defense has holes, and they shift over time. A bad outcome happens only when the holes in every layer line up at once. No single cause did it. That's why "root cause analysis" is a comforting myth: real accidents are usually five or six contributing conditions that happened to align, and picking one to blame is a story invented after the fact.

## Design for error, not against it

Since error is guaranteed, the job is containing the damage, not preventing every instance.

- Make actions reversible wherever possible, and hard to reverse where they can't be (delete vs. format).
- Make errors visible before they compound, so someone catches the small one instead of discovering the large one it caused.
- Break the connection between a single mistake and a catastrophic outcome; a good system absorbs one failure without collapsing.
- Assume the error will happen. Design the recovery path with the same care as the primary one.

## What to keep

- When someone "makes a mistake," ask what design decision made that mistake reachable.
- Diagnose slip vs. mistake before proposing a fix. They need opposite solutions.
- Stop hunting for the root cause. Look for the set of conditions that had to align, and remove one of them.
