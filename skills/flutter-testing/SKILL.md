---
version: "1.2.0"
name: flutter-testing
description: Use when writing, running, diagnosing, reviewing, or stabilizing Flutter or Dart unit, widget, or golden tests — including requests like "add a test for this", "why is this test failing or flaky in CI", "update the goldens", "test this widget", or "review these tests". Not for integration, plugin/native platform testing, static analysis, manual QA, or building the feature itself.
---

# Flutter Testing

Build confidence in observable behavior with the smallest deterministic test
that can fail for the intended reason. Match the project's Flutter version,
test stack, target platforms, and CI commands instead of introducing a parallel
testing convention.

## Workflow

### 1. Discover the project

Find the package root and read `pubspec.yaml`, SDK constraints, existing tests,
test dependencies, generated-code conventions, and configured test commands.
Read checked-in scripts and CI to learn the real test commands, tags, sharding,
and coverage policy.

Select the toolchain from the repository before running anything. Look for FVM
(`.fvm/`, `.fvmrc`), Puro (`.puro/`), Melos (`melos.yaml`), or a script wrapper,
and use it. Use global `flutter` or `dart` only when the repository selects no
wrapper; a mismatched global SDK produces misleading failures.

Before running any wrapper command, including `--version`, `--help`, and
`dart format`, check that the pinned SDK is already installed with a command that
does not invoke the SDK (for example `fvm list` or `puro ls`). Wrappers
download a missing SDK on first use, which is a large, slow side effect on the
user's machine and can leave a half-installed SDK behind. If the pinned SDK is missing, ask before installing
it; if the user declines, report verification as incomplete rather than
substituting the global SDK silently. Confirm syntax with `--help` once the
right SDK is selected.

### 2. Classify the request

Pick the intent, then the test layer. Intent decides the steps; layer decides
which reference to read.

| Intent | What to do |
| --- | --- |
| Create or extend tests | Follow the layer reference, then the steps below. |
| Run tests | Use the repository command and report the result; do not rewrite tests. |
| Diagnose, review, or stabilize existing tests | Read [diagnosing-and-stabilizing.md](references/diagnosing-and-stabilizing.md). Do not apply creation-only steps such as writing a new regression test first. |

| Layer | Read |
| --- | --- |
| Pure Dart or Flutter logic, async, streams, state management | [unit-testing.md](references/unit-testing.md) |
| Rendered widgets, interaction, accessibility, layout variants | [widget-testing.md](references/widget-testing.md) |
| Pixel comparison against a baseline image | [golden-testing.md](references/golden-testing.md) |
Use [GLOSSARY.md](GLOSSARY.md) for ambiguous terms. If the request is only
about integration, plugin or native platform testing, static analysis, manual QA
steps, or implementing a feature with no test work, this skill does not apply.

### 3. Protect existing behavior (when changing production code)

Before modifying production code, find tests that exercise the behavior being
changed. If the behavior is uncovered, pause before changing production code
and explain the specific gap and risk. Ask the user to authorize a path; a
general request to fix or change the code does not bypass this confirmation.
Recommend adding the smallest focused test first and, for a bug fix, observing
it fail for the reported reason before applying the production change. Other
valid choices are to proceed without new coverage while accepting the risk, to
investigate a testability seam first, or to leave production code unchanged.
Follow the user's choice. For covered behavior, or when the user has already
authorized a specific path, proceed in the requested order.

Ask separately before a material scope expansion such as a new dependency,
production seam, service, credential, or CI change.

### 4. Implement narrowly

Write the narrowest test and any minimal, behavior-preserving testability seam.
Prefer existing dependencies and conventions. Add a package, harness, service,
or CI configuration only when the requested behavior requires it and the user
has authorized that scope; otherwise name the boundary and ask.

### 5. Verify and report

Run the narrowest affected test first, then the owning package suite, using the
selected toolchain. Apply the repository's formatter and analyzer when Dart
files changed. Preserve checked-in tags, sharding, and coverage policy; do not
present generic coverage thresholds as requirements.

End every task by reporting:

- the toolchain used (for example `fvm flutter`, or global `flutter 3.x`);
- the exact commands run;
- the test environment (for example, the Flutter test binding on the host);
- the observed result;
- the status of the wider suite;
- every verification that remains incomplete and why.

Do not imply full-suite verification when only a focused test ran. Report
platform-sensitive golden verification as incomplete when the required
environment is unavailable.

## Asking the User

Ask only at a real scope or authority boundary. When several meaningful paths
exist, give two to four mutually exclusive options, put the recommended one
first labeled **Recommended**, and give each a one-sentence tradeoff. When one
missing fact has one answer (such as a test-file path), ask for it directly.
Never manufacture options to fill a format.

## Decision Guide

| Situation | Default |
| --- | --- |
| No test-double convention exists | Real dependency, small fake/stub, manual fake, then a mocking package |
| Existing Mockito, Mocktail, Patrol, or custom harness | Preserve it unless the task requests migration |
| Network, clock, randomness, or stored state affects results | Inject and control the boundary |
| Production behavior to change has no existing coverage | Ask the user to authorize a path before changing production code; recommend a focused test first |
| Test waits on elapsed time | Wait for an observable readiness condition with a bound |
| Repository has a coverage gate | Honor it; otherwise test risk and behavior rather than a universal percentage |

## Boundaries

- Treat retries, longer timeouts, arbitrary sleeps, and broad `pumpAndSettle()`
  calls as diagnostics, not flake fixes.
- Keep live APIs, shared accounts, and mutable staging state out of ordinary
  tests. Use a controlled environment only when end-to-end behavior is explicit.
- Do not hand-edit generated mocks or pin package versions copied from examples.
- Update golden baselines only as an explicit, reviewed action, never to turn a
  failing check green.

## Common Mistakes

| Mistake | Correction |
| --- | --- |
| Running global `flutter` in an FVM/Puro/Melos repo | Use the repository's wrapper or script. |
| Adding a familiar framework immediately | Inspect the project's dependencies and use the lightest existing-compatible double. |
| Testing calls made to a mock | Assert user-visible or domain behavior; verify interactions only when the interaction is the contract. |
| Treating a request to fix uncovered behavior as production-change authorization | Explain the coverage gap and ask the user to authorize a path first. |
| Inventing a reset API, app entrypoint, key, flag, or runner command | Resolve concrete seams and commands from the repository; otherwise describe the required capability as unverified. |
| Hiding a flake with retries | Isolate state, time, network, and readiness, then reproduce with the recorded seed or runtime environment. |
