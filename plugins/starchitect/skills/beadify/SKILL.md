---
name: starchitect:beadify
description: >
  Turn a plan into a complete, self-documenting implementation task graph in bd (beads).
  Captures every FR, AC, TR, contract, seam, journey, and artifact from the planning documents
  into beads with rich background, rationale, and acceptance criteria — including the wiring,
  test-harness, and integration-verification work that makes the feature actually run. Reads
  the feature index, then lazily loads feature PRDs, floorplan, contracts, TDDs, technology
  choices, and artifacts as needed.
  Triggers: "beadify", "taskify", "create tasks", "break into tasks", "implementation tasks",
  "task hierarchy", "decompose into tasks".
user-invocable: true
---

# beadify: Plan to Implementation Task Graph

Convert a plan into a `bd` (beads) task graph that a team of agents can execute to arrive at a **running, verified, shippable** system — not a pile of implemented-but-unwired modules.

Your output is `bd` issues: epics, features, tasks, and subtasks with structured fields, typed dependencies, per-bead narrative comments, and traceability back to every identifier in the plan.

<HARD-GATE>
Do NOT skip to creating beads. Every task graph must be presented to the user for review and confirmation before writing to `bd`. Do NOT load all documents upfront — load lazily as each feature is visited. Do NOT emit `test:*` labels; those belong to the test-plan skill.
</HARD-GATE>

## Reasoning Budget

**This skill requires your maximum reasoning depth.** Decomposition errors here are expensive and invisible: a missing wiring bead does not fail loudly, it produces a codebase where every module is "done" and nothing works. The cost of thinking harder is minutes; the cost of thinking less is days of integration archaeology.

- **Claude Code / Claude:** engage `ultrathink`-level extended thinking for the whole session.
- **Other hosts:** set the highest available thinking or reasoning-effort level before starting.
- If your host exposes no such control, slow down deliberately: write out the ledgers (Phase 2) in full before proposing a single bead.

If you were invoked without an extended-thinking directive, say so once at the start — "This skill works best with maximum thinking enabled; I'm running at the deepest level available to me" — and continue. Do not stop to ask.

Three points demand the most thought, and each is marked **[THINK DEEPLY]** below:

1. Building the **seam ledger** (Phase 2) — the step that closes the integration gap.
2. **Decomposing** each ledger row into bead layers (Phase 3).
3. Wiring the **dependency graph** (Phase 5) — where false parallelism hides.

## The Completeness Contract

Every run must satisfy four invariants. State them to the user at the start, and check each one explicitly in Phase 8.

1. **Nothing in the plan is lost.** Every identifier in scope (CAP, P, UC, UJ, FR, AC, G, NG, COMP, BD, DF, SL, ENT, API, EVT, TR, ART) is either represented in a bead or explicitly recorded as out of scope with a reason.
2. **Nothing is unreachable.** Every piece of implemented behavior is wired into a composition root, a route table, a subscriber registry, a CLI surface, a build graph — some path by which real execution reaches it.
3. **Nothing is unverified.** Every seam has an automated test that exercises both sides for real. Every acceptance criterion has a bead that proves it. Every artifact has a bead that builds and runs it.
4. **Nothing is unexplained.** Every bead carries enough background, rationale, and context that an agent with no memory of the plan can execute it correctly from the bead alone.

## Autopilot Mode

When invoked with the word **"autopilot"** (e.g., "beadify on autopilot"), all confirmation gates below become **soft**: the skill still presents its output at each checkpoint but proceeds immediately without waiting for user confirmation. The user can interrupt at any point to adjust.

Autopilot does NOT skip content — it skips waiting. You still show your work at every gate, and you still run every validation rule.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover & scope** — check for beads, inventory the plan documents, pick the scope, name what's missing
2. **Load the plan** — lazily read the documents for the chosen scope
3. **Build the ledgers** — capture ledger, seam ledger, journey & artifact ledger **[THINK DEEPLY]**
4. **Decompose into bead layers** — turn each ledger row into beads **[THINK DEEPLY]**
5. **Write bead content** — self-documenting descriptions, design, acceptance criteria, notes, comments
6. **Wire the graph** — dependencies, ordering, parallelism, gates **[THINK DEEPLY]**
7. **Review & commit** — present per-feature, get confirmation
8. **Write to bd** — graph plan, comments, verification
9. **Validate** — run every validation rule, report pass/fail
10. **Handoff** — tell the user what to run next

---

## Phase 0: Discover & Scope

### Check for beads

Check for a `.beads/` directory in the project root.

If not found:
- Tell the user: "No beads workspace found. Run `bd init` to initialize one, then re-run this skill."
- Stop here.

Do not run `bd init` yourself — prefix and storage mode are the user's choice.

### Inventory the plan documents

Use Glob to check every location. Read nothing yet except the feature index.

| Document | Locations to check | What beadify needs from it |
|----------|--------------------|---------------------------|
| **Feature index** | `docs/features/index.org`, `docs/features/index.md` | Epic/feature structure (EP, FT), dependency graph, phases |
| **Feature PRDs** | `docs/features/<slug>.{org,md}` | FRs, ACs, UCs, scope boundaries |
| **Root PRD** | `docs/prd.{org,md}` | CAP, P, UC, UJ, G, NG — context and journeys |
| **Floorplan** | `docs/floorplan.{org,md}` | COMP, BD edges, DF flows, SL sequences |
| **Contracts** | `docs/contracts.{org,md}`, `docs/contracts/` | ENT schemas, API operations, EVT events |
| **TDDs** | `docs/tdd.{org,md}`, `docs/tdd/` | TR statements, algorithms, trade-offs, deferred improvements |
| **Technology** | `docs/technology.{org,md}` | Languages, frameworks, test tooling, CI |
| **Artifacts** | `docs/artifacts.{org,md}`, `docs/artifacts/` | ART catalog, source paths, packaging, signing |

Report the inventory to the user as a table with ✅ found / ❌ missing.

**Missing documents are gaps, not blockers.** Name the consequence of each so the user can decide:

| Missing | Consequence you must state |
|---------|---------------------------|
| Feature index | No epic/feature structure — offer the ad-hoc entry path below |
| Floorplan | No component boundaries or BD edges — the seam ledger will be incomplete and wiring beads will be guesses |
| Contracts | No ENT/API/EVT — no contract-types beads and no seam verification targets |
| TDD | No TRs — non-functional behavior (performance, concurrency, recovery) will go unverified |
| Technology | No test tooling or CI target — harness beads will name no concrete framework |
| Artifacts | **Nothing will be shippable** — no build, package, or install beads; the epic completes with no runnable output |

For each missing document ask once: "Proceed without it, or run `<skill>` first?" In autopilot, proceed and record the gap in the epic's Background & Intent comment.

### Ad-hoc entry path

If there is no feature index but the user has a plan (a design doc, a written brief, an issue, or a verbal description), you can still run:

1. Read or transcribe the plan into a scratch file so you can cite it.
2. Synthesize the minimum structure: one epic, plus a feature for each coherent chunk of work.
3. Derive pseudo-identifiers for traceability (`FR-adhoc-1`, `COMP-adhoc-api`) and record in the epic comment that they are synthesized, not from a PRD.
4. Run every remaining phase unchanged. The seam ledger matters **more** here, not less — an ad-hoc plan has no floorplan to cross-check against.
5. At handoff, recommend running the missing starchitect skills to backfill real identifiers.

### Propose and confirm the project slug

Every bead this skill creates carries a `proj:<slug>` label so the whole project can be filtered as a unit.

**Step 1 — generate a candidate.** Derive from, in order of preference: the project name at the top of the feature index; the product name in the PRD; the repository directory name. Slugify: lowercase, hyphens for spaces, no special characters, shortened (e.g., "Acme Internal Customer Portal" → `acme-portal`).

**Step 2 — find existing slugs:**

```bash
bd list --json | jq -r '.[].labels[]? | select(startswith("proj:"))' | sort -u
```

**Step 3 — present options.** Offer the generated candidate alongside any existing `proj:*` slugs. If one of the existing slugs clearly matches the work at hand, recommend it. Otherwise let the user choose or supply their own.

<HARD-GATE>
Do NOT proceed until the user has confirmed the slug — a wrong slug makes every downstream query wrong. Remember it for the whole session; it goes on every bead.

**Autopilot:** present the options, then proceed with the recommended slug.
</HARD-GATE>

### Determine what's already beadified

For each epic and feature in the index:

```bash
bd query --label proj:<slug> --label ep:<EP-id> --json
bd query --label proj:<slug> --label ft:<FT-id> --json
```

Classify each feature:

- **Not started** — no beads. Full decomposition.
- **Partial** — beads exist but the Phase 8 validation rules would fail (for example, impl beads with no wiring, harness, or verification beads). **This is the common case for features beadified by an earlier version of this skill.** Offer a **completion pass**: add the missing wiring, harness, verification, and gate beads without duplicating existing impl beads.
- **Complete** — beads exist and all validation rules pass. Skip unless the user asks for a re-pass.

To judge "partial" quickly, count beads by kind:

```bash
bd query --label proj:<slug> --label ft:<FT-id> --json \
  | jq -r '.[].labels[]? | select(startswith("kind:"))' | sort | uniq -c
```

A feature with `kind:impl` beads but no `kind:wiring` and no `kind:verify-*` beads is exactly the failure mode this skill exists to fix.

### Present scope and get confirmation

Show the user:

- The document inventory, with gaps and their consequences
- The epic/feature table with per-feature status (not started / partial / complete)
- Recommended scope: one epic at a time, in the index's dependency order, plus which epics it unblocks
- Estimated bead count for the scope (use the sanity bands in Phase 3)

<HARD-GATE>
Do NOT load feature documents until the user has confirmed the scope.

**Autopilot:** present the recommendation, then proceed.
</HARD-GATE>

---

## Phase 1: Load the Plan

For the chosen scope, load lazily — only what the current feature needs. Do not read whole documents when an identifier lookup will do.

**Load once for the epic:**
- Root PRD sections for the UJs and UCs this epic serves — **journeys are epic-level, not feature-level**, and they are what ledger C is built from
- Floorplan COMP definitions, BD edges, DF flows, and SL sequences touching this epic's components
- Artifacts entries whose producing COMP is in this epic
- Technology entries for the epic's languages, frameworks, test tooling, and CI

**Load per feature:**
- The feature PRD (FRs, ACs, UCs, scope boundaries, feature dependencies)
- Contract entries (ENT/API/EVT) named by those FRs or by the BD edges the feature crosses — load the contracts index first for identifier lookup, then the specific detail files in `docs/contracts/`
- TDD sections for the feature's subject: TR statements, algorithms, error handling, trade-offs, deferred improvements, sub-TDD pointers

**Cache what you load.** Features in the same epic share contracts, floorplan, and technology context; don't re-read.

**Record what you could not find.** If an FR names an API that isn't in the contracts doc, or a TDD names a TR with no measurable threshold, that is a hole in the plan. Surface it now — do not paper over it in Phase 3.

---

## Phase 2: Build the Ledgers **[THINK DEEPLY]**

This is the phase that closes the integration gap. Before decomposing anything, write out three ledgers. They are working artifacts: show them to the user, but do not commit them to the repo.

### Ledger A: Capture ledger

One row per identifier in scope. This ledger enforces invariant #1 — nothing lost.

| ID | Kind | Source | Statement (abbreviated) | Disposition |
|----|------|--------|------------------------|-------------|
| FR-12.1 | requirement | features/auth.org | "System MUST validate credentials against…" | → beads |
| FR-12.2 | requirement | features/auth.org | "…MUST issue a signed session token" | → beads |
| AC-4 | acceptance | features/auth.org | "Given an expired token, when…, then 401" | → verify bead |
| TR-7 | tech req | tdd/auth.org | "Token validation MUST complete in <5ms p99" | → verify-runtime bead |
| ENT-3 | entity | contracts/entities.org | Session {id, user_id, expires_at} | → migration + fixture |
| API-9 | operation | contracts/api-auth.org | POST /sessions | → provider + consumer + seam |
| EVT-2 | event | contracts/events.org | session.revoked | → publisher + subscriber + seam |
| NG-2 | non-goal | prd.org | "No SSO in v1" | out of scope — recorded |

Enumerate **every** identifier. Multi-part FRs get one row per sub-item: an FR that says "MUST validate, persist, and emit" is three rows, because it is three pieces of work and three things that can be missed.

Dispositions must be one of:

| Disposition | Meaning |
|-------------|---------|
| `→ beads` | Becomes implementation work |
| `→ verify bead` | Becomes verification work |
| `out of scope — <reason>` | Deliberately excluded; reason recorded on the epic bead |
| `→ spike` | An unknown blocks decomposition |
| `→ decision` | The plan left a choice open |
| `→ deferred` | A TDD deferred improvement, with its trigger condition |

A row with no disposition is an unfinished ledger. Do not proceed with blanks.

### Ledger B: Seam ledger

**This is the most important table in this skill.** A seam is any place where two things must meet. Features fail to work not because components are wrong, but because seams were never made live and never tested.

For every seam, answer two questions:

> **Which bead makes this seam live?** — real code on both sides, running through real config, no placeholder.
>
> **Which bead proves this seam?** — an automated test that exercises both sides with no mocked counterpart *at the seam itself*.

**Enumerate seams from every one of these sources.** Walk the list; do not sample it.

| Seam source | How to enumerate |
|-------------|------------------|
| **BD edges** (floorplan) | Every edge between two in-scope COMPs |
| **API operations** (contracts) | Each operation twice: the provider side, and each consumer side separately |
| **EVT events** (contracts) | Each event: the publisher side, and each subscriber separately |
| **DF crossings** (floorplan) | Every point where a data flow crosses a COMP boundary |
| **SL sequences** (floorplan) | Every message in a swim lane, plus the ordering constraint itself |
| **Persisted ENTs** (contracts) | Schema ↔ migration ↔ mapper/ORM ↔ query path |
| **External dependencies** (technology) | Every third-party service, SDK, or system API |
| **Process / module boundaries** | IPC, FFI, worker queues, dynamic loading, plugin registries |
| **Artifact boundaries** (artifacts) | Build output ↔ packaging ↔ installation ↔ runtime environment ↔ launch mechanism |
| **Config & secret boundaries** | Every config value: where it is defined, defaulted, validated, and consumed |
| **Auth / identity boundaries** | Every place a principal crosses a trust boundary |
| **UI ↔ backend boundaries** | Every view that reads or writes server state |

Format:

| Seam | Sides | Live via | Proven by | Notes |
|------|-------|----------|-----------|-------|
| API-9 provider | api-svc → session handler | `impl-api9-handler` | `verify-seam-api9` | |
| API-9 consumer | web-ui → api-svc | `wiring-web-apiclient` | `verify-seam-api9` | client needs base-URL config |
| EVT-2 publish | auth-svc → bus | `impl-evt2-publish` | `verify-seam-evt2` | |
| EVT-2 subscribe | notifier ← bus | `wiring-notifier-sub` | `verify-seam-evt2` | subscriber registration is the usual gap |
| ENT-3 persist | session store → db | `ops-migrate-session` | `verify-seam-ent3` | needs a fixture in the harness bead |
| BD-4 | cli → auth-svc | `wiring-cli-authclient` | `verify-journey-uj1` | |
| CFG auth.token_ttl | config → validator | `ops-config-schema` | `verify-seam-cfg-ttl` | default must be documented |
| ART-2 install | build → user machine | `impl-art2-package` | `verify-artifact-art2` | signing deferred to v2 (`kind:deferred`) |

**A seam proven only by a test that mocks the other side is not proven.** Write the real-counterpart test as the proving bead. If a real counterpart is genuinely unavailable, record it in the stub ledger (Phase 3) with a replacing bead — never leave the mock as the answer.

<HARD-GATE>Do not proceed past ledger B with any blank in the "Live via" or "Proven by" column. A blank is a hole in the plan; fill it with a bead, a spike, or an explicit out-of-scope note.</HARD-GATE>

### Ledger C: Journey & artifact ledger

Seams prove that pieces connect. Journeys prove the product works. Artifacts prove it ships.

| Item | Type | Path through the system | Demonstrated by | Runnable output |
|------|------|------------------------|-----------------|-----------------|
| UJ-1 | journey | cli → auth-svc → db → cli | `verify-journey-uj1` | `make demo-login` |
| UC-3 | use case | web-ui → api-svc → bus → notifier | `verify-journey-uc3` | integration suite |
| ART-1 | artifact | build → `dist/app` | `verify-artifact-art1` | `make build && ./dist/app --version` |
| ART-2 | artifact | build → `.deb` → installed systemd service | `verify-artifact-art2` | container install smoke test |

Every in-scope UJ and UC needs a row. Every ART whose producing COMP is in scope needs a row.

**If ledger C is empty, the epic will produce nothing a human can run.** Say so plainly before continuing, and recommend running the `artifacts` skill.

### Present the ledgers

<HARD-GATE>
Show all three ledgers to the user before decomposing. Call out explicitly: any ledger A row without a disposition, any ledger B blank, any UJ or ART with no demonstration.

**Autopilot:** present them and continue — but still enumerate the blanks in your message, and create `kind:spike` or `kind:decision` beads for them rather than silently filling them in.
</HARD-GATE>

---

## Phase 3: Decompose Into Bead Layers **[THINK DEEPLY]**

### The bead kind taxonomy

Every bead carries exactly one `kind:*` label. This is beadify's namespace; `test:*` belongs to test-plan (see "Interaction with test-plan").

| Label | bd type | Purpose | Single-COMP rule |
|-------|---------|---------|------------------|
| `kind:spike` | `spike` | Resolve an unknown that blocks design | n/a |
| `kind:decision` | `decision` | Record a choice the plan left open | n/a |
| `kind:scaffold` | `chore` | Create module/package/directory structure | **scoped** |
| `kind:contract-types` | `task` | Generate or hand-write types from ENT/API/EVT | **scoped** |
| `kind:harness` | `chore` | Test infrastructure: runner, fixtures, seed data, CI job | **exempt** |
| `kind:skeleton` | `task` | Walking skeleton — the thinnest end-to-end path | **exempt** |
| `kind:impl` | `task` | Real behavior inside one component | **scoped** |
| `kind:wiring` | `task` | Make a seam live | **exempt** |
| `kind:verify-seam` | `task` | Prove one seam, both sides real | **exempt** |
| `kind:verify-journey` | `task` | Prove a UJ or UC end to end | **exempt** |
| `kind:verify-artifact` | `task` | Prove an ART builds, installs, and runs | **exempt** |
| `kind:verify-runtime` | `task` | Prove a non-functional TR (performance, concurrency, recovery) | **exempt** |
| `kind:ops` | `task` | Migrations, config, secrets, observability, deploy | **exempt** |
| `kind:gate` | `task` / `milestone` | Definition-of-done checkpoint for a FT or EP | **exempt** |
| `kind:deferred` | `task` | Deferred improvement, with its trigger condition | n/a |
| `kind:docs` | `chore` | README, runbook, or ADR the plan calls for | **scoped** |

### The single-COMP rule, corrected

Implementation beads stay inside one component so agents don't collide on files. **But integration work is inherently cross-component, and forbidding it is precisely why plans produce unwired code.**

- **Scoped** kinds touch files in one COMP only. Label with a single `comp:<COMP-id>`.
- **Exempt** kinds may touch multiple COMPs by design. Label with **every** `comp:<COMP-id>` they touch, so the overlap is visible even though it is permitted.

Exempt beads *are* the seams. Sequence them so two exempt beads touching the same file never run concurrently (Phase 5).

### Walking skeleton first

For each feature — or for the epic, when a single feature is too small to stand alone:

> The **first** substantial bead after scaffolding is a walking skeleton: the thinnest possible path that runs end to end through every COMP the feature touches, returning a hardcoded or trivial result.

The skeleton exists to make the seams live before any real logic lands. Once it runs, every later impl bead has somewhere real to plug into, and every wiring bead extends an existing composition root rather than inventing one.

A skeleton bead's acceptance criteria always include a command a human can run and an observable output.

### The wiring checklist

For each feature, walk this list explicitly. Every item that applies becomes a `kind:wiring` bead (or a named line item inside one). Record in the feature bead's comment which items you determined do **not** apply, and why — that record is what makes the omission reviewable instead of accidental.

1. **Composition root / DI registration** — the new type is constructed and injected where it is needed
2. **Route / handler / endpoint registration** — the handler is reachable from the server's route table
3. **Event subscriber registration** — subscribers are registered; topics, queues, and streams exist
4. **Migration registration and run order** — the migration is in the migration list, in the right position
5. **Config schema, defaults, and plumbing** — every new config value is declared, defaulted, validated, and threaded to every consumer
6. **Secret / credential provisioning** — where the secret comes from in dev, CI, and production
7. **Client construction** — for each API consumer: base URL, timeouts, retries, auth, error mapping
8. **Serialization registration** — codecs, custom type handlers, schema-registry entries
9. **Feature exposure** — CLI subcommand, menu item, nav entry, exported symbol, public API surface
10. **Startup / shutdown ordering and health checks** — the component starts in the right order and reports readiness
11. **Cross-boundary glue** — IPC channel, FFI binding, worker registration, plugin manifest entry
12. **Error mapping at the boundary** — domain error → wire error → user-visible message
13. **Build wiring** — new module added to the build graph, packaging manifest, entry point, dependency declaration

### The harness checklist

Test infrastructure is **beadify's job**, not test-plan's. test-plan specifies *what* to test; without a harness there is nothing for the tests to run in. For each feature — or shared across the epic, since one harness bead can serve many features — walk this list:

1. **Test runner configuration** for each level the feature needs
2. **Real dependency provisioning** — containers, in-memory servers, temp dirs, local brokers
3. **Fixtures / factories** for every persisted ENT the feature touches
4. **Seed data** and migrate-on-test-start
5. **Determinism controls** — injectable clock, ID generator, seeded randomness
6. **Identity doubles at the edge only** — test auth may be faked at the outermost boundary, never at an internal seam
7. **Network policy** — loopback permitted, external calls blocked and loudly failing
8. **Contract assertion helpers** — reusable assertions for the wire shapes in ENT/API/EVT
9. **Teardown and isolation** — tests do not leak state into each other
10. **CI job** — each test level runs in CI, with logs, coverage, and traces retained as job artifacts
11. **One-command local runner**, documented in the harness bead's acceptance criteria, so the next agent can run the suite without archaeology

Harness beads are `kind:harness`, type `chore`, and they **block** every verify bead that depends on them.

### Verification beads

- One `kind:verify-seam` bead per ledger B row (or per tightly-coupled pair of rows, e.g. an operation's provider and consumer sides proven by the same test).
- One `kind:verify-journey` bead per ledger C journey or use case.
- One `kind:verify-artifact` bead per ledger C artifact.
- One `kind:verify-runtime` bead per non-functional TR, with the TR's threshold inlined verbatim and the measurement method named.

**The no-mocks-at-the-seam rule:** a verify-seam bead's acceptance criteria must state that both sides run real code. If the test mocks the counterpart, it verifies nothing about the boundary. The only acceptable doubles are the outermost external service — with a separate contract test against the real one — and the clock.

Verify beads are exempt from the single-COMP rule and are blocked by:
- the impl beads for each side,
- the wiring beads that make the seam live,
- the harness bead that provides the environment.

### The stub ledger

Sometimes a real counterpart genuinely isn't available yet (a component in a later epic, a vendor sandbox that doesn't exist). Then:

1. Label the bead that introduces the placeholder `stub:<what-is-stubbed>`.
2. Create the bead that replaces it with the real thing — even if it lands in a later epic.
3. Link them with a `related` edge, and name the stub explicitly in the replacing bead's description.
4. Record the pair in the epic's Background & Intent comment.

A stub with no replacing bead is a permanent hole. Validation rule 12 catches it.

### Gate beads

Every feature gets exactly one `kind:gate` bead. Every epic gets exactly one.

- Type `task` for feature gates, `milestone` for epic gates.
- Priority equal to the highest priority among its siblings.
- Blocked by **every** other bead in its feature — or, for the epic gate, by every feature gate.
- Acceptance criteria: run the demonstration command, observe the stated output, and **post a comment on the gate bead containing the evidence** (command, output, timestamp).

The epic gate's acceptance criteria are the **runnability checklist**:

1. The build command produces every ART the epic needs
2. There is a documented command that starts the system locally
3. There is a documented command that runs each test level, and it passes
4. Every config value has a default or a documented source
5. Every migration runs from empty on a fresh environment
6. At least one UJ is demonstrable end to end by a human following written steps
7. Every failure mode named in the TDD has an observable signal (log, metric, or error message)

### Spikes, decisions, and deferrals

- **Spike** (`kind:spike`, type `spike`): an unknown that blocks design. Acceptance criteria = the question answered and written down, plus a follow-up bead created. Spikes `blocks` the beads that depend on the answer. Time-box in the estimate.
- **Decision** (`kind:decision`, type `decision`): the plan left a choice open. Acceptance criteria = the choice made, recorded (an ADR or a TDD update), and the affected beads updated. Never let a decision hide inside an impl bead.
- **Deferred** (`kind:deferred`, type `task`, priority 4): a TDD Deferred Improvement. Put the **trigger condition verbatim** in the description ("when p99 exceeds 200 ms", "when tenant count passes 50"). These beads are what keep the plan's future-self knowledge from evaporating.

### Granularity

- **Subtask**: 60–90 minutes of focused work.
- **Task**: one working session, ≤ ~240 minutes. If larger, make it a parent task with subtasks.
- Split on natural boundaries: one seam, one operation, one entity, one migration, one checklist item.
- Do not split so finely that a bead has no independently verifiable outcome.

### Sanity bands

Use these to catch collapsed decomposition *before* writing anything:

| Feature shape | Expected bead count |
|---------------|--------------------|
| 3–6 FRs across 2–4 COMPs, 2–4 seams | **15–40 beads** |
| 1–2 FRs, single COMP, 1 seam | 6–12 beads |
| 8+ FRs across 5+ COMPs | 40–80 beads — consider splitting the feature |

**If a feature of the first shape yields fewer than 10 beads, you collapsed the wiring or the verification.** Go back to ledger B and count the rows again.

A healthy feature's bead mix is roughly 35–45% impl, 15–20% wiring, 20–30% verification, 10% harness/ops/scaffold, plus one gate. **A feature that is 90% impl beads is the exact failure this skill exists to prevent.** Report the count and the mix alongside the band when you present the feature, and explain any number below the band.

### Priority rules

| Priority | Meaning |
|----------|---------|
| 0 | Blocks everything; on the critical path to the walking skeleton |
| 1 | Core FR implementation tied to a CAP; required for the feature gate |
| 2 | Important but not gate-blocking |
| 3 | Secondary functionality; nothing depends on it |
| 4 | Deferred improvements, optional polish |

**Wiring and verification beads inherit the maximum priority of the work they complete — never lower.** A P1 impl bead whose wiring is P3 yields a P1 feature that doesn't run. This is the most common priority error; apply the rule mechanically.

Gate beads take the maximum priority of their siblings. Features and epics take the maximum priority of their children.

---

## Phase 4: Write Bead Content

Beads must be self-documenting. An agent picking one up six weeks from now, with no memory of the plan and no planning documents in its context, must be able to execute it correctly.

### The orphan-context test

Before finalizing each bead, ask: **if this bead were the only thing an agent could read, would it do the right work?**

It must know: what to build, why it matters, where the files go, what interfaces to code against, what "done" means, how to verify it, and what mistakes to avoid.

### The balance rule

> **Inline what changes the implementation. Point to what merely explains it.**

Inline: the exact schema, the operation signature, the error codes, the TR threshold, the file paths, the config key names, the library and version.

Point to: the PRD's market rationale, the floorplan's full diagram, the TDD's extended alternatives discussion.

A bead that says "implement API-9 per contracts" fails the orphan test. A bead that pastes the entire contracts document fails on noise. Inline the operation; cite the document for the surrounding context.

### Field map

`bd` beads have these fields. Use all of them.

#### `title`

Imperative and specific, ≤ 80 characters. "Register session.revoked subscriber in notifier composition root" — not "Notifier work".

#### `description` — what and why

```markdown
## What
<One paragraph: the concrete change, in terms a stranger to the plan understands.>

## Why it matters
<What breaks or is missing without this. For wiring beads, name the code that
stays unreachable. For verify beads, name the failure mode this catches.>

## Scope boundaries
<What is explicitly NOT in this bead, and which bead covers it instead.>

## Traceability
Implements: FR-12.2, TR-7
Contracts: API-9, ENT-3
Floorplan: COMP-3, BD-4
Serves: CAP-2 → UJ-1
```

The `Serves:` line is the through-line to the product goal. It is what lets a future agent judge a trade-off correctly when the bead's instructions don't cover the case it hit.

#### `design` — how

```markdown
## Approach
<The technical approach: algorithm, data structure, control flow. Lift the
relevant TDD content; do not merely cite it.>

## Interfaces
<Inline the exact signatures, schemas, and wire shapes from the contracts.>

## Files & locations
<Concrete paths. Which files are created, which are modified.>

## Technology constraints
<From docs/technology: language, framework, library and version, plus any
idioms the project has settled on.>

## Wiring points
<For anything that needs registration: exactly where it must be registered,
and by which bead if not this one.>
```

#### `acceptance_criteria` — what done means

````markdown
- [ ] <Observable, checkable outcome>
- [ ] <Observable, checkable outcome>

## Verification
```bash
<the exact command(s) that prove this bead is done>
```

## Evidence
Post a comment on this bead with the command output.
````

Every leaf bead needs at least one **runnable** verification command. "Code review passes" is not acceptance criteria. For verify beads, the command is the test invocation, and the criteria state explicitly that no counterpart is mocked at the seam.

#### `notes` — the future-self field

```markdown
## Considerations
<Edge cases, failure modes, performance characteristics, security implications.>

## Alternatives rejected
<What else was considered and why it lost — from the TDD's trade-offs section.
This is what stops a future agent from "fixing" a deliberate choice.>

## Gotchas
<Ordering constraints, surprising behavior, sharp edges in the libraries,
platform differences.>

## Deferred
<Improvements consciously out of scope, with their trigger conditions.>

## Open questions
<What is still unresolved, and which bead resolves it.>
```

#### `metadata` — structured traceability

```json
{
  "frs": ["FR-12.2"],
  "acs": ["AC-4"],
  "trs": ["TR-7"],
  "entities": ["ENT-3"],
  "apis": ["API-9"],
  "events": ["EVT-2"],
  "comps": ["COMP-3"],
  "edges": ["BD-4"],
  "flows": ["DF-2"],
  "sequences": ["SL-1"],
  "journeys": ["UJ-1"],
  "artifacts": ["ART-2"],
  "seam": "API-9 consumer: web-ui → api-svc",
  "kind": "wiring"
}
```

Omit empty keys. The Phase 8 validation rules query against this, so it must be accurate: every ledger A identifier must appear in some bead's metadata.

#### `external_ref`

The primary requirement identifier (e.g. `FR-12.2`), for quick scanning.

#### `estimate`

Minutes. Required on every leaf bead — the granularity bands are checked against it.

#### `labels`

`proj:<slug>`, `ep:<EP-id>`, `ft:<FT-id>`, `kind:<kind>`, one `comp:<COMP-id>` per component touched, plus `stub:<what>` where applicable. Also keep `fr:<FR-id>` labels for the FRs the bead advances — they make `bd query` filtering by requirement possible.

**Never `test:*`.**

### Required comments

Fields describe the work. Comments carry the narrative. Every bead gets at least one comment.

#### Comment 1: Background & Intent (mandatory, every bead)

```markdown
## Background & Intent

**Where this comes from.** <The chain: product goal → capability → requirement →
this bead. Written as prose a human would say out loud.>

**What we're actually trying to achieve.** <The outcome, not the output: what
the user or the system can do afterward that it couldn't before.>

**How it serves the larger goal.** <Why this exists in the architecture at all.
For wiring and verification beads especially: what class of bug it prevents.>

**Thinking that led here.** <The reasoning behind the approach: what we knew,
what we assumed, what we were worried about.>

**What we considered and didn't do.** <Alternatives, and why they lost.>

**What future-you should know.** <The thing that will be non-obvious in six
weeks: sharp edges, coupling that isn't visible in the code, a decision that
looks arbitrary but isn't.>
```

This is where the "future self" requirement is satisfied. Do not write it generically — **a Background comment that would apply equally to any bead in the feature is a wasted comment.** It must name specifics: this seam, this schema, this threshold, this trade-off.

#### Comment 2: Sequencing rationale (where the ordering isn't obvious)

Explain why this bead must follow that one, what breaks if they're reordered, and what could be parallelized but deliberately isn't.

Because `bd` permits only one edge type per pair of beads, this comment is also where **soft** dependencies live: "should follow `abc-123` for consistency, but not blocking."

#### Comment 3: Feature and epic context

On each **feature** bead: the feature's purpose, its FR set, its seam inventory, which wiring-checklist items were determined not to apply and why, and the demonstration command.

On each **epic** bead: the epic's role in the product, the runnability checklist, the document gaps recorded in Phase 0, the stub ledger, and the full seam ledger as a markdown table. **The epic bead is the plan's memory** — someone should be able to read it and understand the whole shape of the work.

---

## Phase 5: Wire the Graph **[THINK DEEPLY]**

### Dependency types

`bd` supports: `blocks`, `tracks`, `related`, `parent-child`, `discovered-from`, `until`, `caused-by`, `validates`, `relates-to`, `supersedes`.

**There is no dependency-strength flag.** Encode strength in the type:

| Intent | Mechanism | Effect |
|--------|-----------|--------|
| Hard — cannot start until done | `--type blocks` | Excluded from `bd ready` |
| Soft — prefer this order | `--type related` | Advisory only; explain in a comment |
| Test proves implementation | `--type validates` | Advisory; documents the pairing |
| Parent / child | the node's `parent` field | Hierarchy, not an edge |

<HARD-GATE>
Only **one** edge may exist between any pair of beads. `bd dep add` fails if an edge already exists with a different type ("dependency already exists with type X"). Choose the strongest applicable type and put the rest in a comment.
</HARD-GATE>

In practice: verify beads take `blocks` edges from their impl, wiring, and harness prerequisites — they genuinely cannot run before those land — and `validates` is reserved for pairs that have no `blocks` relationship.

### Canonical dependency skeleton

Within a feature, the spine runs:

```
scaffold ──▶ contract-types ──▶ skeleton ──▶ impl ──▶ wiring ──▶ verify-seam ──▶ gate
                     │             │          │        │             │
   harness ──────────┴─────────────┴──────────┴────────┴─────────────┘
      ▲
    ops (migrations, config, secrets)
```

- `scaffold` blocks everything in its component.
- `contract-types` blocks any bead that codes against those types.
- `skeleton` blocks the impl beads whose seams it establishes.
- `harness` blocks every `verify-*` bead.
- `ops` (migration, config) blocks the impl and verify beads that need the schema or the config value.
- `wiring` is blocked by the impl beads on both sides of its seam.
- `verify-seam` is blocked by both sides' impl beads and by the wiring bead.
- `verify-journey` is blocked by every `verify-seam` along its path.
- `verify-artifact` is blocked by the build and packaging beads.
- The feature `gate` is blocked by everything else in the feature.
- The epic `gate` is blocked by every feature gate.

### Cross-feature dependencies

Use the feature index's dependency graph. Where feature B depends on feature A:

- B needs A's **contract types** → A's `contract-types` bead blocks B's beads that use them.
- B needs A's **runtime behavior** → A's feature gate blocks B's relevant impl beads.
- B needs A's **decision** only → A's `decision` bead blocks B.

Prefer the narrowest true edge. `A-gate blocks B-scaffold` serializes two features that could have overlapped.

If feature A hasn't been beadified yet, fall back to A's feature-level bead as the dependency target so the edge is tracked, and note in a comment that it should be narrowed once A is decomposed.

### Parallelism analysis

Group beads into waves and present them:

```
Wave 1 (parallel, 4): scaffold-api, scaffold-web, harness-integration, ops-migrate-session
Wave 2 (parallel, 3): contract-types-api, contract-types-web, ops-config-schema
Wave 3 (serial):      skeleton-login-path
Wave 4 (parallel, 5): impl-* beads
Wave 5 (serial, 2):   wiring-* beads that touch the composition root
Wave 6 (parallel, 4): verify-seam-* beads
Wave 7 (serial):      verify-journey-uj1, then feature gate
```

**Check exempt beads for file collisions.** Two `kind:wiring` beads that both edit the composition root cannot run concurrently, even though nothing logically blocks them. Add a `blocks` edge between them and explain it in a sequencing comment. This is the most common source of false parallelism — and it is invisible unless you look for it, because both beads are individually correct.

### Verify the graph before writing

- No cycles (you'll confirm with `bd dep cycles` after the write, but reason about it now).
- Every bead is reachable from its feature gate by walking `blocks` edges backward.
- The wave-1 ready set is non-empty.
- No bead is blocked by something in a later epic — unless that is intentional, in which case say so.

---

## Phase 6: Review & Commit

Present each feature to the user as:

1. **Feature summary** — FR set, COMP set, seam count, bead count and kind mix against the sanity band
2. **The bead table** — key, title, kind, type, priority, comp(s), estimate, blocked-by
3. **Coverage confirmation** — every ledger A row's disposition, every ledger B row's live/proven beads, every ledger C row's demonstration
4. **The waves** — the parallelism plan, with any deliberate serialization explained
5. **What's deliberately missing** — out-of-scope items, stubs, deferrals, open questions

Offer commit checkpoints: after each feature, or after the whole epic. **Default to per-feature** — it limits blast radius if the user wants changes.

<HARD-GATE>
Do NOT write to `bd` until the user confirms.

**Autopilot:** present and proceed.
</HARD-GATE>

---

## Phase 7: Write to `bd`

### Preferred path: atomic graph plan

`bd create --graph` writes the whole graph in one transaction and returns a key→id map. Use it.

Write the plan to the **scratch directory** — never into the repository.

```json
{
  "nodes": [
    {
      "key": "ep-auth",
      "title": "EP-1: Authentication",
      "type": "epic",
      "priority": 1,
      "labels": ["proj:myapp", "ep:EP-1"],
      "description": "## What\n…"
    },
    {
      "key": "ft-session",
      "title": "FT-3: Session issuance",
      "type": "feature",
      "parent": "ep-auth",
      "priority": 1,
      "labels": ["proj:myapp", "ep:EP-1", "ft:FT-3"],
      "description": "## What\n…"
    },
    {
      "key": "impl-api9",
      "title": "Implement POST /sessions handler",
      "type": "task",
      "parent": "ft-session",
      "priority": 1,
      "labels": ["proj:myapp", "ep:EP-1", "ft:FT-3", "kind:impl", "comp:COMP-3", "fr:FR-12.2"],
      "description": "## What\n…\n\n## Why it matters\n…",
      "design": "## Approach\n…",
      "acceptance_criteria": "- [ ] …\n\n## Verification\n…",
      "notes": "## Considerations\n…",
      "external_ref": "FR-12.2",
      "estimate": 120,
      "metadata": { "frs": ["FR-12.2"], "apis": ["API-9"], "kind": "impl" },
      "deps": [{ "target": "contract-types-api", "type": "blocks" }]
    }
  ],
  "edges": [
    { "from_key": "verify-seam-api9", "to_key": "impl-api9", "type": "blocks" }
  ]
}
```

Schema notes — verified against `bd`; do not improvise:

- Node identity is **`key`**, not `id`.
- The graph-plan field is **`acceptance_criteria`** (snake_case); the `bd create` flag is `--acceptance`.
- Dependencies inside a node use `deps: [{ "target": "<key>", "type": "<type>" }]` — the field is **`target`**.
- Top-level edges use **`from_key`/`to_key`** (or `from_id`/`to_id` to reference beads that already exist).
- Hierarchy is the node's **`parent`** field, not an edge.
- Nodes accept: `key`, `title`, `type`, `priority`, `labels`, `description`, `design`, `acceptance_criteria`, `notes`, `external_ref`, `estimate`, `metadata`, `parent`, `deps`.
- Nodes do **not** accept comments — comments are a separate step.
- Graph-plan nodes do **not** inherit labels from their parent. Put every label on every node. (This differs from `bd create`, which inherits by default unless `--no-inherit-labels` is passed.)

Write it:

```bash
bd create --graph plan.json --json > created.json
jq -r '.ids | to_entries[] | "\(.key)\t\(.value)"' created.json > ids.tsv
```

`created.json` has the shape `{"ids":{"ep-auth":"abc-1k2", …},"schema_version":1}`.

### Add the comments

Write each bead's comments to `comments/<key>.md` in the scratch directory, then:

```bash
while IFS=$'\t' read -r key id; do
  [ -f "comments/$key.md" ] && bd comment "$id" --file "comments/$key.md"
done < ids.tsv
```

Use `--file`; do not pass long markdown through shell arguments.

### Fallback path: per-bead creation

If `--graph` is unavailable in the installed `bd`, create beads individually in dependency order:

```bash
bd create "Implement POST /sessions handler" \
  -t task -p 1 \
  --parent "$FT_ID" \
  -l proj:myapp,ep:EP-1,ft:FT-3,kind:impl,comp:COMP-3,fr:FR-12.2 \
  --body-file desc.md \
  --design-file design.md \
  --acceptance "$(cat accept.md)" \
  --notes "$(cat notes.md)" \
  --metadata @metadata.json \
  --external-ref FR-12.2 \
  --estimate 120 \
  --no-inherit-labels \
  --silent
```

`bd create` supports `--design`, `--acceptance`, and `--notes` directly — there is no need for a create-then-update two-step. Prefer the `--body-file` / `--design-file` / `--metadata @file.json` forms for long content.

Then wire dependencies in bulk with newline-delimited JSON:

```bash
# deps.ndjson — one object per line
# {"from":"<blocked-id>","to":"<blocker-id>","type":"blocks"}
bd dep add --file deps.ndjson
```

And add comments:

```bash
bd comment "$ID" --file comments/<key>.md
```

### Idempotency and completion passes

Use **deterministic node keys** derived from identifiers (`impl-api9`, `wiring-web-apiclient`, `verify-seam-evt2`), never random ones. Before writing, query existing beads for the feature and compare titles, so a re-run or a completion pass adds what's missing instead of duplicating.

For a **completion pass** on a partially-beadified feature:

1. `bd query --label proj:<slug> --label ft:<FT-id> --json` to get existing beads and ids.
2. Map existing beads onto ledger rows; mark which ledger rows are already covered.
3. Build a graph plan containing only the new beads, using `from_id`/`to_id` in `edges` to attach them to the existing beads.
4. Backfill what the old beads lack: add the missing `kind:*` label with `bd update`, and post the Background & Intent comment if absent.
5. Re-run the full Phase 8 validation over the whole feature, not just the new beads.

### Re-beadification

When the user asks to re-beadify a feature, do **not** delete existing beads:

1. Create the new beads first, so references exist before old ones are updated.
2. For beads not yet started (`open`): update in place if the change is minor, or close with a reason naming the superseding bead.
3. For beads in progress (`in_progress`): update description, design, and acceptance criteria to match the changed requirements. Do not close active work without user confirmation.
4. Link old → new with `bd dep add <old-id> <new-id> --type supersedes`.

### Environment note

If `bd` reports "legacy Dolt server workspace detected", ambient `BEADS_DOLT_SERVER_MODE` / `BEADS_DOLT_SERVER_PORT` variables are interfering. Run through a wrapper:

```bash
env -u BEADS_DOLT_SERVER_MODE -u BEADS_DOLT_SERVER_PORT bd "$@"
```

### Confirm the write

Report: beads created by kind, dependencies added, comments posted, and anything that failed.

---

## Phase 8: Validate

Run every rule. Report each as ✅ pass or ❌ fail **with the specific offenders**. Do not downgrade a failure to a warning — a failed rule means the Completeness Contract is broken.

```bash
bd lint
bd dep cycles
bd query --label proj:<slug> --label ep:<EP-id> --json > all.json
bv --robot-plan --label proj:<slug>
```

| # | Rule | How to check |
|---|------|--------------|
| 1 | Every ledger A FR sub-item appears in ≥1 bead's `metadata.frs` | jq over `all.json` |
| 2 | Every AC maps to ≥1 `kind:verify-*` bead | jq over metadata |
| 3 | Every TR maps to ≥1 bead; every non-functional TR to a `verify-runtime` bead | jq |
| 4 | Every persisted ENT has an `ops` migration bead and a fixture named in a `harness` bead | jq + read |
| 5 | Every API operation has a provider impl, a consumer wiring, and a `verify-seam` bead | ledger B vs. beads |
| 6 | Every EVT has a publisher bead, a subscriber bead, and a `verify-seam` bead | ledger B vs. beads |
| 7 | Every in-scope BD edge has ≥1 `kind:wiring` bead | ledger B vs. beads |
| 8 | Every DF crossing a COMP boundary appears in a `verify-seam` or `verify-journey` bead | ledger B vs. beads |
| 9 | Every SL has a verify bead covering its ordering constraint | ledger B vs. beads |
| 10 | Every in-scope UJ and UC has a `verify-journey` bead | ledger C vs. beads |
| 11 | Every in-scope ART has a build bead and a `verify-artifact` bead | ledger C vs. beads |
| 12 | Every `stub:*` bead has a linked replacing bead | `bd query --label stub:` |
| 13 | Each feature has exactly one `kind:gate` bead, blocked by all its siblings | jq over deps |
| 14 | Each epic has exactly one `kind:gate` bead, blocked by every feature gate | jq over deps |
| 15 | Every wiring and verify bead's priority ≥ the max priority of what it completes | jq |
| 16 | Every feature's bead mix is within the sanity band, or the deviation is explained | count by kind |
| 17 | `bd dep cycles` returns nothing | command |
| 18 | `bd lint` passes | command |
| 19 | Every bead has ≥1 comment, and the first is a Background & Intent comment | `bd comments` sample + count |
| 20 | No bead carries a `test:*` label | `bd query --label test:` empty for this proj |
| 21 | Every leaf bead has an `estimate` and acceptance criteria containing a runnable command | jq |
| 22 | `bv --robot-plan` shows a non-empty ready set and no unreachable beads | command |

Then re-state the Completeness Contract and answer each invariant with evidence:

1. **Nothing lost** — N identifiers in ledger A; N represented; M out of scope with reasons
2. **Nothing unreachable** — N seams in ledger B, all with a live-via bead
3. **Nothing unverified** — N seams with proving beads, N journeys, N artifacts, N runtime TRs
4. **Nothing unexplained** — N beads, N Background & Intent comments

If any invariant fails, **fix it before declaring the phase done**: add the missing beads. Do not note the gap and move on.

---

## Phase 9: Handoff

Tell the user:

1. **What was created** — counts by kind, the epic and feature ids
2. **Where to start** — `bd ready --label proj:<slug>` and the wave-1 bead list
3. **The demonstration commands** — from the gate beads: what will prove the feature and the epic actually work
4. **What's still open** — spikes, decisions, stubs, deferred beads, document gaps
5. **Next skills** — `test-plan` to add unit-test specs and enrich the verification beads; `tech-plan` if harness beads surfaced new tooling decisions; `artifacts` if ledger C was empty

Suggest the working loop:

```bash
bd ready --label proj:<slug>        # what can be picked up now
bd show <id>                        # full context, self-contained
bd close <id>                       # unblocks the next wave
bv --robot-plan --label proj:<slug> # re-plan after progress
bv --robot-priority                 # recommended ordering
```

---

## Interaction with test-plan

beadify and test-plan both write test-related beads. The division of labor:

| | beadify | test-plan |
|---|---------|-----------|
| **Owns** | The verification **skeleton** — which seams, journeys, artifacts, and runtime requirements must be proven, plus the harness to prove them in | The test **specification** — cases, inputs, expected outputs, unit-level coverage |
| **Labels** | `kind:verify-seam`, `kind:verify-journey`, `kind:verify-artifact`, `kind:verify-runtime`, `kind:harness` | `test:unit`, `test:integration`, `test:e2e`, `test:ux` |
| **Granularity** | One bead per seam / journey / artifact / non-functional TR | Case-level specs attached to beads |

**beadify must never emit a `test:*` label.** test-plan treats any feature that already has a `test:`-labeled bead as planned and skips it — so a stray `test:` label from beadify would silently suppress unit-test planning for the entire feature.

test-plan correspondingly **enriches** beadify's verification beads rather than creating parallel ones: when it finds a `kind:verify-*` bead covering a seam or journey, it appends the case-level specification to that bead and adds the appropriate `test:*` label to it, instead of creating a duplicate.

**Run order: beadify first, then test-plan.** beadify establishes the harness and the integration skeleton; test-plan fills in the cases.

---

## Important Constraints

- **Use maximum reasoning depth.** See the Reasoning Budget section. This is not optional.
- Your ONLY output is `bd` issues (plus the review questions and ledgers you present in-conversation)
- Do NOT write code, create implementation plans, or produce any other artifact
- Do NOT load all starchitect documents upfront — lazy-load per feature to conserve context
- Do NOT create beads without user review and confirmation (except in autopilot)
- Do NOT invent requirements — beads must trace to FRs, TRs, contracts, floorplan elements, or artifacts. If coverage is incomplete, flag the gap rather than filling it with assumptions
- **Never skip the seam ledger.** It is the difference between a plan that produces modules and a plan that produces a working system
- **Integration work is real work.** Wiring, harness, and verification beads are not overhead; they are the beads that make everything else matter
- **Scoped beads stay in one COMP; exempt beads must name every COMP they touch.** Both halves matter
- **No mocks at the seam.** A test that mocks its counterpart verifies nothing about the boundary
- **Never emit `test:*` labels**
- **Never run `bd init`** — tell the user to do it
- **Never write the graph plan into the repository** — it belongs in a scratch directory
- **One edge per bead pair.** Strength lives in the edge type; nuance lives in comments
- **Wiring and verification inherit the priority of what they complete.** Never lower
- **Every bead gets a Background & Intent comment**, specific to that bead
- **Report failures as failures.** A validation rule that fails is a gap in the plan, not a footnote
