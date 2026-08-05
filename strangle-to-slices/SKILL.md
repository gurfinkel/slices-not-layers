---
name: strangle-to-slices
description: Migrate an existing layered or framework-organized codebase to vertical feature slices — incrementally, on trunk, releasable at every commit; never a big-bang rewrite or a long-lived branch. Strangler-fig, one feature at a time, behavior pinned before it moves, boundaries enforced by a fitness function, legacy deleted last. Repo-agnostic: discovers and binds to the target repo's conventions first. Use when moving layered code to feature slices, extracting one feature into a module, deciding whether to migrate at all, or reviewing a migration plan or PR for big-bang / long-lived-branch risk. Composes with `vertical-slicing` and `validate-the-validator`.

---

# strangle-to-slices

A methodology for moving an existing codebase — layered (`controllers/ services/ repositories/`) or framework-organized — into vertical feature slices **incrementally, on trunk, releasable at every commit**, so the migration never becomes a rewrite or a branch that diverges for months. The patterns are operationalized here as a skill; the underlying ideas are not mine — they come from Martin Fowler's *Strangler Fig Application* and *Branch by Abstraction*, Joel Spolsky's *Things You Should Never Do*, Sam Newman's *Monolith to Microservices*, Michael Feathers' *Working Effectively with Legacy Code*, the *Mikado Method* (Ellnestam & Brolund), Kent Beck's *make the change easy, then make the easy change*, and the layered→slices guidance of Derek Comartin and Feature-Sliced Design. Sources are cited per principle.

## Core idea

A strangler fig grows on a host tree, sends roots down around its trunk, and eventually stands where the tree was — the old tree dies inside, leaving an echo of its shape. That is the only safe way to migrate a live codebase to a new structure: grow the vertical slices **through** the existing layers, one feature at a time, while the old structure keeps serving every feature not yet moved. The system is shippable at every single commit; there is never a moment where it is half-rewritten and un-releasable.

The failure this skill exists to prevent is the **big-bang re-foldering** — dragging the whole repo into `features/` in one heroic branch. That is exactly the rewrite Spolsky warns against, just at the directory level: it discards the working behavior encoded in years of paid-for bug fixes, makes the system un-releasable for the duration, and diverges until it never lands. The migration is itself a **vertical-slice process** (one thin, whole-feature increment at a time) applied to code that already exists — and a codebase that is deliberately half-layered, half-sliced for a long time is the *safe* state, not a failure.

## Inputs and output

**In:** an existing codebase organized by layer or framework convention, a reason the structure is causing pain, and the imminent feature or fix that motivates touching it.

**Out:** either (a) a per-feature migration plan — which slice, the seam, the characterization tests, the cut-over and rollback — sequenced so trunk stays green throughout; or (b) a verdict on whether to migrate at all, and a review of a proposed plan for big-bang / long-lived-branch / grab-bag risk.

## When to apply this skill

- Moving a layered or framework-organized codebase toward feature slices — **incrementally**, driven by real work.
- Extracting one feature into its own self-contained module while the rest stays as-is.
- Reviewing a migration plan for the fatal shapes: a big-bang re-folder, a long-lived "the-migration" branch, or a stalled dual-structure with a `shared/utils` dumping ground.

Do NOT apply — and this gate is load-bearing:

- **When the structure isn't actually hurting.** Migration delivers *zero* user value on its own and carries real risk. If feature work isn't slow and changes aren't smeared across many files, leave it alone — restructuring a working, stable, not-actively-developed app is refactor-for-aesthetics, the most expensive kind of motion.
- **When the code isn't under active development.** The migration only pays off through the features you'll build on the new structure. No upcoming work → no payoff → don't start.
- **As a standalone "migration project."** There is no such thing here. It is always preparatory refactoring for an imminent change (Principle 7), never an open-ended cleanup with its own budget and timeline.

## Relationship to the sister skills

|  | The skill | What it answers |
|---|---|---|
| **`vertical-slicing`** | plans new work as slices | For code we're *about to build*, what increments and in what order? |
| **`strangle-to-slices`** (this) | moves existing code into slices | For code that *already exists*, how do we get it into that structure without a rewrite? |
| **`validate-the-validator`** | makes the eval trustworthy | Is the characterization test / parallel-run comparator that gates the move actually sound? |

They compose: `vertical-slicing` defines the target structure (a shared kernel with feature slices on top); this skill *strangles* an existing layered app into it; and the characterization test and parallel-run comparator that make each move safe are evals that `validate-the-validator` keeps honest.

## The principles

### Principle 1 — Migrate, never rewrite

**Prevents:** discarding years of encoded, paid-for behavior — and handing that time back to nobody's benefit — by "starting clean."

**The rule:** grow the new structure *through* the old (Fowler, *Strangler Fig*); never re-fold the whole repo at once. A layered codebase's controllers and services encode countless edge cases and bug fixes; a from-scratch reorganization throws them away, and per Spolsky there is "no reason to believe you'll do a better job than the first time." Big-bang re-foldering is that rewrite at the directory level.

```
Bad  — a branch that moves the entire app into features/ over three weeks; trunk un-shippable throughout.
Good — one feature grows into a slice this week, on trunk, shippable at every commit; the rest stays layered.
```

**Anti-pattern:** "while we're at it, let's just reorganize everything." That *while-we're-at-it* is the rewrite.

**When overkill:** never — it is the thesis. The rest of the skill is how to obey it.

---

### Principle 2 — One feature at a time, never one layer at a time

**Prevents:** a migration that sweeps all controllers, then all services, then all repositories — which is still a big-bang, just in three acts, and leaves the system un-releasable between them.

**The rule:** pull **one feature's whole vertical** — its entry point, domain logic, data access, DTOs, and tests — into a single feature module, and leave everything else layered (Bogard; Comartin). Minimize coupling *between* slices, maximize it *within*. A codebase intentionally **half-layered, half-sliced** for months is the *safe* state, not an unfinished mess.

```
Bad  — Phase 1: move all controllers to features/. Phase 2: move all services. (un-releasable between phases)
Good — features/checkout/ owns checkout end-to-end; orders, billing, invites stay layered until each is touched.
```

**Anti-pattern:** a `features/` folder whose slices are still layered internally (a mini `service/`+`repository/` inside each) — that is re-foldering, not slicing.

**When overkill:** a feature already isolated enough to move whole in one commit needs no phasing.

---

### Principle 3 — Pin behavior before you move it

**Prevents:** a "move" that is secretly a rewrite — silently changing behavior (a lost email, a double-charge) because nothing proved the old behavior still holds.

**The rule:** legacy is code without tests (Feathers) — you cannot safely move what you cannot re-run and diff. Before touching a feature, write **characterization / approval tests** at the **highest stable entry point** the slice will own (the endpoint, not a private helper — an internal-method test shatters the instant code moves between layers). Capture the *full* observable behavior: response status and body, rows written, messages queued, events emitted, **outbound-call arguments**. Pin what the code does *today* — bugs and all; "fixing" anything is a separate, later commit. Prove the pin is tight with a deliberate mutation: if breaking the code doesn't turn a test red, the net has a hole.

```
Bad  — extract checkout into a slice, click through it once, ship. (a response-only glance misses the lost email)
Good — approval test at POST /checkout capturing body + DB deltas + the charge call's args; mutation-checked; THEN move.
```

**Anti-pattern:** characterizing only the HTTP response and missing the side effects a slice extraction reshuffles.

**When overkill:** a feature already covered by trustworthy end-to-end tests at the right boundary — the net exists; reuse it. Also a **mechanical move** (re-folder + re-wire imports, no logic reshape) in a strongly-typed codebase with CI — `typecheck` + existing boundary tests are the net; a fresh characterization test adds little. And you **cannot pin behavior that was never pinned**: moving already-untested code as-is neither adds nor removes coverage, so don't gate the move on tests the code never had (that's a backlog item, not a prerequisite). Keep full characterization for moves that reshape logic or touch untested side-effecting facades.

---

### Principle 4 — Insert a seam, then flip callers behind it

**Prevents:** a hard cut-over where old and new can't coexist, so there's no way to switch gradually or roll back.

**The rule:** a **seam** is a place to change behavior without editing in place (Feathers); in one process the seam is **Branch by Abstraction** (Fowler/Hammant). Put an interface/port over the feature's layered chain, repoint every caller at it *while it still delegates to the old code* — behavior unchanged, fully releasable, you've only inserted a swap point. Then add the new slice as a **second implementation of the same port**. Old and new now coexist, selectable per call, guarded by a **migration toggle** inside the port's factory (default → old; on → slice). Deploy dark, release by flipping config, **roll back the same way — instantly, no redeploy, no git revert.** All on trunk (Humble): a migration on a long-lived branch prevents both delivery and refactoring.

```
Bad  — delete OrderService, wire in the new slice, deploy, pray. (no coexistence, no rollback)
Good — OrderPlacement port over the layered chain → slice implements the same port → toggle selects → flip / revert live.
```

**Anti-pattern:** a "migration toggle" that never gets removed. It is release-category inventory, not a feature flag — see Principle 8.

**When overkill:** a leaf feature with a single caller and no shared dependencies can sometimes move in one green commit without a standing abstraction — as can a **pure re-foldering with no behavior change**, verified by types + CI; the module's public-interface convention (barrels/index) is the swap point if one is later needed.

---

### Principle 5 — Verify against production with a parallel run

**Prevents:** shipping a slice that is subtly wrong or slower than the code it replaces, discovered by users.

**The rule:** before any user sees the slice, **dark-launch** it (Fowler) as a **Parallel Run** (Newman): for real traffic, invoke *both* the trusted layered path and the new slice, return only the trusted result, and reconcile the slice's output — and its latency — against the trusted one until parity is effectively total. A wrong or slow slice is caught with **zero user impact**, and only then does the toggle open.

```
Bad  — flip the slice to 100% and watch error dashboards.
Good — shadow the slice on live traffic, diff its result vs the layered path, fix mismatches, THEN open the toggle.
```

**Anti-pattern:** treating a green characterization suite as sufficient for cut-over — the suite covers what you imagined; the parallel run covers what production actually sends.

**When overkill:** a low-risk, low-traffic feature where the characterization suite genuinely covers the input space — skip the shadow, but say so. And a **pure code-motion with no behavior change** needs no parallel run at all — types + CI already prove equivalence. Say so.

---

### Principle 6 — Discover prerequisites by experiment; revert on red

**Prevents:** the long-lived broken branch — you start the move, hit tangled coupling, and keep editing deeper for days on a trunk that won't compile.

**The rule:** the **Mikado Method** (Ellnestam & Brolund). Set the goal ("move feature X into a slice"), attempt the naive move directly, and let the compiler and failing tests **enumerate what breaks** — each breakage is a *prerequisite*, recorded on a graph, not fixed in place. Then **revert to green.** The knowledge is the deliverable; the half-done edit is not. Execute the graph **leaves-first, bottom-up**, each prerequisite a clean green commit, redrawing as you go — the goal node lands last, and by then it's "the easy change." One hat per commit (Fowler's two-hats): a commit is *either* behavior-preserving refactor *or* behavior change, never both. When a green commit becomes impossible, you skipped a prerequisite — revert, don't power through.

```
Bad  — start moving Invoicing, discover a god-repository shared by three features, keep hacking; trunk red for a week.
Good — attempt the move, note "shared repo," "cross-feature import," "layer-based DI" as prerequisites, REVERT; land each leaf green; then the move is trivial.
```

**Anti-pattern:** treating the exploratory move as progress and building on top of it instead of reverting.

**When overkill:** a small, well-understood feature whose prerequisites you can already see — draw the graph in your head and go.

---

### Principle 7 — Migrate as preparatory refactoring for an imminent change

**Prevents:** an open-ended "planned refactoring" that has no finish line, ships no value, and diverges into the rewrite you were avoiding.

**The rule:** "make the change easy, then make the easy change" (Beck; Fowler). Carve the slice you are **about to build a feature in or fix a bug in** — the migration is the *make-it-easy* step for real, imminent work, not a standalone cleanup (the workflow Fowler flags as hardest to justify). Value ships with every step because every step is in service of shippable work. To make it self-propelling, **mandate the new structure for all new features** — new work is born sliced, so the layered code shrinks whether or not you ever run a dedicated pass.

```
Bad  — "Q3 tech-debt initiative: migrate to vertical slices." (no feature attached, no finish line)
Good — "I need to add split-tender to checkout → first strangle checkout into a slice, then add split-tender there."
```

**Anti-pattern:** a migration with no feature or bug attached to it. That's the standalone project the When-NOT gate forbids.

**When overkill:** a genuinely greenfield module — there's nothing to migrate; use `vertical-slicing` and build it sliced from the start. Also bounded-exempt: the **one-time foundational pass** that stands up the kernel + boundaries (Principle 8) and codifies "new work is born sliced" as a written rule — legitimately standalone even with no feature attached. The rule still forbids open-ended, layer-by-layer migration with no finish line.

---

### Principle 8 — Stand up a minimal kernel, depopulate the shared layer every iteration, delete legacy last

**Prevents:** the two ways a migration rots — a `shared/utils` grab-bag that recreates the layering you're removing, and scaffolding (toggles, shadow comparators, abstractions, dead legacy paths) that calcifies into permanent indirection.

**The rule:** extract a **minimal, dependency-free shared kernel first** (framework bootstrap, cross-cutting infra, core domain types) that slices sit on and that imports nothing above it (Comartin; FSD). Then, **every iteration**, push anything now used by only one slice *back down into that slice* — depopulating the shared layer is a continuous defense, not an end-of-project cleanup. Prefer duplication over premature abstraction; promote to the kernel only on the third occurrence, once the shape is stable (Rule of Three). Evolve shared kernel types with **expand / migrate / contract** (Fowler *Parallel Change*) — never break all consumers at once. And **contract aggressively**: after a slice bakes at 100%, delete the old layered path, the shadow comparator, and the toggle, and collapse the abstraction if it no longer earns its keep — deleting legacy is a deliberate **final** step per feature, after which rollback cost jumps. Enforce boundaries mechanically with a **fitness-function linter** (e.g. Steiger): migrated slices set to error, legacy to warn, tightening as you go — an unguarded half-migration silently drifts back. And name each extracted slice **by intent, per an established taxonomy** (`features/` verbs, a `core` / `domain` kernel — never a mechanism name like "engine" or "utils"): the migration target *is* the named structure the sister skill `vertical-slicing` (Principle 9) defines, so migrate toward those names, not toward a new dumping ground.

```
Bad  — every extracted helper lands in shared/utils; toggles from six months ago still in the router; legacy code left "just in case."
Good — helpers used by one slice live in that slice; each toggle ships with its removal ticket; legacy deleted after the bake; a linter forbids cross-slice and upward imports.
```

**Anti-pattern:** "we'll clean up the shared layer and remove the flags at the end." There is no end; the scaffolding becomes the architecture.

**When overkill:** a two-slice migration may not need a linter — but it still needs the depopulate-and-delete discipline.

## Bind the skill to the repo first (discover-and-bind)

The principles here are repo-agnostic; the repo supplies the facts. Before applying any of them, read the target repo and bind each generic concept to what this repo actually does — so there is no open question of *how* the skill applies. Do this **at the correct ref** (verify the branch/commit you're on) and treat every binding as a **hypothesis to ground-truth against the actual code**, never an assumption — a wrong ref or an imagined convention produces a wrong migration.

Discover, and write down, for this repo:
- **Slice boundary & naming** — the top-level module structure it uses or intends (inspect the tree + its architecture doc). This is what a "feature slice" *is* here, and where it lives.
- **Public-interface convention** — how a module exposes its boundary (barrels, `index` files, explicit exports). This is your seam (Principle 4).
- **Runtime-split convention** — how server-only code is marked vs. client/browser (a suffix, a folder, a guard package). A boundary you must not cross.
- **Fitness function** — the import-boundary linter / CI gate, if any: what it enforces, and what it leaves to discipline (Principle 8).
- **Safety net** — type-system strength, CI, and existing test coverage *at the boundary you're moving*. This decides how much pinning a move needs (Principles 3–5).
- **Gate inputs** — is this code under active development, with imminent work here? (the When-NOT gate, Principle 7).
- **Documented deviations** — read the repo's own docs (CLAUDE.md / ADRs) for *deliberate* departures from the generic ideal, and treat them as binding. A deviation's rationale is usually not derivable from code — do not "fix" what the repo intentionally chose.

Output a short explicit binding — *"in this repo: slice = …, seam = …, server marker = …, fitness function = … enforcing …, safety net = …, deviations = …"* — and run the rest of the procedure against that, not against a generic assumption.

**Filled example (illustrative — derive your own, don't copy):** *slice = `features/<name>/` (a product capability: its UI, actions, service, and persistence together); seam = a browser barrel + a server barrel (`index.ts` / `index.server.ts`); server marker = a `.server.ts` suffix; fitness function = an import-boundary lint at error in CI, enforcing barrel-only + no-upward imports, but NOT the client/server boundary; safety net = strong types + CI + unit tests at the feature entry point; deviations = a documented reason a server-guard package is deferred — treat as binding.*

**Persist it.** Once derived, write the binding into the repo's own docs (its CLAUDE.md / an ADR) so the next run reads it instead of re-deriving — and a human can correct it.

## The procedure (per feature)

0. **Bind** — discover-and-bind the skill to the repo (above) before gating.
1. **Gate it** — confirm the structure is actually hurting *and* there's imminent work here (When-NOT). If not, stop.
2. **Pin** — characterization / approval tests at the feature's stable entry point, side effects included; mutation-check the net (Principle 3).
3. **Map** — attempt the naive move, record prerequisites on a Mikado graph, revert to green (Principle 6).
4. **Seam** — branch-by-abstraction over the feature's layered chain; repoint callers; it still delegates to old code (Principle 4).
5. **Build** — the new slice as a second implementation behind the same port, owning its whole vertical (Principle 2); pull shared-only code down (Principle 8).
6. **Shadow** — parallel-run the slice on live traffic; reconcile to parity (Principle 5).
7. **Flip** — open the migration toggle; watch; roll back instantly if needed (Principle 4).
8. **Contract** — after the bake, delete legacy, comparator, toggle; collapse the abstraction; tighten the linter (Principle 8).

## Definition of done (run before trusting a migration plan)

```
□ There's a real reason to migrate (active pain + imminent work) — not aesthetics, not a standalone project.
□ It moves one feature at a time, never one layer across all features; a half-sliced repo is an accepted state.
□ Each feature's behavior is pinned (characterization/approval, side effects, mutation-checked) BEFORE any code moves.
□ Every cut-over is behind a seam (branch-by-abstraction) with a migration toggle: dark-deploy, flip to release, instant rollback.
□ Risky/high-traffic slices are parallel-run against the trusted path to parity before the toggle opens.
□ Work lands on trunk in green commits (one hat each); prerequisites are Mikado-mapped, not powered through; no long-lived branch.
□ The migration rides real feature/bug work (preparatory refactoring); new features are mandated sliced.
□ A minimal kernel exists; the shared layer is depopulated every iteration; legacy/toggles/comparators are deleted after each bake; a fitness function guards the boundaries; extracted slices are named by intent (features / core / shared), never "engine"/"utils".
□ The system is releasable at every single commit.
```

## Reviewing with this skill (findings are hypotheses, not verdicts)

Applying the skill *generates candidate findings; it does not certify them.* A green Definition-of-done is not proof, and a skill flag, a sub-agent claim, and a quick grep are all hypotheses. Before you act on one — above all before you comment on someone else's PR — discharge it:

- **Verify at the correct ref.** Confirm the branch/commit you're reading, and re-derive the fact from the actual code, not from what the skill expects to find.
- **Sort into three buckets, not pass/fail:** (1) **real defect** → comment; (2) **skill too strict for this repo/change** → *scope the skill* (add a when-not), don't fault the code; (3) **concern dissolves on inspection** → drop it, silently.
- **Rank by blast radius.** A money / credentials / auth path outranks a folder name; spend scrutiny where a wrong answer costs most.
- **Never manufacture a finding on moved-untested code.** You cannot pin behavior that was never pinned; a pure move that neither adds nor removes coverage is not this change's defect — a backlog note at most.

(The full discipline for trusting a review's own findings lives in `validate-the-validator`.)

## Costs to acknowledge honestly

- **The safety net is most of the work.** Characterization tests and a parallel-run comparator often cost more than the move itself — that cost *is* the de-risking; skipping it turns "migrate" into "rewrite and hope."
- **Scaffolding has carrying cost.** Every seam, toggle, and shadow comparator is inventory you must remove; a migration that adds them faster than it retires them is going backwards.
- **Half-migrated is the normal state for a long time.** Two structures coexisting is correct, but it demands the boundary discipline (Principle 8) or it drifts into permanent mud.
- **The hardest part is not starting the ones you shouldn't.** The When-NOT gate saves more than any technique here — most "we should migrate this" impulses fail it.

## Source citations

The method is a synthesis; sources are cited per principle in [CITATIONS.md](./CITATIONS.md).
