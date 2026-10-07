# Flutter Testing Glossary

## End-to-end test

A test that exercises an application together with external or native systems
across the full user journey. It may require capabilities beyond Flutter's
standard application integration harness.

## Flake

A test that passes and fails on the same code. Treat it as a defect in the test
or the code under test, caused by shared state, time, randomness, ordering,
network, or unclear readiness, not as noise to retry away.

## Golden test

A widget test that compares rendered pixels to a checked-in baseline image.
Use it for a visual contract; use ordinary widget assertions for behavior.

## Hermetic test

A test whose result depends only on its own inputs: no live network, shared
accounts, wall-clock time, unseeded randomness, or state left by other tests.

## Host machine

The computer that launches a test and collects its result.

## Integration test

A test of a complete Flutter application or a substantial application flow on
a target device. It may replace external systems with controlled boundaries.

## Native test

A test of Android, iOS, macOS, Linux, or Windows implementation code, written
in the platform's own harness (JUnit, XCTest, GoogleTest, and similar) and run
by the platform's own tooling rather than by `flutter test`.

## Performance test

A test that measures timing, frame, or startup behavior, typically in profile
mode on a real target. It produces evidence about speed; passing functionally
does not establish performance.

## Target device

The emulator, simulator, browser, desktop environment, or physical device on
which the Flutter application under test runs.

## Test harness

The runner and support code that launches tests and provides their environment,
such as `flutter_test`, `integration_test`, Patrol, or a project's custom
runner.

## Unit test

A test of one function, method, class, or small collaboration in process, with
unstable dependencies controlled at its boundary.

## Widget test

A test that builds Flutter widgets in the test binding, without a device or the
full app, and asserts what they render or how they respond to interaction.
