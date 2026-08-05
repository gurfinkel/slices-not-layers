# strangle-to-slices — source citations

The method is a synthesis, cited by the layer each source owns.

**Why incremental, not big-bang**
- **Martin Fowler** — *StranglerFigApplication*: grow the new system on the edges of the old; investment and returns accrue gradually and visibly.
- **Joel Spolsky** — *Things You Should Never Do, Part I*: the evidence that from-scratch rewrites fail — reading code is harder than writing it, so working code is mistaken for junk.
- **Sam Newman** — *Monolith to Microservices*: the incremental-decomposition pattern catalog; the modular-monolith end state.

**The safety net**
- **Michael Feathers** — *Working Effectively with Legacy Code*: legacy = code without tests; characterization tests; the Seam Model; Sprout/Wrap; the Legacy Code Change Algorithm.
- **Emily Bache / Nicolas Carlo** — approval / golden-master testing for wide outputs.

**Keeping it releasable**
- **Martin Fowler / Paul Hammant / Jez Humble** — *Branch by Abstraction*: coexist old and new behind one abstraction, on trunk.
- **Martin Fowler** — *Parallel Change* (expand/migrate/contract) and *Dark Launching*; **Pete Hodgson** — *Feature Toggles*: migration toggles as short-lived, removal-ticketed inventory.

**Disciplined sequencing**
- **Ola Ellnestam & Daniel Brolund** — *The Mikado Method*: discover prerequisites by experiment, revert on red, execute leaves-first.
- **Kent Beck** — "make the change easy, then make the easy change"; **Martin Fowler** — *Preparatory Refactoring*, *Refactoring* (the two-hats rule).

**The target structure**
- **Derek Comartin** — *Restructuring to a Vertical Slice Architecture*; **Feature-Sliced Design** — the migration guide and the *Steiger* fitness-function linter; **Jimmy Bogard** — coupling within vs. between slices; **Bulletproof React** — Alan Alickovic — the practical feature-folder taxonomy; **Ports & Adapters** — Alistair Cockburn — for naming the extracted core behind interfaces. (The naming discipline itself lives in the sister skill `vertical-slicing`, Principle 9.)
