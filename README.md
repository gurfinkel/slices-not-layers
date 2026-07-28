# slices-not-layers

Skills for [Claude Code](https://claude.com/claude-code) that plan and build software as thin **vertical slices** — each cutting end-to-end through every layer to deliver demonstrable behavior — instead of horizontal layers that only integrate at the end, where the project dies.

| Skill | Produces | Core idea |
|---|---|---|
| [`vertical-slicing`](./vertical-slicing) | an ordered slice plan (slice 0, 1, 2 …) with a per-slice definition of done — or a verdict on whether an existing plan is genuinely vertical | start with a walking skeleton, then grow the system in thin, demoable, eval-gated increments — sequenced risk-first, never a horizontal band |

The family is meant to grow: `vertical-slicing` owns the planning method, technology-agnostic; planned siblings — `story-splitting` (the cut-line patterns) and `walking-skeleton` (kick-starting a greenfield project) — would sit beside it.

## Install

Copy the skill directories into your Claude Code skills folder:

```bash
git clone https://github.com/gurfinkel/slices-not-layers.git
cp -R slices-not-layers/vertical-slicing ~/.claude/skills/
```

Start a new Claude Code session and the skill becomes available.

## Credits

The skill operationalizes a synthesis of named work; the underlying ideas are not mine:

- **Walking Skeleton** — Alistair Cockburn (*Crystal Clear*); **Tracer Bullets** — Andrew Hunt & David Thomas (*The Pragmatic Programmer*); the thinnest end-to-end slice in *Growing Object-Oriented Software, Guided by Tests* — Steve Freeman & Nat Pryce.
- **Vertical Slice Architecture** — Jimmy Bogard ("SOLID Architecture in Slices Not Layers"); **Screaming Architecture** — Robert C. Martin.
- **INVEST** — Bill Wake; **Elephant Carpaccio** — Alistair Cockburn & Henrik Kniberg; story-splitting patterns — Richard Lawrence; **SPIDR** — Mike Cohn.
- **The Wrong Abstraction** — Sandi Metz; the **Rule of Three** — Martin Fowler & Kent Beck; the **Shared Kernel** — Eric Evans.

## License

[MIT](./LICENSE)
