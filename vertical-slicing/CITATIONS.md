# vertical-slicing — source citations

The method is a synthesis, cited by the layer each source owns.

**Starting a build (the walking skeleton)**
- **Alistair Cockburn** — *Walking Skeleton* (*Crystal Clear*): the thinnest end-to-end implementation that links the main architectural components, grown while kept running.
- **Andrew Hunt & David Thomas** — *The Pragmatic Programmer*, "Tracer Bullets": lean-but-complete code fired along the whole path, kept and grown — versus a throwaway prototype.
- **Steve Freeman & Nat Pryce** — *Growing Object-Oriented Software, Guided by Tests*, ch. 10: "the thinnest slice of real functionality we can automatically build, deploy, and test end-to-end"; the first end-to-end test as iteration zero.

**Organizing by slice**
- **Jimmy Bogard** — *Vertical Slice Architecture* and "SOLID Architecture in Slices Not Layers": minimize coupling between slices, maximize it within; organize around the axis of change.
- **Robert C. Martin** — *Screaming Architecture*: the top-level structure should scream the domain, not the framework.

**Sizing & sequencing**
- **Bill Wake** — *INVEST in Good Stories*: Independent, Negotiable, Valuable, Estimable, Small, Testable; split through the layers, not by layer.
- **Alistair Cockburn & Henrik Kniberg** — *Elephant Carpaccio*: decompose one feature into ~15–20 one-day, end-to-end slices.
- **Richard Lawrence** — the story-splitting patterns; **Mike Cohn** — *SPIDR* and the value/risk sequencing in *Agile Estimating and Planning*.

**Share vs. duplicate**
- **Sandi Metz** — *The Wrong Abstraction*: duplication is far cheaper than the wrong abstraction; inline it back when it sprouts conditionals.
- **Martin Fowler & Kent Beck** — *Refactoring*: the Rule of Three.
- **Eric Evans** — *Domain-Driven Design*: the Shared Kernel — share only where the cost of divergence exceeds the cost of coordination.

**Naming & structure**
- **Feature-Sliced Design** (layers / slices / segments, and the *Steiger* linter) and **Bulletproof React** — Alan Alickovic — for the JS/TS folder taxonomy; **Ports & Adapters (Hexagonal)** — Alistair Cockburn — for naming the cross-cutting core behind interfaces. (*Screaming Architecture* and the *Shared Kernel* are cited above.)
