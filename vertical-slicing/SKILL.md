---
name: vertical-slicing
description: Plan and build software as thin, vertical, end-to-end slices — each cutting through every layer to deliver demonstrable behavior — instead of horizontal layers that integrate only at the end. Walking skeleton first, Carpaccio-thin increments, eval-first definition of done, deliberate shared kernel, taxonomy-named folders. Repo-agnostic: discovers and binds to the target repo's conventions first. Use when planning a build, defining slice 0/1/2, decomposing a feature or epic into increments, sequencing under deadline or risk, kicking off a greenfield project, naming a folder structure, or reviewing whether a plan is genuinely vertical vs. a disguised layered or big-bang plan. Composes with `strangle-to-slices` (migrating existing code into this structure) and `validate-the-validator`.
---

# vertical-slicing

A methodology for planning a build as a sequence of thin **vertical slices** — increments that each cut through every layer of the system to change observable behavior — and for checking that a plan is genuinely vertical rather than a layered or big-bang plan in disguise. The patterns are operationalized here as a skill; the underlying ideas are not mine — they come from Alistair Cockburn's *Walking Skeleton*, the *Tracer Bullets* of Hunt & Thomas, the end-to-end skeleton of Freeman & Pryce's *Growing Object-Oriented Software*, Jimmy Bogard's *Vertical Slice Architecture*, Bill Wake's *INVEST*, Cockburn & Kniberg's *Elephant Carpaccio*, the story-splitting patterns of Richard Lawrence and Mike Cohn, Sandi Metz's *The Wrong Abstraction*, and Eric Evans' *Shared Kernel*. Sources are cited per principle.

## Core idea

Picture the system as a stack of layers — data, logic, API, interface. There are two ways to build it. **Horizontally**: finish the whole data layer, then the whole logic layer, then the whole interface — one band at a time across the entire app. Nothing *works* until the very end, when the bands are finally stacked and integrated — which is exactly when they don't fit, in the last week, with nothing to show and no time to fix it. **Vertically**: take one capability and build it all the way down the stack — a thin wedge with a little of every layer — so that one capability *actually runs, end to end*. Then build the next wedge.

The failure mode this skill exists to prevent is the **horizontal band masquerading as progress**: a "retrieval engine," an "ingestion module," a "data layer" — real work that does nothing observable and hides all the integration risk until the end. A slice is the opposite: **narrow scope, full depth, demonstrable now.** Build vertically and the system always runs, the riskiest assumptions are retired first while change is cheap, and every increment can be shown, tested, and trusted on its own.

## Inputs and output

**In:** either (a) a capability / feature / epic to build, plus the known risks and constraints; or (b) an existing build plan to review.

**Out:** either (a) an ordered slice plan — slice 0 (a walking skeleton) then slices 1..n — each with a one-line demoable outcome and its acceptance/eval check; or (b) a verdict on whether the plan is genuinely vertical, with the specific horizontal bands, over-fat slices, or mis-sequencing called out and re-cut.

## When to apply this skill

- Kick-starting a greenfield project or a major new subsystem — defining slice 0 and the sequence after it.
- Decomposing a feature or epic into increments you can build, demo, and ship one at a time.
- Sequencing work under a deadline or real technical risk.
- Reviewing a plan (yours or an agent's) to confirm it supports vertical slicing and isn't a layered/big-bang plan wearing slice vocabulary.

Do NOT apply for:
- A single trivial change with no fork and no integration — just do it.
- A pure learning question — that is a timeboxed **spike** (Principle 8), not a slice.
- The *code-organization* debate in isolation (feature folders vs. layers) — that is downstream; this skill plans the build, and the folder structure follows from it.

## Relationship to the sister skills

|  | The skill | What it answers |
|---|---|---|
| **`decision-documentation`** | records a reviewable decision | *What* should we build, and why this over the alternative? |
| **`vertical-slicing`** (this) | plans the build as slices | *How* do we deliver it — in what increments, in what order? |
| **`validate-the-validator`** | makes the eval trustworthy | *Is the check* that gates each slice actually sound? |

They compose in one direction: a decision fixes *what*; this skill slices *how*; and each slice's definition of done is an acceptance/eval case that `validate-the-validator` keeps honest. A slice plan frequently *implements* a set of ADRs — cite them as the constraints the slices must satisfy.

## The principles

### Principle 1 — A slice is vertical, or it is nothing

**Prevents:** a "slice" that is really a horizontal band — a layer or an engine — which delivers no observable behavior and hides integration risk until the end.

**The test:** every slice cuts through the whole path (input → logic → data → output) and **changes behavior you can demonstrate.** If you cannot demo it, it is horizontal; recut. Wake's rule (INVEST): split *through* the layers, never *by* layer.

```
Bad  — Slice 1: "build the retrieval engine."  Slice 2: "build the ingestion layer."
       Neither runs; neither can be shown; the first time they meet is integration day.

Good — Slice 1: "ask one question, get one grounded answer over one document, end to end."
       Thin, but it runs and you can demo it — and it exercises every layer at once.
```

**Anti-pattern:** naming a technical layer a "slice" so a layered plan can keep its vocabulary. A slice is a *capability*, not a *tier*.

**When overkill:** never — this is the load-bearing definition. Everything else refines it.

---

### Principle 2 — Start with a walking skeleton

**Prevents:** months of horizontal build before anything runs end-to-end, so architecture and integration risk surface only when it is expensive to fix.

**The rule:** the **first** slice is a *walking skeleton* (Cockburn; Freeman & Pryce) — the thinnest end-to-end path that actually runs, **built, deployed, and tested by one command from day one.** Keep the feature "so simple it's obvious and uninteresting" so attention goes to wiring the pipes and enforcing the correctness contract, not to feature richness. It is a **tracer bullet** (Hunt & Thomas): lean but real, *kept and grown into the product* — not a prototype you throw away. Wiring the automated build+test *is* iteration zero; feature slices begin only once it is green.

```
Bad  — spike the hard part in a notebook, build features for weeks, wire deploy "later."
       The skeleton never walked; the first real end-to-end run is week 8.

Good — one command migrates the store, runs the real pipeline on one input, exercises the
       full path, and asserts the core contract — green in CI on day one. Then grow muscle on it.
```

**Anti-pattern:** a throwaway spike dressed up as the skeleton; or deferring the product's *correctness contract* (the invariant it is judged on — grounded-or-refused, never-double-charge, etc.) out of the skeleton. If the skeleton doesn't walk the path that actually matters, it isn't walking.

**When overkill:** adding to a mature system that already has a walking skeleton — you are past iteration zero; grow slices onto what exists.

---

### Principle 3 — Carpaccio-thin: if you can't demo it in ~a day, cut again

**Prevents:** slices sized in weeks, which are just small horizontal projects — they hide the same integration risk, only later.

**The rule:** slice *Elephant-Carpaccio* thin (Cockburn/Kniberg). A capability that feels like "one feature" decomposes into **~15–20 slices, each roughly a day, each demoable end-to-end.** Two checks: can't demo it → it's horizontal; took more than ~2 days → it's too fat, cut again.

```
Bad  — Slice: "user management."  (weeks; secretly a layered sub-project)

Good — "admin adds one user with a fixed role" → "…sets the role" → "…deactivates" → …
       fifteen one-day slices, each shippable and shown on its own.
```

**Anti-pattern:** treating thinness as sloppiness. A thin slice is *complete* front-to-back for the narrow case it covers — it is small in *scope*, full in *depth*.

**When overkill:** a genuinely atomic change that cannot be cut further without becoming non-demoable.

---

### Principle 4 — Testable means the acceptance check is written first

**Prevents:** a slice declared "done" with no executable proof it works — the seam where correctness quietly rots.

**The rule:** Wake's **T** in INVEST, operationalized. Before building a slice, write its **acceptance check** — for correctness-critical work, a golden **eval case**: the input, the expected output (or the expected refusal/error), and the assertion. A slice whose check is not wired into the gate and passing is **not done.** The check is the definition of done, not an afterthought.

```
Bad  — build the slice, then eyeball whether the answer "looks right."

Good — first commit the case: "Q from a single passage → must answer AND cite a resolvable id."
       Then build until the gate goes green. Eval-first, per validate-the-validator.
```

**Anti-pattern:** "I'll add tests after the demo." The check written *after* is shaped to pass, not to catch — and usually never comes.

**When overkill:** throwaway spikes (Principle 8), which are learning, not delivery, and carry no definition of done. And for a **mechanical, strongly-typed slice**, `typecheck` + a boundary test can be the acceptance check; reserve full eval-cases for correctness-critical behavior (money, credentials, auth).

---

### Principle 5 — Cut along one axis at a time

**Prevents:** splits that produce dependent, un-shippable fragments because they cut on several axes at once (or on the layer axis).

**The rule:** find the *single* cut-line that yields an independently shippable slice. Use a named pattern — Lawrence's (workflow-step, business-rule variation, data variation, interface, CRUD operation, effort, simple→complex, defer-performance, spike) or Cohn's **SPIDR** (Spike, Path, Interface, Data, Rules). Prefer the split that exposes low-value work you can *drop*, and prefer roughly equal-sized slices over one fat + one trivial.

```
Bad  — split "checkout" into "front-end slice" + "back-end slice" (the layer axis — neither ships).

Good — split by path: "checkout with one saved card, no coupons, no address change" first;
       then coupons, then new address, then alternate payment — one rule added per slice.
```

**Anti-pattern:** splitting by component/tier. That is the horizontal band (Principle 1) wearing a splitting-pattern name.

**When overkill:** a slice already atomic and shippable needs no further cut.

---

### Principle 6 — Sequence risk-first, then value-first

**Prevents:** building the easy, low-risk slices first and hitting the project-killing unknown in the last week.

**The rule:** two phases. **First, the walking skeleton**, chosen to link every architectural component end-to-end and retire *architecture* risk (Cockburn). **Then order by Cohn's value/risk matrix:** high-value/high-risk first, then high-value/low-risk; *defer* low-value/high-risk; *drop* low-value/low-risk. In a correctness-critical system, the behavior that most destroys trust when wrong (a confident-but-ungrounded answer, a silent mischarge) is high-risk and belongs *early*, not last.

```
Bad  — happy path, then polish, then — finally — the refusal/error path in week 11.

Good — skeleton (proves the architecture) → the failure/refusal behavior (top correctness risk)
       → the precision refinements → new content types → the highest-complexity slice, last.
```

**Anti-pattern:** sequencing by what's *easy* or by feature-list order. Order by what you'll most regret learning late.

**When overkill:** a flat backlog of equal-risk, equal-value slices — then value order alone suffices.

---

### Principle 7 — Share on purpose behind a narrow contract; slice the rest

**Prevents:** two opposite failures — copying a cross-cutting invariant into every slice until the copies drift, *and* forcing genuinely-divergent work into one abstraction that grows conditionals.

**The rule:** a cross-cutting invariant that must **never** diverge (the correctness engine, auth, money math) is a **deliberate Shared Kernel** (Evans) — extracted *up front, on purpose,* behind a small, stable contract, justified because the cost of divergence exceeds the cost of coordination. Everything genuinely variant stays **sliced**; between slices, **default to duplication** and extract only on the third strike (**Rule of Three**, Fowler/Beck) — and only if you can name the abstraction *without a type flag*. If naming it needs `if (type === X)`, it is the wrong abstraction (Metz): inline it back. The kernel must **never branch on which slice called it** — push per-variant difference *down* into the contract's typed data, never *up* as a branch in shared code.

```
Bad  — grounding logic copied into prose/, charts/, webinar/ (drift → a miscited answer);
       OR one process(type) in the engine sprouting `if type === 'chart'` conditionals.

Good — engine/ is a leaf: a narrow contract (search/ground/cite over a canonical Chunk type)
       shared by every slice; each ingestion slice PARSES its raw input into that Chunk.
       Slices never import each other; the engine never imports a slice.
```

**Solo governance:** a Shared Kernel normally needs a change board. Solo, replace the board with **the eval gate** — any change to the contract or the kernel must re-green the whole gate before merge. The gate is your compatibility test.

**Anti-pattern:** a catch-all `shared/utils.ts` grab-bag; a new boolean parameter added to a shared function to serve a new variant; premature extraction before the Rule of Three.

**When overkill:** no genuine cross-cutting invariant exists → no kernel; keep everything sliced.

---

### Principle 8 — A spike is timeboxed learning, not a slice

**Prevents:** counting throwaway investigation as delivered value, and estimating a slice you don't yet understand.

**The rule:** when a slice isn't *estimable* because of an unknown, precede it with a **spike** (Cohn) — a timeboxed experiment that answers **one** question so the following real slice becomes estimable. A spike is disposable, carries no definition of done, and is **not** counted as a shipped slice. Its output is knowledge; the real vertical slice comes after.

```
Bad  — "Slice 4: chart lookup" estimated at a day — but 90% is unknown extraction work.

Good — Spike (timeboxed): can we extract structured figures from these decks, and how?
       Then Slice 4: "answer a fact living in one chart," now estimable, on the proven engine.
```

**Anti-pattern:** an open-ended "research slice" with no timebox and no question — that is not a spike, it's a stall.

**When overkill:** the slice is already estimable — no spike needed; just build it.

---

### Principle 9 — Name by intent; adopt an established taxonomy, don't invent folders

**Prevents:** an invented cross-cutting folder — `engine/`, `manager/`, `helpers/`, `utils/` — that hides intent, belongs to no convention, and grows into a dumping ground.

**The rule:** folder names must **scream the domain** (Martin, *Screaming Architecture*), and you **adopt a battle-tested taxonomy** rather than coining your own — Feature-Sliced Design or Bulletproof React for JS/TS; Ports & Adapters, domain services, and the Shared Kernel (DDD / Hexagonal) for the cross-cutting core. Actions are `features/` named by a **verb** (`ask`, not `qa`); domain nouns and the shared correctness kernel get a **domain** name (`core`, `domain`, `entities`) — never a mechanism name. Imports flow one way (down into the kernel and shared, never sideways between slices), and you **enforce the taxonomy with a linter** (Steiger; `eslint import/no-restricted-paths`) — an un-enforced convention erodes within weeks.

```
Bad  — src/engine/ (retrieval+grounding+citation), src/qa/, src/utils/ — invented, mechanism-named, unenforced.
Good — src/core/{retrieval,grounding,citation,model}, src/features/ask, src/shared/ — named by intent, linter-enforced.
```

**Anti-pattern:** a `shared/utils.ts` or `engine/` catch-all. If you can't name a folder by what it *is* in the domain, you haven't found its home yet.

**When overkill:** a throwaway spike — but any code that survives it gets a real, convention-named home.

## Bind the skill to the repo first (discover-and-bind)

The principles here are repo-agnostic; the repo supplies the facts. Before planning slices, read the target repo and bind each generic concept to what this repo actually does — so a new slice is **born conformant**, not retrofitted. Do this **at the correct ref** (verify the branch/commit you're on) and treat every binding as a **hypothesis to ground-truth against the actual code**, never an assumption.

Discover, and write down, for this repo:
- **Slice home & naming** — the top-level module structure new work lives in (inspect the tree + its architecture doc). Where a new slice goes, and what it's named (Principle 9).
- **Public-interface convention** — how a module exposes its boundary (barrels, `index` files, explicit exports). How the new slice is consumed.
- **Runtime-split convention** — how server-only code is marked vs. client/browser. A boundary the slice must obey.
- **Fitness function** — the import-boundary linter / CI gate the slice must pass (Principle 9).
- **Safety net** — type-system strength, CI, and the acceptance-check style the repo expects (Principle 4).
- **Shared kernel** — the existing cross-cutting core the slice sits on rather than duplicates (Principle 7).

Output a short explicit binding — *"in this repo: slice home = …, interface = …, server marker = …, fitness function = …, acceptance check = …, kernel = …"* — and plan the slices against that.

**Filled example (illustrative — derive your own):** *slice home = `features/<verb>/` (named by action); interface = a browser barrel + a server barrel; server marker = a `.server.ts` suffix; fitness function = an import-boundary lint at error in CI; acceptance check = `typecheck` + a boundary test for mechanical work, a golden eval-case for correctness-critical behavior; kernel = the existing cross-cutting core (auth, money, the correctness engine) the slice sits on.*

**Persist it.** Once derived, write the binding into the repo's own docs (its CLAUDE.md / an ADR) so the next run reads it instead of re-deriving.

## The procedure

To produce a slice plan from a capability:

0. **Bind** — discover-and-bind the skill to the repo (above) before drawing the skeleton.
1. **State the capability as observable behavior** — what a user (or caller) can *do* when it's done.
2. **Draw the walking skeleton** (Principle 2) — the thinnest end-to-end path that runs, with the correctness contract baked in, buildable/deployable/testable by one command. This is slice 0.
3. **Enumerate candidate slices** by cutting along *one axis at a time* (Principle 5) — list the paths, rules, and data variations.
4. **Size each Carpaccio-thin** (Principle 3) — ~a day, demoable; recut anything fatter.
5. **Write each slice's acceptance/eval check first** (Principle 4) — that is its definition of done.
6. **Sequence** (Principle 6) — skeleton, then risk-first, then value-first; defer/drop the low-value.
7. **Mark the shared kernel vs. the sliced variants** (Principle 7) — name the narrow contract; insert spikes (Principle 8) before any un-estimable slice.

## Definition of done (run before trusting a slice plan)

```
□ Slice 0 is a walking skeleton: thinnest end-to-end path, runs by one command in CI, correctness contract baked in.
□ Every slice cuts through all layers and is demoable — no horizontal bands, no "engine"/"layer" slices.
□ Every slice is Carpaccio-thin (~a day); anything fatter has been recut.
□ Every slice has its acceptance/eval check written and wired into the gate — that is its definition of done.
□ Each split cuts along ONE named axis (Lawrence / SPIDR); none split by tier.
□ Sequence is skeleton → risk-first → value-first; low-value/high-risk deferred, low-value/low-risk dropped.
□ Cross-cutting invariants are a deliberate shared kernel behind a narrow contract; variant work is sliced; the kernel never branches on the caller.
□ Un-estimable work is preceded by a timeboxed spike, labeled as throwaway learning — not counted as a delivered slice.
□ The system stays runnable (gate green) after every slice — never a "big integration" step at the end.
□ Folders are named by an established taxonomy (features / core / shared — a verb for actions, a domain name for the kernel), never an invented mechanism name like "engine"; enforced by a linter.
```

## Reviewing with this skill (findings are hypotheses, not verdicts)

Applying the skill *generates candidate findings; it does not certify them.* A green Definition-of-done is not proof, and a skill flag, a sub-agent claim, and a quick grep are all hypotheses. Before you act on one — above all before you comment on someone else's plan or PR — discharge it:

- **Verify at the correct ref.** Confirm the branch/commit you're reading, and re-derive the fact from the actual code, not from what the skill expects to find.
- **Sort into three buckets, not pass/fail:** (1) **real defect** → comment; (2) **skill too strict for this repo/work** → *scope the skill* (add a when-not), don't fault the code; (3) **concern dissolves on inspection** → drop it, silently.
- **Rank by blast radius.** A money / credentials / auth path outranks a folder name; spend scrutiny where a wrong answer costs most.
- **Never manufacture a finding.** A missing test on pre-existing behavior this slice doesn't touch is not this slice's defect — a backlog note at most.

(The full discipline for trusting a review's own findings lives in `validate-the-validator`.)

## Costs to acknowledge honestly

- **Iteration zero is real upfront work.** Wiring build+deploy+test to run by one command before the first feature has cost — paid back the first time it flushes an integration mistake while change is still cheap.
- **Thin-slicing takes discipline.** It is easier to write "user management" than fifteen one-day slices; the decomposition is the work, and it is where the value is.
- **The shared-kernel line is a judgment call.** Share too early and you get the wrong abstraction; share too late and correctness logic drifts. Principle 7 gives the test, not a formula.
- **Eval-first is slower per slice, faster per project.** Writing the check before the code feels like drag on slice 1 and saves the project by slice 10.

## Source citations

The method is a synthesis; sources are cited per principle in [CITATIONS.md](./CITATIONS.md).
