# Unit Testing

Use this workflow for a function, method, class, or small collaboration whose
dependencies can be controlled in-process.

## Choose the Existing Runner

- Use `package:test` for pure Dart packages and tests that need no Flutter
  binding.
- Use the SDK's `flutter_test` when the code imports Flutter or needs Flutter
  test bindings.
- Place ordinary tests under the owning package's `test/` directory with names
  ending in `_test.dart`. When no tests exist, create only the paths needed for
  the requested tests and mirror their paths under `lib/`. In a monorepo, keep
  each test with its package; create a root `test/` only when the root is itself
  a Dart or Flutter package.
- Add shared fixtures, fakes, or helpers when the first concrete reuse appears,
  following an established project location when one exists.
- Discover project-specific flags from CI and `dart test --help` or
  `flutter test --help`; do not transplant flags from another SDK version.

## Design the Test

1. Name one observable behavior, boundary, or error case.
2. Choose cases by risk: the happy path, the boundaries where the behavior
   changes, errors the contract exposes, and the regression condition. Skip
   cases that cannot change the outcome; a universal checklist adds noise.
3. Identify the production change that would make the assertion fail.
4. Arrange only the state needed for that behavior.
5. Act once and assert the returned value, emitted state, persisted result, or
   domain error. Verify calls only when making the call is itself the contract.
6. For a regression, run the focused test before the fix and confirm it fails
   for the reported bug rather than setup, compilation, or an unrelated error.

When relevant to the behavior, consider empty and malformed input, numeric
limits, Unicode and grapheme boundaries, timezone and DST transitions (use
explicit UTC instants unless timezone is the subject), and seeded randomness.

## Uncovered Functions

Before changing an existing function, search the relevant tests for direct and
indirect coverage of the behavior that will change. Use an existing coverage
report when available, but do not require one when test code and execution
establish the answer.

When the behavior is uncovered and testing is in scope, add the smallest
characterization or regression test by default and, for a bug fix, observe it
fail before changing the function. No permission is needed for that. Ask only
when reliable coverage needs a material seam, new dependency, harness, service,
device, credential, or CI change, and state the options with a recommendation.

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

When the project uses generated code (Mockito `*.mocks.dart`, `freezed`,
`json_serializable`, or another generator), change the annotated source and run
the repository's established generation command, not a guessed one. Treat
generated files as outputs; never hand-edit them, and regenerate stale ones
before diagnosing a failure. In a monorepo, follow the repository's package
generation order.

When the code uses Bloc, Riverpod, Provider, or a custom state machine, keep
the project's existing test layer (for example `bloc_test`, a `ProviderContainer`
with overrides) and assert emitted domain or UI state rather than framework
internals.

## Async and Isolation

- Await the behavior the test owns; expose an explicit readiness future instead
  of sleeping.
- Streams: collect with `expectLater(stream, emitsInOrder([...]))` or
  `emitsDone`, and assert the terminal state (done or error), not only the first
  event. Cancel subscriptions you open.
- Cancellation, debounce, and concurrency: use controlled clocks or
  `fake_async` when the project supports them and timing is the subject. Start
  overlapping operations explicitly and assert the outcome, not the schedule.
- Isolates: await completion and shut down owned isolates in teardown so no
  work outlives the test.
- Restore mutated globals, bindings, and temporary resources (files, ports) in
  teardown, and make sure no timer or future is left pending.
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
