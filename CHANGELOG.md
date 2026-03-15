# Changelog

## starchitect 0.6.0

### Fixes

- Improve bv-taskify skill for production readiness — priority assignment (P0-P4) with rules table, intra-feature dependency logic, cross-feature dep fallback to feature-level issues, design field as reference pointers, re-taskification guidance, fixed br create example ([378141b](https://github.com/visigoth/marketplace/commit/378141b))

## starchitect 0.5.0

### Features

- Split contracts output into index + detail files — compact index (~100-120 lines) with summary tables and traceability, detail directory with separate files for entities, API boundaries, and events. Enables on-demand loading by downstream skills. ([32cfe84](https://github.com/visigoth/marketplace/commit/32cfe84))

## starchitect 0.4.0

### Fixes

- Improve contracts skill for production readiness — lazy-load feature PRDs, add missing HARD-GATE to Phase 3, sensible-defaults guidance for interviews, chunked review for large documents, feature-scoped contract consolidation ([cc90a53](https://github.com/visigoth/marketplace/commit/cc90a53))

## starchitect 0.3.0

### Features

- Add bv-taskify skill for feature-to-task decomposition — component-scoped task hierarchies in br (beads) with lazy document loading, parallelism-aware dependencies, and FR coverage tracking ([f9d402e](https://github.com/visigoth/marketplace/commit/f9d402e))

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
