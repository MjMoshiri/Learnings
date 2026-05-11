# LiveChat (Haskell, IHP)

Basic live chat app built with Haskell and the IHP framework. Goal was less about chat and more about touching a pure functional web stack end to end: routing, types, persistence, views.

IHP's type-driven workflow was the interesting part. Schema changes propagate through types, so a missing migration shows up as a compile error, not a runtime crash.
