# Starchitect

A workflow for transforming product ideas into implementation-ready task hierarchies. Rather than jumping from PRD to code, Starchitect guides you through architecture, technology choices, feature decomposition, contracts, and test planning — each step building on the last.

## Workflow Progression

```
┌─────────────┐     ┌───────────┐     ┌───────────┐     ┌───────────────────────┐
│  prd-create │────▶│ floorplan │────▶│ tech-plan │────▶│ prd-feature-breakdown │
└─────────────┘     └───────────┘     └───────────┘     └───────────────────────┘
                                                                    │
       ┌────────────────────────────────────────────────────────────┘
       ▼
┌───────────┐     ┌───────────┐     ┌──────────┐     ┌───────────┐
│ contracts │────▶│    tdd    │────▶│ beadify  │────▶│ test-plan │
└───────────┘     └───────────┘     └──────────┘     └───────────┘
                                                           │
                                       ┌───────────────────┤
                                       ▼                   ▼
                                 ┌───────────┐     ┌───────────┐
                                 │ artifacts │     │ tech-plan │  (revisit)
                                 └───────────┘     └───────────┘
```

### 1. prd-create

Start with an idea, end with a structured PRD. The skill interviews you to gather requirements, constraints, and context, then produces an org-mode PRD ready for architectural analysis.

### 2. floorplan

Transform the PRD into an architectural floorplan: block diagrams showing component relationships, data flow diagrams showing how data moves through the system, and swim-lane diagrams showing multi-component protocols. Validates coverage against every PRD item.

### 3. tech-plan

Walk through each component in the floorplan and make technology decisions. The skill scans for existing tech in the repo, researches current options, presents trade-offs, and records your choices with rationale in `docs/technology.md`.

### 4. prd-feature-breakdown

Break the PRD into epics and features. Identifies epic boundaries from capabilities and floorplan clusters, decomposes into features, analyzes dependencies, and generates feature-level PRDs with typed dependency graphs.

### 5. contracts

Define the interfaces between components: entity schemas, API operations, wire protocols, and async events. Uses the floorplan's edges and data flows as the starting point — each edge becomes an API or event contract. The feature index helps prioritize which API boundaries need the cleanest interfaces.

### 6. tdd

Create Technical Design Documents that describe how the system fulfills its functional requirements. TDDs define the internal technical approach — algorithms, data structures, behavioral semantics, storage strategies, concurrency models, and error recovery. Introduces Technical Requirements (TRs) in a global namespace that downstream skills (beadify, test-plan) consume alongside FRs and contract elements. Supports recursive decomposition for complex subsystems.

### 7. beadify

Convert features into a complete implementation task graph in beads (`bd`) — including the wiring, test-harness, and integration-verification work that makes the feature actually run.

Three ledgers drive the decomposition. The **capture ledger** gives every identifier in every upstream document an explicit disposition, so nothing in the plan is silently dropped. The **seam ledger** enumerates every place two things must meet — block-diagram edges, API providers and each consumer, event publishers and each subscriber, data-flow crossings, persisted entities, external dependencies, process boundaries, config and auth boundaries — and answers two questions per row: which bead makes this seam *live*, and which bead *proves* it. The **journey ledger** tracks what a human can actually run at the end.

Beads carry a `kind:*` label — `scaffold`, `contract-types`, `harness`, `skeleton`, `impl`, `wiring`, `verify-seam`, `verify-journey`, `verify-artifact`, `verify-runtime`, `ops`, `gate`, and others. Implementation beads stay scoped to a single component so agents work in parallel without file conflicts; wiring, harness, and verification beads are exempt, because integration work is cross-component by nature and forbidding it is what leaves plans producing unwired code. Gate beads per feature and per epic require running a demonstration command and posting the output before anything is called done.

Every bead is written to pass the orphan-context test: an agent that opens it with no other document available should be able to do the work correctly. A mandatory Background & Intent comment records where the work came from, what it's really for, how it serves the larger goal, what was considered and rejected, and what future-you should know.

### 8. test-plan

Produce test specifications from PRDs, contracts, and task hierarchies. Adds unit test specs to implementation tasks, then **enriches beadify's verification beads** with case-level specs — appending the concrete inputs, expected outputs, and error paths to the bead that already owns each seam — rather than creating a parallel task beside it. New test tasks are created only for coverage no verification bead claims, and that case is reported, since it usually means a seam went unnoticed. TRs from TDDs become additional test targets.

### 9. artifacts

Identify the build artifacts the project must produce to service its runtime environment — application binaries, packages (`.app`, `.exe`, `.ipa`, `.apk`, `.deb`, `.rpm`), container images, service definitions (LaunchAgents, systemd units, Kubernetes manifests), config templates, locale bundles, signing/notarization outputs, and update manifests. Platform-sensitive: validates per-target-platform coverage and cross-platform parity. Records source-of-truth locations in the repo for each artifact's inputs. Can be scoped project-wide, to a single component, or to a single feature.

### 10. tech-plan (revisit)

Test planning and artifact specification often surface new technology decisions — test frameworks, packaging tooling (electron-builder, Tauri, goreleaser), signing infrastructure, CI integration. Run tech-plan again after test-plan/artifacts to capture these choices.

## Usage

Each skill can be invoked by name or trigger phrase:

| Skill | Triggers |
|-------|----------|
| prd-create | "write a PRD", "product requirements", "I have an idea for..." |
| floorplan | "floorplan", "architecture diagram", "how do the components fit together" |
| tech-plan | "tech plan", "technology choices", "what tech should we use" |
| contracts | "contracts", "define the APIs", "entity model", "data schema" |
| tdd | "tdd", "technical design", "how should we build this", "technical requirements" |
| prd-feature-breakdown | "feature breakdown", "break down the PRD", "split into features" |
| beadify | "beadify", "taskify", "create tasks", "break into tasks", "implementation tasks" |
| test-plan | "test plan", "test strategy", "test specs", "add tests" |
| artifacts | "artifacts", "build artifacts", "packaging plan", "what gets shipped", "deployable artifacts" |

## Output Artifacts

| Skill | Output Location |
|-------|-----------------|
| prd-create | `docs/prd.org` or `docs/prd.md` |
| floorplan | `docs/floorplan.org` or `docs/floorplan.md` |
| tech-plan | `docs/technology.org` or `docs/technology.md` |
| contracts | `docs/contracts.org` or `docs/contracts.md` + `docs/contracts/` |
| tdd | `docs/tdd.org` or `docs/tdd.md` + `docs/tdd/` |
| prd-feature-breakdown | `docs/features/index.org` + `docs/features/*.org` |
| beadify | `.beads/` (via `bd` CLI) |
| test-plan | `.beads/` (via `bd` CLI) + `docs/test-plan.org` or `docs/test-plan.md` + `docs/test-plan/` |
| artifacts | `docs/artifacts.org` or `docs/artifacts.md` (+ `docs/artifacts/` when split) |

## Division of labor: beadify and test-plan

These two skills share the verification work and must not duplicate it.

| | beadify | test-plan |
|---|---------|-----------|
| **Owns** | Which seams, journeys, and artifacts must be proven; the harness to prove them in | The cases: inputs, expected outputs, error paths, edge cases |
| **Labels** | `kind:verify-*`, `kind:harness` | `test:unit`, `test:integration`, `test:e2e`, `test:ux` |
| **Granularity** | One bead per seam / journey / artifact / TR | Case-level specs attached to beads |

beadify never emits a `test:*` label, which is what makes test-plan's "skip features that already have one" rule safe. Run beadify first.

## Philosophy

Starchitect is about doing the thinking before the typing. Each skill is a checkpoint that forces you to make decisions explicitly rather than discovering them mid-implementation. The artifacts form a chain of traceability: every task traces back to FRs, every FR traces back to capabilities, every capability traces back to the original product vision.

The graph beadify produces answers a narrower question than "is every requirement assigned to someone": **when every bead is closed, is there something a human can run?** A plan can cover every functional requirement and still ship nothing, if the boundaries between the covered pieces belong to no task. That is what the seam ledger exists to prevent.

## Superpowers

Starchitect dovetails with @obra's [Superpowers](https://github.com/obra/superpowers). For example, **superpowers:brainstorm** can be used to flesh out an idea prior to entering the Starchitect workflow. **starchitect:prd-create** can be used to formalize the output of **brainstorm**. While you could continue to use **superpowers:writing-plans**, Starchitect breaks down the various aspects of writing plans into the various pieces: a floorplan, a technical plan, the features, the contracts within and around those features, and finally, tasks. Every item gets an identifier that can be cross-referenced, and files are kept to reasonable sizes to avoid context overload. Starchitect ensures there is an audit trail from task all the way back to product vision. It's still possible to use **superpowers**' development skills, though they aren't tuned to Starchitect's output.
