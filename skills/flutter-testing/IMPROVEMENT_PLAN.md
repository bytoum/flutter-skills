# Flutter Testing Skill Improvement Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use
> `superpowers:subagent-driven-development` (recommended) or
> `superpowers:executing-plans` to implement this plan task-by-task. Steps use
> checkbox (`- [ ]`) syntax for tracking.

**Goal:** Expand `flutter-testing` into a complete, predictable Flutter and
Dart testing skill while preserving project conventions and keeping each test
workflow focused.

**Architecture:** Keep `SKILL.md` as a lean router containing shared principles
and completion criteria. Put test-type and diagnostic guidance in focused files
under `references/`, loaded only when the request reaches that branch.

**Tech Stack:** Markdown, YAML frontmatter, Flutter SDK test tools,
`package:test`, `flutter_test`, `integration_test`, and project-established
test harnesses.

**Spec:** The scope, constraints, and missing-case matrix in this document,
derived from the audit of version `1.0.3`.

## Global Constraints

- Keep the skill name and directory name `flutter-testing`.
- Keep skill-specific files inside `skills/flutter-testing/`.
- Prefer repository commands, wrappers, dependencies, and conventions over
  generic examples.
- Resolve version-sensitive syntax from the selected project toolchain and
  local command help.
- Add dependencies, native harnesses, services, or CI configuration only when
  the requested behavior requires them and the user has authorized that scope.
- Keep `SKILL.md` as the router and place branch-specific detail in referenced
  files.
- Increment the skill version from `1.0.3` to `1.1.0` because the implemented
  scope adds primary testing workflows.
- Update `.changeset/calm-tests-travel.md`, which already describes the
  unreleased `flutter-testing` addition, instead of creating a duplicate
  changeset.
- Do not use `npm test`; the repository defines it as a failing placeholder.

## Review Focus

- A request for a widget or golden test must route to a complete primary
  workflow rather than being treated as incidental.
- A request to diagnose, review, run, or stabilize existing tests must not be
  forced through a test-creation workflow.
- Projects using FVM, Puro, Melos, or checked-in scripts must not accidentally
  use a mismatched global Flutter SDK.
- An uncovered production function must receive a focused test by default when
  that work is already in scope, without an unnecessary permission loop.
- A passing test on one target must not be reported as cross-platform or
  full-suite verification.

---

## Target File Structure

```text
skills/flutter-testing/
├── SKILL.md
├── GLOSSARY.md
├── IMPROVEMENT_PLAN.md
└── references/
    ├── unit-testing.md
    ├── widget-testing.md
    ├── golden-testing.md
    ├── integration-testing.md
    ├── diagnosing-and-stabilizing.md
    └── plugin-and-platform-testing.md
```

The implementation should create only the references whose branches need
independent instructions. If two proposed references remain short and always
trigger together, co-locate them rather than preserving this tree mechanically.

## Missing-Case Coverage Matrix

| Missing case | Destination | Expected behavior |
| --- | --- | --- |
| Widget lifecycle, finders, gestures, scrolling, focus, forms, and animations | `references/widget-testing.md` | Use `testWidgets`, construct the smallest valid widget environment, pump deliberately, and assert observable UI or semantics. |
| Golden images, fonts, assets, pixel ratio, platform drift, and review policy | `references/golden-testing.md` | Make rendering deterministic and update baselines only as an explicit, reviewable action. |
| Accessibility, semantics, contrast, tap targets, text scale, RTL, and localization | `references/widget-testing.md` | Test semantics and representative accessibility/layout variants using project-supported APIs. |
| Streams, isolates, cancellation, debounce, concurrency, and pending timers | `references/unit-testing.md` | Control scheduling, await owned work, and verify cleanup and terminal states. |
| Empty, malformed, boundary, Unicode, timezone, DST, and deterministic-random inputs | `references/unit-testing.md` | Select cases from the behavior's risk boundaries rather than a universal checklist. |
| State-management conventions such as Bloc, Riverpod, Provider, or custom state machines | `references/unit-testing.md` and `references/widget-testing.md` | Preserve the project's existing test layer and assert emitted or rendered behavior. |
| Failure classification, CI-only failures, hangs, flakes, order dependence, and false positives | `references/diagnosing-and-stabilizing.md` | Reproduce narrowly, classify the failure, preserve evidence, isolate the unstable boundary, and verify the fix repeatedly when risk warrants it. |
| Plugin APIs, `MissingPluginException`, federated interfaces, and platform channels | `references/plugin-and-platform-testing.md` | Prefer an application-owned wrapper, then the public API or platform interface, and use channel mocks only at the appropriate boundary. |
| Native Android, iOS, macOS, Linux, and Windows tests | `references/plugin-and-platform-testing.md` | Route native implementation behavior to the platform's established native test harness. |
| Navigation, deep links, restoration, lifecycle, background/resume, and interrupted flows | `references/integration-testing.md` | Start from explicit state and verify the complete device-level journey on the intended target. |
| Offline behavior, latency, timeout, retry, malformed responses, and connection loss | Unit or integration reference according to boundary | Use controlled network behavior and assert user-visible or domain outcomes. |
| Flavors, `--dart-define`, alternate entrypoints, and test configuration | `SKILL.md` inspection step and integration reference | Derive configuration from repository scripts and CI rather than inventing flags. |
| Web/browser differences, desktop targets, Linux display servers, device farms, and physical devices | `references/integration-testing.md` | Use the project's platform command and report the exact browser, desktop target, simulator, emulator, or device. |
| Native dialogs, notifications, platform views, permissions, and device-state manipulation | Integration and plugin/platform references | Use an established native-capable harness or configure preconditions when direct interaction is unnecessary. |
| Performance timelines, profile mode, startup, scrolling, and frame regressions | `references/integration-testing.md` | Use supported profiling workflows and distinguish functional success from performance evidence. |
| Screenshots, traces, logs, machine-readable results, and `reportData` | Integration and diagnostic references | Retain supported artifacts and identify where each artifact was written. |
| Tags, sharding, concurrency, randomized order, coverage, and coverage merging | `SKILL.md` verification and diagnostic reference | Preserve checked-in CI policy and avoid presenting generic thresholds as requirements. |
| Mockito generation, stale outputs, and monorepo generation order | `references/unit-testing.md` | Change annotated sources and run the repository's established generation command. |
| Legacy `flutter_driver` migration | `references/integration-testing.md` | Identify the legacy harness and propose a scoped migration to current project-compatible tooling. |

## Task 1: Redesign the Router and Trigger Description

**Files:**

- Modify: `skills/flutter-testing/SKILL.md`
- Modify: `skills/flutter-testing/GLOSSARY.md`

**Interfaces:**

- Consumes: Existing unit and integration references plus the coverage matrix
  in this plan.
- Produces: A router that selects both the request intent and the relevant test
  type before loading a focused reference.

- [ ] **Step 1: Rewrite the frontmatter description.**

  Cover creation, execution, diagnosis, review, and stabilization for Dart
  unit, Flutter widget, golden, integration, plugin, and platform-related
  tests. Include discriminating trigger language without turning the
  description into a keyword list.

- [ ] **Step 2: Add two-stage classification to `SKILL.md`.**

  First classify intent: create, run, diagnose, review, or stabilize. Then
  classify test layer: unit, widget, golden, integration, plugin/platform, or
  end-to-end/native UI. Each branch must name the reference to read.

- [ ] **Step 3: Add toolchain discovery.**

  Inspect checked-in scripts and CI for FVM, Puro, Melos, or another wrapper;
  use global `flutter` or `dart` only when the repository has no selected
  wrapper.

- [ ] **Step 4: Replace the unconditional uncovered-function stop.**

  When a requested production change lacks coverage, add the smallest focused
  test by default if testing is already within scope. Ask the user only when
  reliable coverage requires a material seam, dependency, harness, service,
  device, credential, or CI expansion.

- [ ] **Step 5: Narrow decision prompts.**

  Provide multiple choices only when a real decision has multiple meaningful
  paths. Permit one direct question when a missing fact has one answer.

- [ ] **Step 6: Add completion criteria.**

  Require reporting the toolchain, command, target, observed result, relevant
  suite status, and every verification that remains incomplete.

- [ ] **Step 7: Update the glossary.**

  Define widget test, golden test, native test, performance test, test harness,
  flake, and hermetic test. Keep definitions aligned with routing decisions.

- [ ] **Step 8: Validate the router.**

  Check every reference link, confirm the frontmatter name remains
  `flutter-testing`, and confirm the version is `"1.1.0"`.

## Task 2: Add the Widget-Testing Workflow

**Files:**

- Create: `skills/flutter-testing/references/widget-testing.md`
- Modify: `skills/flutter-testing/SKILL.md`

**Interfaces:**

- Consumes: Shared determinism and project-convention rules from `SKILL.md`.
- Produces: The primary workflow for testing rendered Flutter widgets without
  launching the complete application on a target device.

- [ ] **Step 1: Define selection boundaries.**

  Distinguish widget tests from pure unit tests, golden tests, and full-app
  integration tests using observable behavior and required runtime context.

- [ ] **Step 2: Specify the minimum widget environment.**

  Cover app shells, themes, localization, navigation, inherited dependencies,
  media size, orientation, and text scale. Require only the wrappers needed by
  the behavior under test.

- [ ] **Step 3: Specify interactions and synchronization.**

  Cover stable finders, gestures, scrolling, focus, text input, animations,
  exact pumps, bounded readiness, and safe use of `pumpAndSettle()`.

- [ ] **Step 4: Add accessibility and layout cases.**

  Cover semantics, labels, tap targets, contrast, RTL, long translations,
  large text, and representative viewport sizes when relevant to the request.

- [ ] **Step 5: Add widget-test verification guidance.**

  Run the focused file or named test first, then the owning package suite using
  the selected repository toolchain.

## Task 3: Add the Golden-Testing Workflow

**Files:**

- Create: `skills/flutter-testing/references/golden-testing.md`
- Modify: `skills/flutter-testing/SKILL.md`

**Interfaces:**

- Consumes: Widget environment rules from `widget-testing.md`.
- Produces: A deterministic visual-regression workflow with an explicit
  baseline-update and review boundary.

- [ ] **Step 1: Define when a golden is appropriate.**

  Use goldens for meaningful visual contracts and use widget assertions for
  behavior that does not require pixel comparison.

- [ ] **Step 2: Define rendering controls.**

  Account for fonts, assets, images, animations, clocks, viewport, device pixel
  ratio, themes, locales, and platform-specific rendering.

- [ ] **Step 3: Define failure diagnosis.**

  Distinguish intended UI changes, environmental drift, missing assets or
  fonts, and true regressions before changing a baseline or tolerance.

- [ ] **Step 4: Protect baseline updates.**

  Treat `--update-goldens` or framework equivalents as intentional output
  changes that require diff review. Never use baseline regeneration merely to
  make a failing check pass.

- [ ] **Step 5: Define CI ownership.**

  Use the repository's canonical generation platform and comparison policy;
  report when the local environment cannot reproduce it.

## Task 4: Expand Unit and Async Coverage

**Files:**

- Modify: `skills/flutter-testing/references/unit-testing.md`

**Interfaces:**

- Consumes: Toolchain and completion rules from `SKILL.md`.
- Produces: Risk-based unit-test guidance for synchronous, asynchronous, and
  generated-test environments.

- [ ] **Step 1: Replace the single-behavior checklist with risk-based cases.**

  Select the happy path, meaningful boundaries, contractually visible errors,
  and the regression condition. Avoid exhaustive case lists when they do not
  affect behavior.

- [ ] **Step 2: Cover asynchronous behavior.**

  Add streams, subscriptions, cancellation, debouncing, isolates, concurrent
  operations, pending timers, and teardown of owned resources.

- [ ] **Step 3: Cover environmental boundaries.**

  Add UTC and timezone-sensitive behavior, DST boundaries, deterministic
  randomness, filesystem cleanup, malformed data, Unicode, and numeric limits
  when relevant.

- [ ] **Step 4: Cover state-management conventions.**

  Preserve the project's established Bloc, Riverpod, Provider, or custom state
  testing style and assert emitted domain or UI state rather than framework
  internals.

- [ ] **Step 5: Generalize generated-test artifacts.**

  Preserve Mockito guidance while also handling other checked-in generators
  and monorepo generation order through repository commands.

## Task 5: Add Diagnosis, Review, and Stabilization Workflows

**Files:**

- Create: `skills/flutter-testing/references/diagnosing-and-stabilizing.md`
- Modify: `skills/flutter-testing/SKILL.md`

**Interfaces:**

- Consumes: Exact commands and target information gathered by the router.
- Produces: A repeatable classification and evidence loop for existing test
  failures and test-quality reviews.

- [ ] **Step 1: Add failure classification.**

  Classify dependency/setup, compilation, assertion, timeout, hang, process or
  device crash, infrastructure, and flaky/order-dependent failures before
  proposing a fix.

- [ ] **Step 2: Add a reproduction ladder.**

  Start with the narrowest recorded command, preserve the seed and target,
  compare local and CI environments, and broaden only after the failure is
  reproducible or the evidence boundary is clear.

- [ ] **Step 3: Add flake isolation.**

  Inspect shared state, clocks, randomness, ports, files, network, animations,
  pending work, test order, parallelism, device state, locale, and platform.
  Use repetition to measure the outcome after addressing the suspected cause.

- [ ] **Step 4: Add the review checklist.**

  Check that assertions can fail for the intended reason; flag false positives,
  overspecified interactions, brittle selectors, hidden sleeps, leaked state,
  broad setup, duplicated fixtures, and misleading test names.

- [ ] **Step 5: Add artifact and reporting guidance.**

  Preserve logs, screenshots, traces, seeds, device identifiers, and
  machine-readable results already produced by the project. Report evidence
  separately from inference.

## Task 6: Expand Integration, Platform, and Plugin Coverage

**Files:**

- Modify: `skills/flutter-testing/references/integration-testing.md`
- Create: `skills/flutter-testing/references/plugin-and-platform-testing.md`
- Modify: `skills/flutter-testing/SKILL.md`

**Interfaces:**

- Consumes: Shared target and determinism rules from `SKILL.md`.
- Produces: Platform-aware application, plugin, native, and performance test
  guidance.

- [ ] **Step 1: Add application-flow cases.**

  Cover navigation, deep links, restoration, background/resume, persisted
  state, offline behavior, controlled failures, and interrupted operations.

- [ ] **Step 2: Add configuration discovery.**

  Resolve flavors, `--dart-define` values, alternate entrypoints, package IDs,
  permissions, and reset hooks from repository sources.

- [ ] **Step 3: Add platform execution branches.**

  Cover mobile devices and simulators, web browsers, macOS, Windows, Linux
  display requirements, physical devices, and checked-in device-farm jobs.

- [ ] **Step 4: Add native-UI boundaries.**

  Route permission dialogs, notifications, platform views, and device-state
  manipulation to the established native-capable harness. Prefer preconfigured
  state when the UI interaction is not itself the contract.

- [ ] **Step 5: Add plugin-package routing.**

  Distinguish app code using a plugin from a plugin package testing its Dart,
  platform-channel, native implementation, and example application layers.

- [ ] **Step 6: Add native-test discovery.**

  Preserve established JUnit, XCTest, GoogleTest, Espresso, XCUITest, or custom
  commands rather than translating native behavior into Dart-only tests.

- [ ] **Step 7: Add performance and artifacts.**

  Cover profile-mode timelines, startup and scrolling measurements, supported
  screenshots, traces, logs, and result data. State when a platform does not
  support the requested measurement.

- [ ] **Step 8: Add legacy-harness handling.**

  Detect `flutter_driver`; preserve it for scoped maintenance or propose a
  separate migration instead of silently mixing harnesses.

## Task 7: Add Evaluation Cases and Validate the Skill

**Files:**

- Create: `skills/flutter-testing/evals/evals.json`
- Modify: `.changeset/calm-tests-travel.md`

**Interfaces:**

- Consumes: All updated skill and reference files.
- Produces: Evidence that the router selects the correct branch and that each
  workflow changes agent behavior usefully.

- [ ] **Step 1: Add representative positive evaluation prompts.**

  Include at least one realistic prompt for unit, widget, golden, integration,
  plugin, diagnosis, review, and flake stabilization. Each prompt must name the
  expected workflow and critical output behavior.

- [ ] **Step 2: Add near-miss trigger prompts.**

  Include static analysis, manual QA, visual design implementation, native-only
  application testing, and generic Dart coding requests that should not invoke
  this skill.

- [ ] **Step 3: Run with-skill and baseline evaluations.**

  Snapshot version `1.0.3` as the baseline, run both configurations for every
  prompt, and record outputs and timing according to the skill-creator workflow.

- [ ] **Step 4: Grade objective requirements.**

  Check correct branch selection, repository command discovery, absence of
  invented project facts, explicit target reporting, deterministic boundaries,
  and honest unverified status.

- [ ] **Step 5: Generate the review viewer.**

  Aggregate benchmark results and generate the skill-creator review artifact so
  a human can compare outputs before accepting the revision.

- [ ] **Step 6: Validate repository requirements.**

  Confirm valid YAML frontmatter, matching directory/frontmatter name, working
  relative links, quoted semantic version, portable commands, and no generated
  or unrelated files. Do not run `npm test`.

- [ ] **Step 7: Update the existing changeset.**

  Describe the expanded widget, golden, diagnostic, plugin, platform, and
  stabilization behavior from the user's perspective while retaining a minor
  package bump.

## Final Acceptance Criteria

- `SKILL.md` routes every supported intent and test layer to one authoritative
  workflow.
- Widget and golden testing are first-class branches.
- Diagnostic and review requests no longer inherit creation-only steps.
- FVM, Puro, Melos, and repository scripts take precedence over global tools.
- Unit guidance covers relevant async, isolation, generated-code, and boundary
  cases without forcing irrelevant cases into every test.
- Integration guidance identifies the exact platform and device and covers
  configuration, native UI, performance, and artifact boundaries.
- Plugin guidance distinguishes application wrappers, Dart plugin APIs,
  platform channels, native implementations, and example-app integration.
- User questions occur only at real scope or authority boundaries.
- Evaluation results demonstrate an improvement over version `1.0.3` without
  increasing invented commands, dependencies, or project assumptions.
- The skill version and existing changeset describe the expanded published
  behavior accurately.
