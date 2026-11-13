# Phases 2–3: Feasibility, Red-Team, and Target Architecture

#### Phase 2: Feasibility Spike, Strategy Fork & Safety Ladder

This is the pivot of the whole plan. **Do it before recommending a target
architecture or writing any phases.** Its job is to answer, quickly and honestly:
*how alive is this system, what migration shape fits, and how much regression
safety can we actually achieve?*

##### 2a. Run a time-boxed feasibility spike

Put a **hard time box** on it (e.g. one day). The spike's question is **NOT "can
we make CI green?"** — it is **"can we get this to run even *once* to capture its
behavior?"** Probe, per component / deployable unit:

- Does it **install** its dependencies today (ideally **from a committed
  lockfile without hand-patching**)?
- Do **native/build steps** succeed on a **currently supported** toolchain?
- Does it **boot / start**?
- Can the **test runner execute at all**, and does **≥1 meaningful test pass**?

Record what you actually observed — including partial success (common in
monorepos: one service installs and tests green while another can't compile). Do
**not** sink two sprints resurrecting a corpse to discover it's unresurrectable.
When the time box expires, decide with the evidence you have.

##### 2b. Locate the Testability Milestone (per component)

**"Testable" is not a precondition you can satisfy on a dead app — for a dead app
it is an *output* of modernization, not an input.** So every plan must name, up
front, its **Testability Milestone**: the specific phase at which the system
(or a given component) first satisfies **all four** conditions at once:

1. Runs on a **supported** (non-EOL) runtime.
2. Dependencies install **from a lockfile without hand-patching**.
3. Native / build steps **succeed on the current toolchain**.
4. The **test runner executes and ≥1 meaningful test passes in CI**.

Assess this **per component** — as the reference Nylas run showed, a near-modern
cloud service can cross the line immediately while a legacy Electron client
doesn't cross it until several phases later. **This milestone is the single most
important marker in the plan.** State where it lands for each component, out loud.

##### 2c. Label every phase pre- or post-testability

The Testability Milestone splits the effort into two regimes with **different
safety rules**:

- **Pre-testability ("dark") regime:** the component can't be trusted to run.
  Safety comes from **non-test** sources (see the safety ladder below).
  **Requiring a CI test gate here is a category error** — you're demanding green
  tests from something that isn't alive yet.
- **Post-testability ("lit") regime:** it builds and runs on a live, supported
  stack. *Now* real CI, characterization, and e2e are achievable and worth the
  investment. The classic green-gate belongs **here**.

**Rule: never require an automated test gate on a component before that component
crosses its own testability line.**

##### 2d. Choose the migration strategy — the A/B fork

The spike decides which of two shapes fits (this is a per-component call in a
monorepo):

- **(A) Freeze-then-lift.** Net the old app *as-is*, then upgrade under the net.
  Requires the old app to be **resurrectable at acceptable cost** — i.e. it
  *almost* runs. This is the traditional "characterization tests before you
  change anything" path. **Opt-in, not the default.**
- **(B) Beachhead-then-expand (walking skeleton).** Don't net the corpse. Drive
  the **thinnest possible end-to-end slice onto the modern stack** until it
  builds, boots, and tests; then strangle the rest in, writing tests **on the new
  stack** as you go. **This is the default for dead apps.** In strategy B, the
  first phase is *"reach the Testability Milestone with the smallest slice,"* not
  *"freeze the whole legacy toolchain."*

If a component **can't be built or run at all**, characterization tests on the
*old* stack are impossible by definition — invert the approach: **stop trying to
net the corpse and build the net on the *target* stack incrementally**, using the
old code/output as the reference oracle.

##### 2e. Pick the highest achievable rung on the safety ladder

Regression safety is a ladder, not a binary. Choose the **highest rung you can
actually reach** per component; **a downgrade to a lower rung is a blessed,
first-class outcome — not a failure — provided the residual risk is named.**

- **L4 — Full automated gate:** green lint + unit + characterization + e2e in CI.
- **L3 — Partial gate:** some suites green + lockfile + CI on the achievable
  subset; the rest **quarantined** with a named list.
- **L2 — Characterization / golden-master only:** can't run in CI, but capture
  I/O snapshots / behavioral diffs (manual is acceptable).
- **L1 — Reversibility-based:** no runnable tests; safety comes from small
  reversible steps, strangler / parallel-run, a smoke checklist, and review.
- **L0 — No net possible:** treat as a spec-driven rewrite (old code = the spec)
  or archive.

**Safety is not only tests.** There are at least five sources, and on dead apps
2–4 often outweigh 1:

1. **Tests** (characterization / unit / e2e) — strongest, often unachievable on
   dead apps.
2. **Reversibility** — tiny commits, branch-per-change, trivial rollback.
3. **Isolation** — feature flags, strangler-fig, run old + new in parallel and
   **diff outputs**.
4. **The running old system as an oracle** — record/replay real I/O to
   characterize behavior without a test harness.
5. **Human review + domain knowledge.**

**Anchor safety at the behavioral *seams*, not the unit.** The durable,
stack-agnostic contracts of any app are its externally observable behaviors:
HTTP endpoints, DB schema/queries, file/wire formats, CLI output, protocols. Pin
*those* as golden-master / approval snapshots — **they survive the rewrite; unit
tests on dead legacy code get deleted the moment you replace the module.** Get an
**oracle**, ranked by what's available:

- **Best — it runs *somewhere*** (prod, a VM, an old container, even read-only),
  **and you're allowed to use it**: record real I/O and replay it against the new
  code. One successful capture beats a permanent green legacy CI. *(Do not assume
  this rung is always available — a faithful third-party reimplementation or a
  resurrected legacy binary may be off-limits by policy, licensing, or an
  explicit decision to build "with only what we have." If so, skip to the
  self-frozen rung.)*
- **Fallback — it doesn't run at all:** the old **source is the spec**;
  supplement with prod logs, DB dumps, docs, tickets, and accept that
  characterization tests get written **against the new implementation** to lock
  it once verified correct.
- **Self-frozen golden master — no external reference exists or is permitted:**
  when nothing trustworthy runs *and* you may not borrow an external oracle, the
  **modernized component records its *own* golden master the moment it first runs**
  (at its Testability Milestone), and every later phase diffs against that frozen
  snapshot. This is a real, usable rung — but **name its residual risk out loud**:
  it guarantees the *self-consistency of later refactors*, **not** the correctness
  of the first boot (the snapshot is only as correct as the run that produced it).
  Pair it with human review / spec cross-checks to bless that first run.

##### 2f. Feed an economic/oracle triage into the choice

Weigh **regression cost** when picking a rung: Is the app **in production**? Does
it have **real users**? Is the **old system still runnable** as a reference? An
abandoned app with zero users has near-zero regression cost — an expensive gate
there is **over-engineering**. A production system with users justifies a higher
rung and a real oracle.

##### 2g. Locate the CI Milestone (name the phase that stands up CI)

Just as the Testability Milestone names *when a component can first run and be
tested*, the **CI Milestone** names *when continuous integration is first stood
up* — and it must be **stated out loud in the roadmap, not buried in a phase
body.** A common failure of modernization plans is leaving the reader unable to
tell which phase introduces CI; name it explicitly.

**Rule: CI is stood up in the *first lit phase* — at or immediately after the
component crosses its Testability Milestone — never before.** CI cannot be green
on a component that cannot yet build/run, so wiring a CI *gate* during the dark
regime is the same category error as demanding a test gate there. (Trivial
scaffolding — a lint-only or build-only workflow — may appear earlier, but the
authoritative test gate lands at the CI Milestone.)

**CI is two distinct steps, and the second is human-only** — call both out:
1. **Author the CI workflow file** (e.g. `.github/workflows/*.yml`, `.gitlab-ci.yml`,
   Azure Pipelines) that runs the gate. **An agent can do this.**
2. **Make that workflow an *enforced* gate** — a **required status check /
   branch-protection rule / merge-request approval rule**. This is a manual
   platform-UI / admin step (GitHub → Settings → Branches; GitLab protected
   branches + MR settings; etc.) that **an agent cannot perform.** Until a human
   configures it, CI *runs* on PRs but does **not** *block* merges.

Because step 2 is outside the agent's reach, **every plan that stands up CI must
emit it as an explicit user action item** (in the phase's exit criteria and in
§9 Open questions / decisions needed from stakeholders), so the reader knows the
gate is not self-enforcing until they turn it on.

**Output of Phase 2:** for each component — the spike result, the chosen strategy
(A or B), the located Testability Milestone, the pre/post label, and the target
safety-ladder rung with any residual risk named — **plus the plan-wide CI
Milestone** (which phase stands up CI) and the reminder that enforcing it is a
manual human step. Everything downstream depends on these.

#### Phase 2.5: Red-team every phase against the hazard catalog (mandatory)

The strategic scaffolding above (testability, safety ladder, CI milestone) is
necessary but **not sufficient**. Modernization plans also fail on *tactical,
ecosystem-predictable* hazards — an incomplete quarantine set, a forgotten
namespace codemod, a runtime bump that leaves the base images behind, a stateful
data-store major treated as an image bump, an edge rewrite that drops the
anonymous route class. These are so recurring that they have their own catalog:
**`references/migration-hazards.md`** (H1–H8).

**Rule: before any phase is implemented, red-team its plan against every hazard
in `references/migration-hazards.md`.** For each hazard, ask "does this phase
trigger it?", run the hazard's detection probe against the real repo, and fold
its plan action into the phase's tasks/exit criteria **before** writing code. A
hazard caught in planning is a task; caught in review it's rework; caught in prod
it's an incident. Record hazards you checked and **cleared**, not just the ones
that fired — a silent skip is indistinguishable from a miss.

This is the institutionalized form of an independent-critic pass. Do it yourself
by walking the catalog, and — where an independent second opinion is available
(e.g. a rubber-duck / critique agent) — use it: on the reference PiggyMetrics run
such a pass caught a **blocking plan gap in every single phase**, and each gap
was an instance of an H1–H8 class. Bake the outcome into each phase's
"Decisions made" and exit criteria so the executing agent inherits the cleared
checklist rather than re-discovering the hazard.

#### Phase 3: Recommend Target Architecture

**Decision framework — always evaluate in this order:**

For each component/dependency, work through these levels from least-disruptive to
most-disruptive. Stop at the first level that solves the problem:

1. **Upgrade in place** — Can you bump the major version and fix breakages?
   (e.g., Node 14→22, Python 2→3, React 16→19, Rails 5→7). This is the default
   answer. Only reject it if the upgrade path is officially unsupported, the
   breaking changes are so extensive they exceed rewrite cost, or the framework
   itself is abandoned/EOL.

2. **Swap the dependency** — If the specific library is dead but the pattern is
   fine, can you swap to a maintained alternative with the same interface shape?
   (e.g., Moment.js → date-fns, Request → got/fetch, CoffeeScript → TypeScript
   with decaffeinate). Prefer drop-in or mechanical migration tools when they
   exist.

3. **Wrap/adapt (Strangler Fig)** — If the component's interface is sound but
   internals are unmaintainable, can you put a clean interface in front and
   incrementally replace internals behind it? This works when coupling is
   manageable and the system can run with old and new coexisting.

4. **Rewrite** — Only when: the framework is abandoned with no upgrade path, the
   architecture fundamentally cannot support required new capabilities (e.g.,
   synchronous-only design that must become real-time), or honest estimation shows
   upgrade cost exceeds rewrite cost. Require explicit justification for every
   rewrite recommendation — "it's old" is not sufficient.

5. **Remove** — Dead code, features with zero usage, deprecated capabilities
   superseded by other systems. Verify usage claims before recommending removal.

**Bias toward conservatism.** The cheapest migration is the one you don't do.
Every level of disruption adds risk, timeline, and cost. A working system on an
older framework is better than a half-finished rewrite on a new one.

**Mechanical-migration riders (apply whenever a recommendation is an upgrade,
swap, or removal).** These are the parts of a target-architecture decision that
first-draft plans routinely drop; each maps to a hazard class in
`references/migration-hazards.md`:

- **Transitive-quarantine completeness (H1).** When removing/cutting over a whole
  dependency family, grep the dead artifact across **all** manifests and list the
  *full* set of dependents that must move or be quarantined — not the subset you
  first thought of. State the post-cutover build target explicitly.
- **Framework-major codemods (H2).** A **major** bump carries mechanical breaking
  changes independent of your logic — namespace renames, removed deprecated APIs,
  a changed **test engine**, changed config keys. Enumerate each as its own task
  and prefer the ecosystem's automated **recipe/codemod tool** (OpenRewrite,
  jscodeshift/react-codemod, `2to3`/`pyupgrade`, etc.) over hand-editing on large
  trees. Don't forget the test-framework migration — it is the most-forgotten one.
- **Runtime↔deployment lockstep (H3).** If a recommendation bumps the
  language/runtime **major**, every place the runtime is pinned for *running* the
  app — base images, `docker-compose` build contexts, CI runner images,
  `engines`/`.nvmrc`/`.tool-versions`, serverless runtimes — must move in the
  **same phase**. Build-runtime and run-runtime must never drift across a phase
  boundary, or the app builds green and then fails to run.

Produce a **Target Architecture** section that includes:

- **Recommended tech stack** with rationale for each choice (framework, language,
  runtime, database, infrastructure)
- **Architectural pattern** recommendation (monolith → modular monolith →
  services, event-driven, serverless, etc.) with justification
- **What stays vs. what goes** — explicitly categorize each major component using
  the decision framework above:
  - ✅ Keep as-is (still modern/adequate)
  - ⬆️ Upgrade in place (bump version, fix breakages)
  - 🔀 Swap dependency (replace dead library with maintained equivalent)
  - 🔄 Wrap/adapt (add an interface layer, incrementally replace internals)
  - 🔁 Rewrite (fundamentally incompatible — justification required)
  - 🗑️ Remove (dead code, deprecated features, superseded)

For each recommendation, write an inline ADR:

```markdown
#### ADR: [Decision Title]
- **Context:** [Why this decision is needed]
- **Decision:** [What we chose and which decision framework level it falls under]
- **Alternatives considered:** [What else was evaluated and why it lost]
- **Consequences:** [Tradeoffs accepted]
```
