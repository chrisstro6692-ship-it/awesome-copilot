# Phases 5–7: Implementation Plan, Governance, and Safety Net

#### Phase 5: Phased Implementation Plan

Produce an ordered, phased plan. Each phase should be independently deployable
(no half-migrated states that can't run in production).

**Phase gating is regime-aware (applies to every phase).** State this rule at the
top of the phased plan and honor it in every phase's exit criteria. A phase is
**not complete** until its Verification & Exit Criteria pass. Those criteria must
be (a) **objectively verifiable**; (b) **actually executed and recorded** before
the next phase starts; and (c) **gated** — do not advance to phase N+1 until
phase N's criteria are demonstrably met. **But which criteria are valid depends
on the component's regime (Phase 2c):**

- **Post-testability ("lit") phases:** exit criteria are runnable commands / green
  CI — **green CI on the phase's branch/PR is the authoritative signal.**
- **Pre-testability ("dark") phases:** the component isn't alive yet, so a green
  CI test gate is a **category error**. Exit criteria come from the achievable
  safety-ladder rung instead — captured seam/oracle snapshots, reversibility
  scaffolding proven present, a passed smoke checklist, recorded review. Do **not**
  block a dark-regime phase on a test suite that can't run yet.

Report to the user any phase whose pass/fail is unknown rather than assuming it
passed.

**The first phase establishes maximum *achievable* safety — not a fixed "green
legacy gate."** Do not hard-code "resurrect everything and make CI green." Shape
the first phase from the Phase 2 outputs, per component:

- **Strategy A (Freeze-then-lift), component is resurrectable:** the first phase
  *is* the classic Safety Net & Baseline — purely additive, no behavior change:
  - **Characterization / golden / contract tests** at the **seams** that pin
    current observable behavior (protocol traces, API snapshots, output goldens).
  - **CI + lint + formatter** wired up and green (an authoritative gate for later
    lit-regime phases).
  - **Pinned dependencies / committed lockfile** and a recorded known-good
    baseline (versions, toolchain, how it builds and runs today).
  - **"Net proven to fail."** Deliberately mutate the code (e.g., change one
    protocol code or return value), confirm the new tests go **red**, then
    revert. A safety net that never fails is proof of nothing — prove it has
    teeth.
- **Strategy B (Beachhead/walking-skeleton), component is dead:** the first phase
  is **"reach the Testability Milestone with the smallest slice"** — drive the
  thinnest end-to-end path onto the modern stack until it builds, boots, and runs
  ≥1 real test green, capturing seam/oracle contracts from the old system to
  validate against. **Do not try to net the corpse.** Its exit criterion is the
  achievable safety-ladder rung with residual risk named — a **blessed downgrade
  is not a failure.**

Only after the first phase's criteria are met (at whatever rung) do the
subsequent infrastructure and feature phases begin.

Structure each phase as:

```markdown
### Phase N: [Name] (T-shirt size: M)

**Goal:** [One sentence]
**Regime:** [pre-testability ("dark") | post-testability ("lit")] — per component
**Safety rung:** [L0–L4, with residual risk named if below L4]
**Prerequisites:** [Which phases must complete first]
**Duration estimate:** [Relative, not calendar — e.g., "2-4 sprints"]

#### Tasks
| ID | Task | Component | Blocked by |
|----|------|-----------|------------|
| N.1 | ... | ... | — |
| N.2 | ... | ... | N.1 |

#### Risks & Mitigations
- **Risk:** ... → **Mitigation:** ...

#### Decisions made
- [Every sub-decision this phase depends on, resolved and documented here so the
  phase can be implemented without further user input. State "dropped" vs
  "deferred" explicitly for anything cut.]

#### Verification & Exit Criteria (Definition of Done)
- [ ] [Regime-appropriate criterion. For LIT phases: a runnable command / green
      CI check, citing the actual command from the Commands & Verification
      Inventory. For DARK phases: the achievable safety-ladder rung's evidence —
      captured seam/oracle snapshots, reversibility proven, smoke checklist
      passed — NOT a green test suite the component can't yet run.]
- [ ] [Include a parity/characterization check against the chosen oracle whenever
      the change is meant to preserve behavior.]
- [ ] [If purely additive, assert it: "no behavior/dependency/logic changed."]
- [ ] [If exiting below L4, state the residual risk and which later phase closes
      it (e.g., "client test gate deferred to the client-migration phase").]
```

**Ordering principles:**
- **Locate the Testability Milestone first**, per component — schedule the
  smallest slice that reaches it early rather than deferring "testable" to the end
  while demanding a gate at the start.
- **Never require a component's automated test gate before it crosses its own
  testability line.**
- **Name the CI Milestone in the roadmap.** State explicitly which phase stands
  up CI (the first *lit* phase, at/after the Testability Milestone) so the reader
  never has to guess when CI arrives — and flag that *enforcing* it (required
  status check / branch protection) is a manual human step, not an agent task.
- **First-phase safety before behavior change** — establish the highest
  achievable rung (not necessarily a full green gate) before modifying behavior.
- Infrastructure and cross-cutting concerns next (auth, config, logging, CI/CD)
- Highest-risk or highest-value features next (prove the pattern early)
- Low-risk, low-coupling features last (easy wins to parallelize)
- Data migrations get their own phase with rollback plans

#### Phase 6: Execution Governance

Modernization plans fail in execution, not on paper. Bake in the governance that
keeps each phase honest and reversible (generic across any ecosystem):

- **Branch per phase.** Never commit phase work to the default branch. Open one
  PR per phase; let CI on the PR be the authoritative green signal before merge
  **for lit-regime phases**. For dark-regime phases, the PR still carries the
  rung's evidence (captured contracts, smoke results, reversibility scaffolding).
- **Merge to trunk before the next phase; never stack (H7).** Each phase branch
  is cut **from trunk**, and its PR is **merged to trunk before the next phase
  starts**. Never base a phase branch or its PR on a *sibling* phase branch — that
  is how PRs get merged into each other, the trunk silently stalls several phases
  behind, and later phases are forced into the exact bind where the no-stacking
  rule has to be broken. Before starting a phase, verify the previous phase is
  merged to trunk and `git log origin/<trunk>..HEAD` is empty at branch creation.
  **If controlled stacking is genuinely unavoidable, it is allowed only with a
  required reconciliation PR** that lands the stack onto trunk **and an explicit
  residual-risk note** — never as a silent default.
- **Confirm the trunk name; retire legacy defaults.** Pin the repo's real trunk
  (e.g. `main`) and, if a second default-ish branch exists (`master`), mark it
  "history only — do not target" so phase work never lands on the wrong branch.
- **The gate matches the regime.** Lit phases advance on green CI; dark phases
  advance on the achievable safety-ladder rung with residual risk named — not on
  a "looks done" judgement, and not on a test suite that can't run yet.
- **Stand up CI at the CI Milestone, and hand off enforcement to a human.**
  Name the phase that introduces CI (the first lit phase). Distinguish the two
  steps: authoring the workflow file is agent-doable; making it an **enforced
  required status check / branch-protection (or MR-approval) rule is a manual
  platform-UI step the agent cannot perform.** Until a human configures it, CI
  *runs* but does not *block* merges — so record "enable branch protection /
  required checks" as an explicit user action item (in the CI phase's exit
  criteria and in §9 stakeholder decisions), never as a completed agent task.
- **Interface-preserving & independently deployable.** Each phase leaves the
  system runnable, so rollback = redeploy the previous version. Prefer
  interface-preserving rewrites (wrap internals behind a stable interface) and
  prove equivalence with the seam/oracle contracts (golden/characterization
  tests where they exist).
- **Living plan.** Update `MODERNIZATION_PLAN.md` status markers as each phase's
  exit criteria are met — ✅ complete / ⏭️ descoped / 🗑️ dropped — record the
  decisions made during implementation, and **update the safety-ladder rung /
  residual-risk register** as components cross their testability lines.
- **Keep the executable docs alive too (H8).** The living-plan discipline is not
  only the plan file. Any phase that changes **topology** — module/reactor
  membership, removed services, renamed branches, changed commands, new/removed
  endpoints — must update `.github/copilot-instructions.md`, the `README`, and any
  module/topology list **in the same PR**. Treat those doc updates as part of the
  phase's Definition of Done, not a follow-up — a stale "quarantined module" list
  or a hard-coded old-branch link actively misleads the next agent/human.
- **Red-team each phase before implementing it (Phase 2.5).** Walk the phase plan
  against `references/migration-hazards.md` (H1–H8) and fold the fixes into tasks
  and exit criteria before writing code. Record the cleared checklist in the
  phase's "Decisions made".
- **Emit editable instructions.** Produce a `.github/copilot-instructions.md`
  (create the `.github/` directory if needed) from the template at
  `references/copilot-instructions.template.md` — a commands table plus the
  regime-aware phase-gate and branch/PR rules — so the executing agent/human has
  the canonical commands and gates in the location GitHub Copilot auto-loads. The
  user edits it to their own gates. If a `.github/copilot-instructions.md` already
  exists, do not overwrite it — merge the commands/gating/branch sections in, or
  write the generated file alongside as
  `.github/copilot-instructions.modernization.md` and tell the user to merge.

#### Phase 7: Migration Safety Net

Document the operational guardrails needed:

- **Feature flags** — what gets flagged and how old/new coexist
- **Data migration strategy** — schema changes, backfills, dual-write periods.
  For a **stateful data-store major upgrade** with persisted volumes (H5), an
  engine image/tag bump is **not** a migration: pick and document one path —
  a sequential/feature-compatibility-version upgrade with steps + rollback, **or**
  an explicit **destructive reset** (drop volumes, re-seed) when the data is
  ephemeral/demo — stated as a deliberate decision, with the prior-major image
  retained for rollback. (e.g. Mongo 3→7 over an existing `/data/db` volume needs
  `down -v` + re-seed for a demo, or a staged FCV upgrade for real data.)
- **Rollback plan** — per phase, how to revert if something fails
- **Transitional-insecure-state register (H6)** — every deliberately-weak state
  introduced to keep the system runnable mid-migration (permit-all shims, CSRF
  disabled, an open endpoint, a placeholder secret) listed with: why it's needed,
  the phase that **closes** it, the residual risk, and a by-design-until-phase-N
  note for reviewers. Scope each weakening as tightly as possible (e.g. a
  permit-all chain should still explicitly disable form-login/HTTP-Basic).
  Pre-declaring these converts recurring scanner/reviewer churn into a tracked
  decision.
- **Oracle & seam contracts** — which running instance / recorded I/O / golden
  snapshots (or a **self-frozen golden master**, when no external reference is
  available or permitted) serve as the behavioral reference, and how new work is
  diffed against them
- **Testing strategy** — what new test coverage is needed (built on the *target*
  stack as components cross their testability line), and what stays quarantined
- **Observability** — metrics/alerts that prove the new system matches the old
