# slices-not-layers

Skills for [Claude Code](https://claude.com/claude-code) that plan, build, and migrate software as thin **vertical slices** — each cutting end-to-end through every layer to deliver demonstrable behavior — instead of horizontal layers that only integrate at the end, where the project dies.

| Skill | Produces | Core idea |
|---|---|---|
| [`vertical-slicing`](./vertical-slicing) | an ordered slice plan (slice 0, 1, 2 …) with a per-slice definition of done — or a verdict on whether an existing plan is genuinely vertical | start with a walking skeleton, then grow the system in thin, demoable, eval-gated increments — sequenced risk-first, never a horizontal band |
| [`strangle-to-slices`](./strangle-to-slices) | a per-feature migration plan — or a verdict on whether to migrate at all, and whether a plan risks a big-bang or a long-lived branch | strangle an existing layered codebase into slices one feature at a time — pinned by characterization tests, releasable at every commit, never a rewrite |

`vertical-slicing` plans new work as slices; `strangle-to-slices` moves existing code into them. The family can still grow — `story-splitting` (the cut-line patterns) and `walking-skeleton` (kick-starting a greenfield project) are natural future siblings.

## Install

Copy the skill directories into your Claude Code skills folder:

```bash
git clone https://github.com/gurfinkel/slices-not-layers.git
cp -R slices-not-layers/{vertical-slicing,strangle-to-slices} ~/.claude/skills/
```

Start a new Claude Code session and the skill becomes available.

## Credits

The skills operationalize a synthesis of named work; the underlying ideas are not mine.

**`vertical-slicing`:**

- **Walking Skeleton** — Alistair Cockburn (*Crystal Clear*); **Tracer Bullets** — Andrew Hunt & David Thomas (*The Pragmatic Programmer*); the thinnest end-to-end slice in *Growing Object-Oriented Software, Guided by Tests* — Steve Freeman & Nat Pryce.
- **Vertical Slice Architecture** — Jimmy Bogard ("SOLID Architecture in Slices Not Layers"); **Screaming Architecture** — Robert C. Martin.
- **INVEST** — Bill Wake; **Elephant Carpaccio** — Alistair Cockburn & Henrik Kniberg; story-splitting patterns — Richard Lawrence; **SPIDR** — Mike Cohn.
- **The Wrong Abstraction** — Sandi Metz; the **Rule of Three** — Martin Fowler & Kent Beck; the **Shared Kernel** — Eric Evans.

**`strangle-to-slices`:**

- **Strangler Fig Application** — Martin Fowler; **Things You Should Never Do** — Joel Spolsky; **Monolith to Microservices** — Sam Newman.
- **Working Effectively with Legacy Code** (characterization tests, seams) — Michael Feathers; **Branch by Abstraction** — Martin Fowler & Paul Hammant; **Parallel Change** / **Feature Toggles** — Martin Fowler & Pete Hodgson.
- **The Mikado Method** — Ola Ellnestam & Daniel Brolund; **preparatory refactoring** — Kent Beck & Martin Fowler; **layered→slices migration** — Derek Comartin & Feature-Sliced Design.

## License

[MIT](./LICENSE)
