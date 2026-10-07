# Flutter Testing Glossary

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

## Test harness

The runner and support code that launches tests and provides their environment,
such as `package:test`, `flutter_test`, or a project's custom runner.

## Unit test

A test of one function, method, class, or small collaboration in process, with
unstable dependencies controlled at its boundary.

## Widget test

A test that builds Flutter widgets in the test binding, without a device or the
full app, and asserts what they render or how they respond to interaction.
