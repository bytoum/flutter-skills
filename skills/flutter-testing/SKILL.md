---
version: "1.0.2"
name: flutter-testing
description: Use when creating, running, diagnosing, reviewing, or stabilizing Flutter or Dart unit tests and Flutter application-level integration tests.
---

# Flutter Testing

Build confidence in observable behavior with the smallest deterministic test
that can fail for the intended reason. Match the project's Flutter version,
test stack, target platforms, and CI commands instead of introducing a parallel
testing convention.

## Workflow

1. Find the package root and inspect `pubspec.yaml`, SDK constraints, existing
   tests, test dependencies, generated-code conventions, configured platforms,
   and CI commands. Check `flutter --version` and local command help when syntax
   may vary by SDK version.
2. Classify the request:
   - For unit tests, read [references/unit-testing.md](references/unit-testing.md).
   - For application-level integration tests, read
     [references/integration-testing.md](references/integration-testing.md).
   - For ambiguous terms, use [GLOSSARY.md](GLOSSARY.md).
3. Define the behavior and the production change that would make the test fail.
   Before modifying an existing function, identify tests that exercise the
   behavior being changed. If none do, stop before editing production code,
   name the uncovered function and behavior, propose the focused test, and ask
   the user to confirm either test-first modification or proceeding without
   coverage. The original change request is not that confirmation.
   Follow the confirmed path. On the test-first path for a bug fix, add a
   regression test and observe the expected failure before changing production
   behavior. For a feature, follow the user's requested development order.
4. Implement the narrowest test and any minimal, behavior-preserving testability
   seam. Prefer existing dependencies and conventions. Introduce a package,
   native harness, test service, or CI/device configuration only when it is
   necessary and within the requested scope; otherwise explain the boundary and
   ask for direction.
5. Run the narrowest affected test first, then the relevant suite. Run an
   integration test on the same explicit target and device used by the project.
   If a required device, service, credential, or platform is unavailable, report
   that verification as incomplete.

## Decision Guide

| Situation | Default |
| --- | --- |
| No test-double convention exists | Real dependency, small fake/stub, manual fake, then a mocking package |
| Existing Mockito, Mocktail, Patrol, or custom harness | Preserve it unless the task requests migration |
| Network, clock, randomness, or stored state affects results | Inject and control the boundary |
| Test waits on elapsed time | Wait for an observable readiness condition with a bound |
| Integration flow needs native system UI | Use an established native-capable harness or propose one explicitly |
| Repository has a coverage gate | Honor it; otherwise test risk and behavior rather than a universal percentage |

## Boundaries

- Widget and golden testing are separate primary workflows. Use them only when
  they are incidental to a requested unit or integration test.
- Treat retries, longer timeouts, arbitrary sleeps, and broad
  `pumpAndSettle()` calls as diagnostics, not flake fixes.
- Keep live APIs, shared accounts, and mutable staging state out of ordinary
  tests. Use a controlled environment only when end-to-end behavior is explicit.
- Do not hand-edit generated mocks or pin package versions copied from examples.

## Common Mistakes

| Mistake | Correction |
| --- | --- |
| Adding a familiar framework immediately | Inspect the project's dependencies and use the lightest existing-compatible double. |
| Testing calls made to a mock | Assert user-visible or domain behavior; verify interactions only when the interaction is the contract. |
| Inventing a reset API, app entrypoint, key, or runner command | Resolve concrete seams and commands from the repository; otherwise describe the required capability as unverified. |
| Claiming integration success from a host-only run | Name the target device and use the project's actual integration command. |
| Hiding a flake with retries | Isolate state, time, network, and readiness, then reproduce with the recorded seed or target. |
