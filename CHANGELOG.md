# Changelog

## starchitect 0.16.0

### Features

- Rewrite beadify around a completeness contract — four invariants the skill must demonstrate rather than assert: nothing lost, nothing unreachable, nothing unverified, nothing unexplained. Validated by 22 numbered rules in a dedicated phase. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Add the seam ledger to beadify — enumerates every place two things must meet (block-diagram edges, API providers and each consumer, event publishers and each subscriber, data-flow crossings, persisted entities, external dependencies, process/config/auth boundaries) and requires two answers per row: which bead makes the seam live, and which bead proves it. A seam proven only by a test that mocks the other side is not proven. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Add capture and journey ledgers to beadify — every identifier in every upstream document gets an explicit disposition, and every journey and artifact gets a runnable output. Closes the gap where `docs/artifacts.*` was produced but never consumed. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Split beadify bead scoping into scoped and exempt — implementation beads stay within one component for parallel safety, while wiring, harness, and verification beads are exempt. The previous strict single-component rule left composition roots, route registration, DI, config plumbing, migration ordering, and feature exposure belonging to no task, which is why plans produced implemented-but-unwired features. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Add a 16-kind bead taxonomy to beadify (`kind:scaffold`, `contract-types`, `harness`, `skeleton`, `impl`, `wiring`, `verify-seam`, `verify-journey`, `verify-artifact`, `verify-runtime`, `ops`, `gate`, `spike`, `decision`, `deferred`, `docs`), with a 13-item wiring checklist and an 11-item test-harness checklist. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Add walking-skeleton-first decomposition and gate beads to beadify — an end-to-end path that runs before any component is built out, plus per-feature and per-epic gate beads whose acceptance requires running a demonstration command and posting the output. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Make beadify beads self-documenting — every bead must pass an orphan-context test and carry a Background & Intent comment covering provenance, intent, how it serves the larger goal, alternatives rejected, and what future-you should know. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Instruct beadify to use its highest available reasoning mode (ultrathink on Claude Code), with three marked think-deeply points: the seam ledger, the decomposition, and the dependency graph. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Make test-plan enrich beadify's verification beads instead of duplicating them — case specifications are appended to the bead that already owns each seam, with the `test:*` label added to it. New test tasks are created only where no verification bead claims the coverage, and that case is reported as a likely missed seam. ([#1](https://github.com/visigoth/marketplace/pull/1))

### Fixes

- Correct three `bd` CLI errors in test-plan: `bd create` does accept `--design`, `--acceptance`, and `--notes` (the create-then-update two-step was unnecessary), `bd dep add` has no `--metadata` flag (dependency strength is the edge type; the reason belongs in a comment), and only one dependency edge may exist per issue pair. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Reconcile test-harness ownership — choosing test frameworks belongs to tech-plan, building the harness belongs to beadify's `kind:harness` beads, and specifying cases belongs to test-plan. Previously test-plan declared test infrastructure out of scope and beadify never mentioned it, so integration test specs had nowhere to run. ([#1](https://github.com/visigoth/marketplace/pull/1))
- Fix the documented test-plan output location in the starchitect README (`docs/test-plan.{org,md}` + `docs/test-plan/`, not `docs/testing/`). ([#1](https://github.com/visigoth/marketplace/pull/1))

## starchitect 0.15.0

### Features

- Add artifacts skill — identifies the build artifacts a project must produce (binaries, app packages, container images, service definitions, installers, signing/notarization outputs) per target platform, records source-of-truth locations in the repo, validates cross-platform parity. Scopes project-wide, per-component, or per-feature. Introduces ART identifiers.

## starchitect 0.8.0

### Refactors

- Replace all references to br (beads_rust) with bd (beads) across skills, design docs, and changelog ([e9dbdac](https://github.com/visigoth/marketplace/commit/e9dbdac))

## starchitect 0.7.0

### Features

- Add test-plan skill — final pipeline stage that produces test specifications from PRDs, contracts, and task hierarchies. Adds unit test specs to implementation tasks, creates separate test tasks for integration/e2e/UX tests, outputs split index + per-feature detail files ([c2deb10](https://github.com/visigoth/marketplace/commit/c2deb10))

## starchitect 0.6.0

### Fixes

- Improve beadify skill for production readiness — priority assignment (P0-P4) with rules table, intra-feature dependency logic, cross-feature dep fallback to feature-level issues, design field as reference pointers, re-taskification guidance, fixed bd create example ([378141b](https://github.com/visigoth/marketplace/commit/378141b))

## starchitect 0.5.0

### Features

- Split contracts output into index + detail files — compact index (~100-120 lines) with summary tables and traceability, detail directory with separate files for entities, API boundaries, and events. Enables on-demand loading by downstream skills. ([32cfe84](https://github.com/visigoth/marketplace/commit/32cfe84))

## starchitect 0.4.0

### Fixes

- Improve contracts skill for production readiness — lazy-load feature PRDs, add missing HARD-GATE to Phase 3, sensible-defaults guidance for interviews, chunked review for large documents, feature-scoped contract consolidation ([cc90a53](https://github.com/visigoth/marketplace/commit/cc90a53))

## starchitect 0.3.0

### Features

- Add beadify skill for feature-to-task decomposition — component-scoped task hierarchies in bd (beads) with lazy document loading, parallelism-aware dependencies, and FR coverage tracking ([f9d402e](https://github.com/visigoth/marketplace/commit/f9d402e))

## starchitect 0.2.0

### Features

- Rewrite prd-feature-breakdown with epic hierarchy — 6-phase pipeline with EP/FT identifiers, coalescing, typed dependencies, and over-decomposition guardrails ([3f654c8](https://github.com/visigoth/marketplace/commit/3f654c8))
- Add contracts skill for entity, API, and protocol definitions ([a3c3803](https://github.com/visigoth/marketplace/commit/a3c3803))
- Add floorplan skill for architectural component diagrams ([a17d33e](https://github.com/visigoth/marketplace/commit/a17d33e))

### Fixes

- Restrict floorplan components to runtime concepts ([bfb7e49](https://github.com/visigoth/marketplace/commit/bfb7e49))
- Remove cyclic dependency between features and contracts skills ([eb59f77](https://github.com/visigoth/marketplace/commit/eb59f77))

### Chores

- Remove redundant skills/agents paths from plugin.json ([0a47d5e](https://github.com/visigoth/marketplace/commit/0a47d5e))

## starchitect 0.1.0

- Initial plugin with prd-create, prd-feature-breakdown, and tech-plan skills ([f71c0b6](https://github.com/visigoth/marketplace/commit/f71c0b6))
