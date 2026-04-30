---
name: starchitect:artifacts
description: >
  Identify the build artifacts a project must produce to service its runtime environment —
  binaries, application packages (.app, .exe, .ipa, .apk, .deb, .rpm), container images,
  configuration files, service definitions (LaunchAgent plists, systemd units, Windows services),
  installers, signing/notarization outputs, locale bundles, and similar files. Platform-sensitive:
  evaluates per-target-platform coverage and cross-platform requirements. Records source-of-truth
  locations in the repository for the inputs that produce each artifact. Can be scoped project-wide,
  to a single component, or to a single feature. Triggers: "artifacts", "build artifacts",
  "what do we need to build", "packaging plan", "what artifacts", "installation packages",
  "what gets shipped", "build outputs", "deployable artifacts".
user-invocable: true
---

# Artifacts: Identify Build Artifacts and Their Sources

Identify every file that must be **produced** by the build/release process to service the project's runtime environment, on every target platform the project supports. For each artifact, record what produces it, where its source inputs live in the repository, and how cross-platform requirements are satisfied.

Artifacts are files that exist at deployment time but are *built*, *generated*, or *assembled* — they are not source code. Examples include application binaries, application packages (`.app`, `.exe`, `.apk`, `.ipa`), installer files (`.dmg`, `.pkg`, `.msi`, `.deb`, `.rpm`, AppImage, Flatpak, Snap), container images, service definitions (launchd `.plist`, systemd `.service`, Windows Service registrations, Kubernetes manifests), configuration templates, locale bundles, signing/notarization outputs, code-signing certificates, auto-update manifests (Sparkle, Squirrel), SBOMs, and documentation bundles.

This skill is **platform-sensitive**: it evaluates artifact coverage against each target platform separately, then surfaces cross-platform parity gaps (e.g., "the macOS build produces a notarized `.app`, but the Linux build has no equivalent system-tray integration").

Your sole output is the artifacts document (and minimal repository scaffolding pointers). You do not write build scripts, Dockerfiles, plists, or any artifact content — those are downstream implementation work.

<HARD-GATE>
Do NOT skip to writing output. Target platforms, the artifact inventory, and per-artifact details must be presented to the user for review and confirmation before recording. The complete artifacts document must be reviewed before writing to disk.
</HARD-GATE>

## Autopilot Mode

When invoked with the word **"autopilot"** (e.g., "artifacts on autopilot", "artifacts autopilot"), all confirmation gates below become **soft**: the skill still presents its output at each checkpoint but proceeds immediately without waiting for user confirmation. The user can interrupt at any point to adjust.

Autopilot does NOT skip content — it skips waiting. You still show your work at every gate.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover inputs & determine scope** — find existing docs, scan repo for artifact-producing files, determine output format, confirm scope (project / component / feature)
2. **Identify target platforms & deployment topology** — derive from PRD, tech-plan, floorplan; confirm with user
3. **Identify artifact categories** — walk fixed checklist + extend with project-specific; present for confirmation
4. **Specify each artifact** — per artifact: identifier, type, platform(s), producing component, source locations, build process pointer, distribution target, cross-platform considerations, signing requirements
5. **Validate & assess** — build traceability matrix (PRD/floorplan/tech-plan → artifacts), check cross-platform coverage and component-deployment coverage
6. **Write & commit** — present complete document for review, write to disk, suggest next steps

---

## Phase 0: Discover Inputs & Determine Scope

### Search for existing documents

Search these locations:

| Document | Locations to check |
|----------|--------------------|
| **Existing artifacts doc** | `docs/artifacts.md`, `docs/artifacts.org`, `docs/artifacts/` |
| **PRD** | `docs/prd.md`, `docs/prd.org`, `docs/prd/`, `docs/prds/` |
| **Floorplan** | `docs/floorplan.md`, `docs/floorplan.org` |
| **Technology choices** | `docs/technology.md`, `docs/technology.org`, `docs/technology/` |
| **Contracts** | `docs/contracts.md`, `docs/contracts.org`, `docs/contracts/` |
| **TDDs** | `docs/tdd.md`, `docs/tdd.org`, `docs/tdd/` |
| **Feature index** | `docs/features/index.org`, `docs/features/index.md` |
| **Feature PRDs** | `docs/features/` |

Use Glob to check all locations. Read any documents found (load the feature index, but do NOT load all individual feature PRDs — load on-demand if scope is feature-specific).

### Scan the codebase for existing artifact-producing files

Existing repository state constrains what artifacts already exist or are partially defined. Scan for:

**Container & orchestration:**
- `Dockerfile`, `Dockerfile.*`, `docker-compose.yml`, `docker-compose.*.yml`
- `kubernetes/`, `k8s/`, `helm/`, `*.yaml` with `kind: Deployment|Service|StatefulSet|DaemonSet|Job|CronJob`
- `docker-bake.hcl`, `compose.yaml`

**Linux service definitions:**
- `*.service` (systemd unit files), `*.timer`, `*.socket`, `*.target`
- `init.d/`, `etc/init/` (SysV, Upstart)
- `*.conf` under `etc/`

**macOS service & app artifacts:**
- `*.plist` (LaunchAgent / LaunchDaemon plists, often under `Library/LaunchAgents/`, `Library/LaunchDaemons/`, or app `Contents/`)
- `Info.plist`, `entitlements.plist`, `*.entitlements`
- `*.app/`, `*.xcodeproj/`, `*.xcworkspace/`, `Package.swift`
- `*.pkg`, `*.dmg` build configs (e.g., `create-dmg`, `productbuild` scripts)
- Code-signing config: `ExportOptions.plist`, `notarize.sh`-style scripts
- Auto-update: `Sparkle.framework`, `appcast.xml`

**Windows artifacts:**
- `*.wxs`, `*.wxi` (WiX installer source)
- `*.iss` (Inno Setup), `*.nsi` (NSIS)
- `*.appxmanifest`, `Package.appxmanifest` (MSIX)
- `*.csproj`, `*.sln` deployment profiles, `Properties/PublishProfiles/`
- `app.manifest`, code-signing config

**Linux installers / packaging:**
- `debian/` directory (Debian packaging)
- `*.spec` (RPM spec files)
- `snapcraft.yaml`, `flatpak/`, `*.flatpakrepo`
- `appimage/`, `*.AppImage` configs
- `PKGBUILD` (Arch), `Formula/*.rb` (Homebrew)

**Mobile artifacts:**
- iOS: `*.xcconfig`, `Info.plist`, provisioning profiles, `*.entitlements`, `ExportOptions.plist`
- Android: `AndroidManifest.xml`, `build.gradle` (release configs), `proguard-rules.pro`, `signingConfigs`, `*.keystore` references

**Static / resource artifacts:**
- `assets/`, `resources/`, `static/`, `public/`, `dist/`, `build/`
- Icon source: `*.icns` (macOS), `*.ico` (Windows), `*.png` icon sets
- Locale bundles: `locales/`, `i18n/`, `*.po`, `*.mo`, `*.lproj/`
- Web bundles: webpack/vite/rollup configs that emit `dist/`

**Build manifests / pipelines:**
- `Makefile`, `justfile`, `Taskfile.yml`
- CI configs: `.github/workflows/`, `.gitlab-ci.yml`, `.circleci/config.yml`, `azure-pipelines.yml`, `.buildkite/`
- Goreleaser: `.goreleaser.yml` / `.goreleaser.yaml`
- electron-builder: `electron-builder.yml`, `package.json` `"build"` field
- Cargo bundle: `Cargo.toml` `[package.metadata.bundle]`
- PyInstaller / py2app: `*.spec` (PyInstaller), `setup.py` `py2app` options

**Config templates / generated files:**
- `*.env.example`, `*.env.template`, `config.example.*`
- Generated schemas: `*.gen.go`, `*.pb.go`, `__generated__/`
- Migration files: `migrations/`, `db/migrate/`, `*.sql` under migration dirs

**Distribution / release artifacts:**
- `release/`, `dist/`, `out/`, `target/`, `bin/`
- SBOM/provenance: `*.spdx.json`, `*.cdx.json`, `attestation.json`
- Auto-update manifests: `latest.yml`, `appcast.xml`, `release-manifest.json`

Use Glob to find these files. Read enough to understand what they produce, but do not exhaustively analyze them — note their existence and what artifact they appear to define.

### Determine output format

- Scan `docs/` for `.org` vs `.md` files (Glob for `docs/**/*.org` and `docs/**/*.md`)
- If org-mode files are present, use `docs/artifacts.org`; otherwise use `docs/artifacts.md`
- Tell the user which format you'll use

### If an existing artifacts doc is found

- Load it and summarize current state to the user (which artifacts are catalogued, which platforms, which components)
- Note the highest existing ART number — new artifacts will continue from there
- Identify any gaps the existing doc has flagged (e.g., "ART4: deferred — needs Linux equivalent")
- Ask the user what to do: extend, update specific artifacts, or start fresh

### Determine scope

Ask the user (or infer from invocation context) what scope this run targets:

| Scope | What it covers | When to use |
|-------|----------------|-------------|
| **Project-wide** | Every artifact every component must produce | First run, or comprehensive update |
| **Component** | Artifacts owned by one COMP from the floorplan | Adding/changing a single component's deployment story |
| **Feature** | Artifacts touched by one feature (across components) | A feature introduces new artifacts (e.g., a new mobile companion app) |

If the invocation doesn't specify, recommend project-wide for the first run, otherwise component or feature based on what's changed.

### Present discovered context

Summarize what you found to the user:
- "From the PRD, target platforms appear to be: [list]"
- "From the tech-plan, production container requirements are: [summary]; devcontainer config covers: [summary]"
- "From the floorplan, components that ship to runtime: [list of COMPs by deployment type]"
- "From the codebase, I see these existing artifact-producing files: [list]"
- "These existing artifacts will be carried forward — I'll catalogue them rather than reinvent them."

<HARD-GATE>
Do NOT proceed to Phase 1 until the user has confirmed the scope and the discovered context.

**Autopilot:** Present the scope and discovered context, then proceed immediately.
</HARD-GATE>

---

## Phase 1: Identify Target Platforms & Deployment Topology

Artifacts only make sense per-platform. Before enumerating artifacts, fix the platform matrix.

### Derive target platforms

Inspect inputs in this order:

1. **PRD** — look for explicit platform AC items (e.g., "AC2: must run on macOS 13+, Windows 11, Ubuntu 22.04 LTS"). Look in capabilities, non-functional requirements, and architectural constraints.
2. **Tech-plan** — production container image requirements specify Linux container targets and CPU architectures (`amd64`, `arm64`). Mobile framework choices imply iOS/Android targets. Desktop framework choices (Electron, Tauri, native) imply OS targets.
3. **Floorplan** — `application` components are usually user-facing and need platform decisions. `service`/`daemon` components usually deploy to Linux (containers or VMs).
4. **Codebase** — existing CI matrices, `Cargo.toml` `[target.*]` sections, Xcode build settings, `electron-builder` `build.<platform>` blocks, etc.

Build a platform matrix:

| Platform | Architectures | Notes |
|----------|---------------|-------|
| macOS | arm64, x86_64 | Universal binary expected? Min OS version? |
| Windows | x86_64, arm64 | Min Windows version? Code-signing required? |
| Linux | amd64, arm64 | Distro families? (deb, rpm, AppImage, Snap, Flatpak) |
| iOS | arm64 | Min iOS version? Distribution: App Store, TestFlight, Enterprise? |
| Android | arm64-v8a, armeabi-v7a, x86_64 | Min API level? Distribution: Play Store, side-load, MDM? |
| Web | n/a | Browser support matrix? Static-host or server-rendered? |
| Container (Linux) | amd64, arm64 | Production runtime — what registry? |
| Embedded / other | (specify) | (specify) |

Drop platforms that the PRD/tech-plan exclude. Add platforms not yet covered if the PRD demands them.

### Derive deployment topology

For each component (COMP) from the floorplan, identify where it runs and how it gets there. Walk the floorplan's component inventory:

| COMP | Type | Deployment target | Distribution channel |
|------|------|-------------------|----------------------|
| COMP1: Web App | application | Static hosting / CDN | Build pipeline → object storage |
| COMP2: API Gateway | gateway | Container in K8s | Registry → cluster |
| COMP3: Background Worker | service | Container in K8s | Registry → cluster |
| COMP4: Desktop App | application | User's machine | Auto-updater + initial download |
| COMP5: iOS App | application | User's device | App Store |
| COMP6: User DB | storage | Managed cloud DB | Provisioned, not built |

Components with code status `external` typically produce no artifacts (they are consumed, not built) — note them but skip during artifact specification.

`storage` components with code status `existing` or `external` (managed databases, cloud object stores) produce no build artifacts but may need **schema artifacts** (migration bundles, seed data). Capture those.

### Present platform matrix and deployment topology

Show the user:
- The platform matrix with architectures
- The deployment topology table (COMP → target → channel)
- Any platforms that came from the PRD but lack a clear owning component
- Any components that lack a clear deployment target

Ask the user to confirm, add, remove, or adjust. Resolve any gaps.

<HARD-GATE>
Do NOT proceed to Phase 2 until the user has confirmed the platform matrix and deployment topology.

**Autopilot:** Present the matrix, then proceed immediately.
</HARD-GATE>

---

## Phase 2: Identify Artifact Categories

Walk a fixed checklist of artifact categories, marking which are relevant for each (component × platform) pair from Phase 1. Then extend with project-specific categories.

### Fixed checklist of artifact categories

For each (component × platform) pair, evaluate whether each category applies. Mark `applies`, `n/a`, or `deferred`.

**Executable / runnable code:**
1. Native executable / binary (per OS, per arch) — e.g., Mach-O, ELF, PE
2. Bytecode / interpreted bundle — e.g., Java JAR, .NET DLL, Python wheel, Node bundle
3. Static / SPA web bundle — HTML/JS/CSS distributable
4. Mobile compiled artifact — `.ipa` (iOS), `.apk`/`.aab` (Android)
5. WebAssembly module — `.wasm`

**Application packaging (user-installable):**
6. macOS application bundle — `.app`
7. macOS installer — `.pkg`, `.dmg`
8. Windows installer — `.msi`, `.exe` (Inno/NSIS), `.appx`/`.msix`
9. Linux package — `.deb`, `.rpm`, AppImage, Snap, Flatpak, Pacman
10. Mobile distribution package — App Store/TestFlight `.ipa`, Play Store `.aab`
11. Browser extension package — `.crx`, `.xpi`, `.zip`

**Container images:**
12. Production container image (per arch) — pushed to registry
13. Sidecar / init container images — supporting containers
14. Devcontainer / development image — for local dev (links to tech-plan devcontainer config)

**Service definitions (runtime registration):**
15. macOS LaunchAgent / LaunchDaemon `.plist`
16. systemd unit files — `.service`, `.timer`, `.socket`, `.target`
17. Windows Service registration / `sc.exe` install scripts
18. Kubernetes manifests — Deployment, Service, ConfigMap, Secret, Ingress, HPA, etc.
19. Helm chart / Kustomize overlay
20. Cloud-platform config — ECS task def, Cloud Run config, Lambda zip + config, App Runner

**Configuration & resource files:**
21. Config templates — `*.env.example`, `config.template.{yaml,toml,json}`
22. Locale / translation bundles — `.lproj/`, `*.po`/`.mo`, JSON locale files
23. Static assets — icons, images, fonts (when bundled separately from the binary)
24. Database migration bundle — versioned migration files shipped with the app
25. Default seed data — initial fixtures shipped with the app
26. Documentation bundle — manpages, HTML docs, PDFs shipped to users

**Signing, notarization, and provenance:**
27. Code-signing certificates / signed binary outputs (per platform that requires signing)
28. macOS notarization ticket (stapled to `.app`/`.pkg`/`.dmg`)
29. Windows Authenticode signature
30. Android signing config / signed APK/AAB
31. iOS provisioning profile, signed `.ipa`
32. SBOM — `*.spdx.json`, `*.cdx.json`
33. SLSA/in-toto attestation, build provenance

**Update & distribution metadata:**
34. Auto-update manifest — Sparkle `appcast.xml` (macOS), Squirrel `RELEASES` (Windows), electron-updater `latest.yml`, custom JSON
35. App Store / Play Store metadata bundle — screenshots, descriptions, what's-new strings
36. Release notes / CHANGELOG snapshot shipped with build

**Build/CI inputs that materialize on disk:**
37. Generated code artifacts — protobuf stubs, OpenAPI clients, GraphQL types — *only* if shipped (not if regenerated by consumers)
38. License files / NOTICE bundle — required by dependencies' licenses
39. Reproducibility manifest — lockfiles snapshot, build environment description

### Extend with project-specific categories

Read PRD, tech-plan, and floorplan for any category outside the fixed checklist. Common extensions:

- **ML model weights** — `.onnx`, `.pt`, `.gguf` shipped with the app or hosted separately
- **Firmware images** — for hardware projects
- **Browser bookmarklet / userscript** — `.user.js`
- **Editor extensions** — VS Code `.vsix`, JetBrains plugin `.zip`
- **CLI completion scripts** — `bash`, `zsh`, `fish` completions installed alongside the binary
- **Manpages and shell integration** — `.1` files, shell hook scripts
- **MCP server registration manifests** — when shipping an MCP server
- **OS extensions** — Spotlight importers, QuickLook generators, Finder Sync Extensions, Windows Shell Extensions
- **Kernel/system extensions** — DriverKit, eBPF programs (these have separate signing requirements)

If a category falls outside this list, add it with a name and a reason.

### Prioritize and present

Order categories so blockers come first (e.g., a `.app` package depends on the macOS executable; the `.dmg` depends on the `.app`; the notarization ticket depends on the signed `.app`). The artifact specification phase will follow this order.

Present the full applicable artifact list to the user. The presentation should be a matrix:

| Artifact category | Component(s) | Platform(s) | Status |
|-------------------|--------------|-------------|--------|
| Native binary | COMP4 | macOS arm64, macOS x86_64, Windows x86_64, Linux amd64 | applies |
| `.app` bundle | COMP4 | macOS (universal) | applies |
| `.dmg` installer | COMP4 | macOS | applies |
| Notarization ticket | COMP4 | macOS | applies |
| Container image | COMP2, COMP3 | linux/amd64, linux/arm64 | applies |
| Kubernetes manifests | COMP2, COMP3 | n/a | applies |
| `.ipa` | COMP5 | iOS arm64 | applies |
| App Store metadata | COMP5 | iOS | applies |
| LaunchAgent plist | COMP4 | macOS | deferred (not needed for v1) |

Ask the user to confirm, remove, add, or change status before proceeding.

<HARD-GATE>
Do NOT proceed to Phase 3 until the user has confirmed the artifact category matrix.

**Autopilot:** Present the matrix, then proceed immediately.
</HARD-GATE>

---

## Phase 3: Specify Each Artifact

For each in-scope artifact (status = `applies` or `deferred-with-spec`), produce a specification. Walk the matrix from Phase 2 in priority order. Group by component to keep related artifacts together.

### Per-artifact fields

For each artifact, capture:

- **Identifier** — `ART1`, `ART2`, etc. (flat global namespace; no dot notation)
- **Name** — descriptive (e.g., "macOS Notarized App Bundle for Editor", "Production API Container Image")
- **Category** — from the fixed checklist or project-specific extension
- **Producing component(s)** — COMP identifier(s); the component(s) whose code goes into this artifact
- **Target platform(s)** — explicit platform + architecture combinations
- **Source inputs** — repository paths whose contents directly feed into building this artifact:
  - Code source roots (e.g., `src/editor/`)
  - Resource directories (e.g., `assets/icons/macos/`)
  - Configuration source (e.g., `packaging/macos/Info.plist.in`, `packaging/macos/entitlements.plist`)
  - Build manifest (e.g., `Cargo.toml` `[package.metadata.bundle]`, `electron-builder.yml`, `Dockerfile`)
  - Locale source (e.g., `i18n/`, `*.lproj/`)
  - Schema / generated source (e.g., `proto/`, `migrations/`)
- **Build process pointer** — the file(s) that orchestrate the build (e.g., `Makefile:dmg`, `.github/workflows/release.yml:build-macos`, `electron-builder.yml`). Do NOT define the build process here — point to where it lives or where it should live.
- **Distribution / installation target** — where this artifact lives at runtime:
  - For a `.app`: `/Applications/Foo.app` or `~/Applications/Foo.app`
  - For a LaunchAgent: `~/Library/LaunchAgents/com.example.foo.plist`
  - For a systemd unit: `/etc/systemd/system/foo.service` or `/usr/lib/systemd/system/foo.service`
  - For a container image: `<registry>/<repo>:<tag>`
  - For a config file: `~/.config/foo/config.toml` or `/etc/foo/config.toml`
  - For an iOS app: App Store distribution + on-device install path
- **Cross-platform parity notes** — when this artifact is part of a cross-platform set, what is the equivalent on each other platform? Cite the ART identifier of the equivalents. If there is no equivalent, justify why.
- **Signing / notarization / attestation** — what cryptographic operations are required to make this artifact distributable:
  - macOS: Developer ID signature + notarization + stapling
  - Windows: Authenticode signature (EV or standard) + optional SmartScreen reputation
  - iOS: Provisioning profile + Apple Distribution signature
  - Android: app signing key + (optional) Play App Signing
  - Linux: detached signature (`.asc`), repo signing keys
  - Containers: cosign / notation signature, SBOM attestation
- **PRD / tech-plan / floorplan provenance** — the FR, AC, COMP, or tech-plan section that justifies this artifact's existence
- **Status** — `applies` (must produce), `deferred` (acknowledged but not yet required; specify when), `existing` (already produced today), `external` (a third party produces it)

### Source-of-truth principle

For every input listed under "Source inputs," the path must be **a path in this repository** (or, for cross-repo monorepo cases, an explicit cross-repo reference). The artifacts document is the index that says: *"to change ART4, edit these files."*

If an input does not yet exist in the repository, list it as a **planned** path (e.g., `packaging/macos/Info.plist.in (planned)`) and flag it in Phase 4 quality assessment as a gap.

If an input lives outside the repository (a Slack file, an email attachment, a designer's local machine), this is a **source-of-truth violation** — flag it. The remediation is to bring the input into the repository (or document why it cannot be).

### Cross-platform considerations

When an artifact is part of a cross-platform group (the same conceptual deliverable across multiple platforms), explicitly walk the parity:

- **Functional parity** — does each platform's artifact deliver the same functionality? If not, document the divergence and the reason.
- **Resource parity** — icons, locales, fonts, graphics: do all platforms ship the same set? Different formats are expected (`.icns` vs `.ico` vs `.png`), but each platform's set must be complete.
- **Configuration parity** — config keys and defaults should match across platforms unless platform-specific behavior justifies divergence.
- **Update mechanism parity** — does each platform have an auto-update story? Sparkle/Squirrel/electron-updater/Play Store/App Store/distro repo are all valid; "no update mechanism" is rarely valid.
- **Naming conventions** — bundle identifiers (`com.example.app`), executable names, and config paths follow each platform's conventions.

### Per-artifact presentation

Present each artifact specification to the user in a structured form (table or list). Group by component. Within each component, order by dependency (binary → bundle → installer → notarization).

After each component group, the user can:
- **Confirm** — accept the specifications as-is
- **Adjust** — modify fields, add/remove artifacts, change source paths
- **Defer** — mark artifacts as deferred with notes on when to revisit

<HARD-GATE>
Do NOT proceed to Phase 4 until the user has confirmed the artifact specifications for every in-scope component.

**Autopilot:** Present the specifications per component, then proceed immediately.
</HARD-GATE>

---

## Phase 4: Validate & Assess

### Build the traceability matrix

For every PRD platform AC, every COMP from the floorplan with a deployment target, and every tech-plan production-container or devcontainer requirement, identify which ART(s) realize it.

| Source item | Type | Realized by |
|-------------|------|-------------|
| AC2 (must run on macOS 13+) | PRD AC | ART1 (macOS binary), ART2 (.app), ART3 (.dmg), ART4 (notarization) |
| AC3 (must run on Windows 11) | PRD AC | ART5 (Windows binary), ART6 (.msi) |
| AC4 (must run on Linux) | PRD AC | ART7 (Linux binary), ART8 (.deb), ART9 (.rpm), ART10 (AppImage) |
| COMP4 deployment | Floorplan | ART1–ART10 |
| Production container reqs (tech-plan §X) | Tech-plan | ART11 (API container), ART12 (worker container) |
| Devcontainer config (tech-plan §Y) | Tech-plan | ART13 (devcontainer image) |

Every platform AC, deployable COMP, and tech-plan deployment requirement must have at least one ART entry. Empty rows are validation failures.

### Run validation rules

Evaluate each rule. State pass or fail with a brief note.

#### Coverage rules (nothing missing)

1. **Platform AC coverage** — every PRD platform AC has at least one ART that targets that platform.
2. **Component deployment coverage** — every floorplan COMP with a runtime deployment target has at least one ART that produces it.
3. **Tech-plan container coverage** — production container image requirements from the tech-plan map to at least one container image ART.
4. **Devcontainer coverage** — devcontainer configuration from the tech-plan maps to a devcontainer image ART (or is explicitly out of scope).
5. **Update mechanism coverage** — every user-installable application artifact has an associated update mechanism ART (auto-update manifest, store distribution, distro repo) or an explicit "no-update" justification.
6. **Signing coverage** — every artifact distributed outside the team has a signing/notarization/attestation ART covering it (where the platform requires it).
7. **License coverage** — if the project depends on libraries with attribution requirements, a NOTICE/license bundle ART exists.

#### Cross-platform parity rules

8. **Functional parity** — for each cross-platform artifact group, every platform delivers the same functional surface, OR divergences are documented and justified.
9. **Resource parity** — every platform's artifact group includes complete icons, locales, and resources, in that platform's expected format.
10. **Naming convention compliance** — bundle identifiers, executable names, and install paths follow each platform's conventions.

#### Source-of-truth rules

11. **Inputs in repo** — every "Source input" path either exists in the repo today or is flagged as planned. No artifact has source inputs outside the repository (without explicit cross-repo reference).
12. **Build process pointer present** — every ART points to a build orchestrator file (existing or planned). No artifact relies on an undocumented manual build process.
13. **No duplicate sources** — when two artifacts share source inputs (e.g., the macOS binary feeds both the `.app` and a CLI install), the dependency is explicit (one ART consumes another's output).

#### Identifier and namespace rules

14. **Identifier integrity** — all ART identifiers are unique, sequential, flat (no dot notation), and do not collide with PRD/floorplan/contracts/TR identifiers (`CAP`, `FR`, `UC`, `UJ`, `AC`, `G`, `NG`, `P`, `COMP`, `BD`, `DF`, `SL`, `ENT`, `API`, `EVT`, `TR`).

### Produce the quality assessment

Write the assessment as the final section of the artifacts document. List each rule with pass/fail and a brief note. For any failure, describe the gap and what would close it.

### Present the validation results

Show the user the traceability matrix and the quality assessment table. Highlight any failures.

The user can:
- **Accept** — keep failures as known gaps in the document
- **Resolve** — go back to Phase 3 and add/adjust artifacts to close gaps
- **Defer** — document gaps with explicit owners and revisit conditions

<HARD-GATE>
Do NOT proceed to Phase 5 until the user has reviewed the validation results.

**Autopilot:** Present the validation results, then proceed immediately.
</HARD-GATE>

---

## Phase 5: Write & Commit

### Decide single-file vs split structure

Default to a **single file** (`docs/artifacts.{md,org}`) when there are fewer than ~12 artifacts.

Use a **split structure** when there are more artifacts or when grouping by component improves navigation:

```
docs/artifacts.{md,org}              ← compact index (overview + summary table)
docs/artifacts/
  <component-or-group>.{md,org}      ← one file per component or platform group
```

Apply the same scoping rules as test-plan: the index is created on first run; subsequent runs update the relevant detail files and refresh the index summary table.

For component-scoped or feature-scoped runs, write only the relevant detail file(s) and update the index.

### Present the complete document

Show the full content. Prompt: "Would you like to review the artifacts document before I write it, or should I go ahead and write to disk?"

If the user wants to review, present and wait for approval. If they choose to skip, write directly. **Autopilot:** skip review and write directly.

### Write to disk

Write to `docs/artifacts.{md,org}` (and `docs/artifacts/<name>.{md,org}` files if split).

### Commit the document

- Stage the new/updated files
- Commit with message: `docs: add artifacts catalogue` (first run) or `docs: update artifacts catalogue (<scope>)` (subsequent runs)

### Suggest next steps

- "Your artifacts catalogue is documented. Next steps you might consider:"
  - For each ART with planned source paths, create implementation tasks (run beadify with the artifacts context, or add tasks manually)
  - For artifacts requiring signing/notarization, ensure the relevant credentials and CI secrets are provisioned
  - Run the tech-plan skill again if new technology choices surface during artifact specification (e.g., picking electron-builder vs Tauri for desktop packaging)
  - Consider running the test-plan skill — environment tests should validate that artifacts build cleanly, install cleanly, and pass smoke tests

---

## Output Document Structure

Use the format determined in Phase 0 (org-mode if `.org` files exist in `docs/`, else markdown). When using the split structure, the index is compact and the detail files contain the per-artifact specifications.

### Single-file template — markdown (`docs/artifacts.md`):

```markdown
# Artifacts

**PRD:** [PRD](prd.md)
**Floorplan:** [Floorplan](floorplan.md)
**Technology:** [Technology Choices](technology.md)
**Date:** YYYY-MM-DD
**Scope:** Project-wide | Component: COMPN — <name> | Feature: FTN — <name>

## Overview

Brief summary of what this catalogue covers. Note the target-platform matrix and the components in scope.

## Target Platforms

| Platform | Architectures | Min version / notes |
|----------|---------------|---------------------|
| macOS | arm64, x86_64 (universal) | macOS 13+ |
| Windows | x86_64 | Windows 11 |
| Linux | amd64, arm64 | Ubuntu 22.04 LTS, RHEL 9 |
| Container (Linux) | amd64, arm64 | Distroless, non-root |

## Deployment Topology

| COMP | Type | Deployment target | Distribution channel |
|------|------|-------------------|----------------------|
| COMP1 | application | Static hosting | CI → object storage → CDN |
| COMP2 | service | K8s cluster | Registry → cluster |
| COMP4 | application | User machines | Auto-updater + initial download |

## Artifacts

### COMP4: Desktop Editor

#### ART1: macOS Universal Binary

- **Category:** Native executable
- **Platforms:** macOS arm64, macOS x86_64
- **Producing component:** COMP4
- **Source inputs:**
  - `src/editor/` — editor source
  - `assets/icons/macos/AppIcon.icns` — app icon
- **Build process pointer:** `Makefile:macos-binary` → `cargo build --release --target=universal-apple-darwin`
- **Distribution target:** Embedded in ART2
- **Cross-platform parity:** ART5 (Windows), ART7 (Linux)
- **Signing:** Developer ID Application signature applied here; consumed by ART2
- **Provenance:** AC2 (macOS support), COMP4
- **Status:** applies

#### ART2: macOS App Bundle

- **Category:** Application package (`.app`)
- **Platforms:** macOS (universal)
- **Producing component:** COMP4
- **Source inputs:**
  - ART1 (binary) — embedded in `Foo.app/Contents/MacOS/Foo`
  - `packaging/macos/Info.plist.in` — bundle metadata template
  - `packaging/macos/entitlements.plist` — entitlements
  - `assets/icons/macos/AppIcon.icns` — bundle icon
  - `i18n/*.lproj/` — localized strings (parity with ART5/ART7 locales)
- **Build process pointer:** `packaging/macos/build-app.sh` (planned)
- **Distribution target:** `/Applications/Foo.app`
- **Cross-platform parity:** ART6 (Windows .msi), ART8 (Linux AppImage)
- **Signing:** Developer ID Application; embedded by `codesign --deep`
- **Provenance:** AC2, COMP4
- **Status:** applies

#### ART3: macOS DMG Installer

- **Category:** Installer (`.dmg`)
- **Platforms:** macOS
- **Producing component:** COMP4
- **Source inputs:**
  - ART2 (`.app`)
  - `packaging/macos/dmg-background.png`
  - `packaging/macos/dmg-config.json` — `create-dmg` config
- **Build process pointer:** `Makefile:dmg`
- **Distribution target:** Downloaded from `https://example.com/downloads/Foo-{version}.dmg`
- **Cross-platform parity:** ART6 (Windows .msi), ART8 (Linux AppImage)
- **Signing:** Inherits ART2 signature; the DMG itself is also signed and notarized
- **Provenance:** AC2, COMP4
- **Status:** applies

#### ART4: macOS Notarization Ticket

- **Category:** Notarization
- **Platforms:** macOS
- **Producing component:** COMP4 (CI)
- **Source inputs:** ART2, ART3, Apple ID + app-specific password (CI secrets)
- **Build process pointer:** `.github/workflows/release.yml:notarize-macos`
- **Distribution target:** Stapled to ART2 and ART3
- **Cross-platform parity:** ART9 (Windows Authenticode), n/a for Linux
- **Signing:** Apple notarization service
- **Provenance:** AC2 (macOS Gatekeeper compliance), COMP4
- **Status:** applies

### COMP2: API Service

#### ART11: API Production Container Image

- **Category:** Container image
- **Platforms:** linux/amd64, linux/arm64
- **Producing component:** COMP2
- **Source inputs:**
  - `services/api/` — service source
  - `services/api/Dockerfile` — image definition
  - `services/api/migrations/` — DB migrations bundled into image
- **Build process pointer:** `.github/workflows/release.yml:build-api-image`
- **Distribution target:** `ghcr.io/example/api:{version}` and `:latest` on main
- **Cross-platform parity:** n/a (server-only)
- **Signing:** cosign signature, SBOM attestation
- **Provenance:** Tech-plan §Production Container Image Requirements, COMP2
- **Status:** applies

## PRD / Floorplan / Tech-Plan Traceability

| Source item | Type | Realized by |
|-------------|------|-------------|
| AC2 (macOS support) | PRD AC | ART1, ART2, ART3, ART4 |
| AC3 (Windows support) | PRD AC | ART5, ART6, ART9 |
| AC4 (Linux support) | PRD AC | ART7, ART8, ART10 |
| COMP4 deployment | Floorplan | ART1–ART10 |
| Production container reqs | Tech-plan | ART11, ART12 |

## Quality Assessment

| # | Rule | Status | Notes |
|---|------|--------|-------|
| 1 | Platform AC coverage | Pass | All platform ACs map to ARTs |
| 2 | Component deployment coverage | Pass | All deployable COMPs covered |
| 3 | Tech-plan container coverage | Pass | API + worker images defined |
| 4 | Devcontainer coverage | Deferred | Out of scope this run |
| 5 | Update mechanism coverage | Pass | Sparkle (macOS), Squirrel (Win), AppImageUpdate (Linux) |
| 6 | Signing coverage | Pass | All distributed artifacts signed |
| 7 | License coverage | Fail | NOTICE bundle ART not yet defined |
| 8 | Functional parity | Pass | Same feature set on all desktop platforms |
| 9 | Resource parity | Pass | Icons + locales complete on all platforms |
| 10 | Naming convention compliance | Pass | Bundle IDs follow conventions |
| 11 | Inputs in repo | Pass | All source inputs in repo or flagged planned |
| 12 | Build process pointer present | Pass | Every ART points to a build file |
| 13 | No duplicate sources | Pass | Inter-artifact deps explicit |
| 14 | Identifier integrity | Pass | ART1–ART13 unique, no collisions |
```

### Single-file template — org-mode (`docs/artifacts.org`):

```org
#+TITLE: Artifacts
#+DATE: YYYY-MM-DD

* Overview

Brief summary of what this catalogue covers.

PRD: [[file:prd.org][PRD]]
Floorplan: [[file:floorplan.org][Floorplan]]
Technology: [[file:technology.org][Technology Choices]]
Scope: Project-wide | Component: COMPN -- <name> | Feature: FTN -- <name>

* Target Platforms

| Platform           | Architectures            | Min version / notes        |
|--------------------+--------------------------+----------------------------|
| macOS              | arm64, x86_64 (universal) | macOS 13+                  |
| Windows            | x86_64                   | Windows 11                 |
| Linux              | amd64, arm64             | Ubuntu 22.04 LTS, RHEL 9   |
| Container (Linux)  | amd64, arm64             | Distroless, non-root       |

* Deployment Topology

| COMP  | Type        | Deployment target  | Distribution channel              |
|-------+-------------+--------------------+-----------------------------------|
| COMP1 | application | Static hosting     | CI -> object storage -> CDN       |
| COMP2 | service     | K8s cluster        | Registry -> cluster               |
| COMP4 | application | User machines      | Auto-updater + initial download   |

* Artifacts

** COMP4: Desktop Editor

*** ART1: macOS Universal Binary

- Category :: Native executable
- Platforms :: macOS arm64, macOS x86_64
- Producing component :: COMP4
- Source inputs ::
  - =src/editor/= -- editor source
  - =assets/icons/macos/AppIcon.icns= -- app icon
- Build process pointer :: =Makefile:macos-binary=
- Distribution target :: Embedded in ART2
- Cross-platform parity :: ART5 (Windows), ART7 (Linux)
- Signing :: Developer ID Application signature; consumed by ART2
- Provenance :: AC2, COMP4
- Status :: applies

*** ART2: macOS App Bundle

- Category :: Application package (=.app=)
- Platforms :: macOS (universal)
- Producing component :: COMP4
- Source inputs ::
  - ART1 (binary)
  - =packaging/macos/Info.plist.in=
  - =packaging/macos/entitlements.plist=
  - =assets/icons/macos/AppIcon.icns=
  - =i18n/*.lproj/=
- Build process pointer :: =packaging/macos/build-app.sh= (planned)
- Distribution target :: =/Applications/Foo.app=
- Cross-platform parity :: ART6 (Windows .msi), ART8 (Linux AppImage)
- Signing :: Developer ID Application
- Provenance :: AC2, COMP4
- Status :: applies

* PRD / Floorplan / Tech-Plan Traceability

| Source item               | Type        | Realized by              |
|---------------------------+-------------+--------------------------|
| AC2 (macOS support)       | PRD AC      | ART1, ART2, ART3, ART4   |
| AC3 (Windows support)     | PRD AC      | ART5, ART6, ART9         |
| AC4 (Linux support)       | PRD AC      | ART7, ART8, ART10        |
| COMP4 deployment          | Floorplan   | ART1--ART10              |
| Production container reqs | Tech-plan   | ART11, ART12             |

* Quality Assessment

| # | Rule                              | Status   | Notes                                                |
|---+-----------------------------------+----------+------------------------------------------------------|
| 1 | Platform AC coverage              | Pass     | All platform ACs map to ARTs                         |
| 2 | Component deployment coverage     | Pass     | All deployable COMPs covered                         |
| 3 | Tech-plan container coverage      | Pass     | API + worker images defined                          |
| 4 | Devcontainer coverage             | Deferred | Out of scope this run                                |
| 5 | Update mechanism coverage         | Pass     | Sparkle, Squirrel, AppImageUpdate                    |
| 6 | Signing coverage                  | Pass     | All distributed artifacts signed                     |
| 7 | License coverage                  | Fail     | NOTICE bundle ART not yet defined                    |
| 8 | Functional parity                 | Pass     | Same feature set on all desktop platforms            |
| 9 | Resource parity                   | Pass     | Icons + locales complete                             |
| 10 | Naming convention compliance     | Pass     | Bundle IDs follow conventions                        |
| 11 | Inputs in repo                   | Pass     | All inputs in repo or flagged planned                |
| 12 | Build process pointer present    | Pass     | Every ART points to a build file                     |
| 13 | No duplicate sources             | Pass     | Inter-artifact deps explicit                         |
| 14 | Identifier integrity             | Pass     | ART1--ART13 unique, no collisions                    |
```

### Split structure — index file (`docs/artifacts.{md,org}`):

When using the split structure, the index contains:

- Overview, target platforms, deployment topology (as in the single-file template)
- A summary table per component or per group:

| Group | Artifacts | Detail |
|-------|-----------|--------|
| COMP4 — Desktop Editor | ART1–ART10 (10 artifacts) | [link to detail file] |
| COMP2 — API Service | ART11–ART12 (2 artifacts) | [link to detail file] |
| Cross-cutting | ART13 (NOTICE bundle), ART14 (SBOM) | [link to detail file] |

- Traceability matrix (project-wide; not duplicated in detail files)
- Quality assessment (project-wide)

Detail files (`docs/artifacts/<group>.{md,org}`) contain the per-artifact specifications for that group, in the same per-artifact field format as the single-file template.

---

## Identifier Reference Guide

Artifacts identifiers live alongside (but do not collide with) PRD, floorplan, contracts, and TDD identifiers. All are globally unique within the project.

| Entity | Prefix | Example | Notes |
|--------|--------|---------|-------|
| Artifact | ART | ART1, ART2, ART3 | Flat list — no dot notation |

**Existing identifiers in the namespace (do not reuse):** CAP, P, UC, UJ, FR, G, NG, AC (PRD); COMP, BD, DF, SL (floorplan); ENT, API, EVT (contracts); TR (TDDs); FT, EP (features).

---

## Important Constraints

- Your ONLY output is the artifacts document. Do NOT write build scripts, Dockerfiles, plists, installer configs, signing scripts, CI workflow files, or any artifact content.
- Do NOT invent platforms or components. Every platform must trace to a PRD AC, tech-plan choice, or floorplan-defined component. If a platform appears in code but not in the PRD, ask the user whether to add a PRD AC or to drop it from the matrix.
- Do NOT skip cross-platform analysis. When a project supports multiple platforms, parity is a first-class validation concern. A Windows-only auto-updater on a cross-platform desktop app is a parity gap, not an oversight.
- Do NOT list artifacts whose source inputs are entirely outside the repository without flagging the source-of-truth violation. The remediation is to bring inputs into the repo, not to silently document the external state.
- Do NOT confuse artifacts with components. A `.app` is an artifact produced *by* the desktop application component (COMP4). The component is the runtime concept; the artifact is the deliverable. Do not introduce new COMPs for artifacts.
- Do NOT load all starchitect documents in full upfront — load selectively based on scope. For project-wide runs, the PRD, floorplan, and tech-plan are required; feature PRDs only when scope is feature-specific.
- When artifacts already exist in the repository (Dockerfile present, plist present, electron-builder.yml present), catalogue them and carry their existing names, identifiers, and build mechanisms forward — do not propose alternatives unless the user asks.
- Prefer precision over verbosity. Cite specific paths, specific PRD ACs, specific COMPs. Avoid hand-waving phrases like "appropriate signing" — name the signing mechanism (Developer ID, Authenticode, cosign).
- For artifacts that depend on credentials/secrets (signing keys, API tokens, registry credentials), note the dependency but do NOT propose values. Secret provisioning is an operational concern, not an artifact specification concern.
