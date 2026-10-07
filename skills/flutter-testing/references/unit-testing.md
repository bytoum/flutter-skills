# Unit Testing

Use this workflow for a function, method, class, or small collaboration whose
dependencies can be controlled in-process.

## Choose the Existing Runner

- Use `package:test` for pure Dart packages and tests that need no Flutter
  binding.
- Use the SDK's `flutter_test` when the code imports Flutter or needs Flutter
  test bindings.
- Place ordinary tests under `test/` with names ending in `_test.dart`. Mirror
  the production layout when the repository already does so.
- Discover project-specific flags from CI and `dart test --help` or
  `flutter test --help`; do not transplant flags from another SDK version.

## Design the Test

1. Name one observable behavior, boundary, or error case.
2. Identify the production change that would make the assertion fail.
3. Arrange only the state needed for that behavior.
4. Act once and assert the returned value, emitted state, persisted result, or
   domain error. Verify calls only when making the call is itself the contract.
5. For a regression, run the focused test before the fix and confirm it fails
   for the reported bug rather than setup, compilation, or an unrelated error.

## Control Dependencies

Prefer, in order: the real dependency when it is fast and deterministic, a
small in-memory fake or stub, an existing tested fake, a manual fake, then the
project's mocking framework. Preserve established Mockito or Mocktail patterns.
If no framework exists, do not add one merely to avoid a small fake.

Inject unstable boundaries such as HTTP clients, clocks, randomness, storage,
and platform adapters. Capture time once for a decision and use explicit UTC
instants when timezone behavior is not under test. For plugin-backed code,
prefer wrapping the plugin behind an application-owned interface; mock platform
interfaces or channels only when a higher boundary is unavailable.

When the project uses generated Mockito mocks, update the annotated source and
run its established `build_runner` command. Treat generated `*.mocks.dart`
files as outputs.

## Async and Isolation

- Await the behavior the test owns; expose an explicit readiness future instead
  of sleeping.
- Use controlled clocks or `fake_async` when the project already supports them
  and timer behavior is the subject.
- Restore mutated globals, bindings, and temporary resources in teardown.
- Use a recorded randomization seed to reproduce order dependence. Fix shared
  state before changing concurrency, timeouts, or retry counts.

## Verification

Run the repository's exact commands when available. Typical focused commands
are:

```bash
flutter test test/path/to/example_test.dart
dart test test/path/to/example_test.dart
```

Then run the relevant package suite. Apply the repository's formatter and
analyzer commands when production or test Dart files changed. Report the exact
commands and any checks that could not run.
