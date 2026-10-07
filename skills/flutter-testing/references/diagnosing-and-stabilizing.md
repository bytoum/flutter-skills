# Diagnosing, Reviewing, and Stabilizing Tests

Use this workflow when tests already exist and the request is to understand a
failure, review test quality, or make a test reliable. Do not start by writing a
new test. Start by getting evidence. Answer the question asked first:
when the user asks why something fails or what is weak, report the cause or
findings and offer the fix; edit files only when asked to fix, stabilize, or
change them.

## Classify the Failure

Name the class before proposing a fix; each class has a different remedy.

| Class | Typical signal |
| --- | --- |
| Dependency or setup | `pub get` errors, version solving, missing generated files, wrong SDK |
| Compilation | Analyzer or compiler errors before any test runs |
| Assertion | Test ran and an expectation failed |
| Timeout or hang | No result within the bound; often a pending timer or `pumpAndSettle()` loop |
| Process or device crash | Runner or app exits, device disconnects |
| Infrastructure | Runner, emulator boot, network to a registry, resource limits |
| Flaky or order-dependent | Passes alone, fails in the suite, or varies run to run |

A wrong toolchain is the cheapest cause to rule out: confirm the SDK matches the
repository's FVM, Puro, or script selection before investigating further.

## Reproduce Narrowly

1. Record the exact failing command, file, test name, target, and random seed
   from the log or CI output.
2. Run the narrowest form first, for example one file with `--plain-name`.
3. Keep the seed (`--test-randomize-ordering-seed`) and the target the same.
4. Compare local and CI: SDK version, OS, locale, timezone, flavor,
   `--dart-define` values, tags, shard, and concurrency.
5. Broaden only once it reproduces, or once you can state where the evidence
   stops (for example, a CI-only failure you cannot reproduce locally).

## Isolate Flakes

Inspect the unstable boundary before touching timeouts: shared or global state,
singletons, clocks, randomness, ports, files, network, animations, pending
timers and futures, test order, parallelism, device state, locale, and
platform. Fix the cause, then use repetition to measure the result, in
proportion to how often it failed and what it protects. Repetition shows that
nondeterminism exists; it does not remove it. Retries, longer timeouts, and
sleeps are diagnostics, not fixes.

Reproduce order dependence with the recorded seed and run the suspect test both
alone and after its neighbors.

## Review Existing Tests

Check, in order:

- Can the assertion fail for the intended reason? Mutate the code mentally or
  actually; a test that cannot fail is a false positive.
- Does it assert behavior, or only that a mock was called?
- Overspecified interactions, brittle selectors (coordinates, localized text),
  hidden sleeps, leaked state between tests, broad setup, duplicated fixtures,
  and names that do not describe the behavior.
- Does it rely on live network, wall-clock time, or unseeded randomness?

Report findings ranked by risk, each pointing at a file and line, and say which
are evidence and which are inference.

## Preserve and Report Evidence

Keep what the project already produces: logs, screenshots, traces, seeds, device
IDs, golden failure images, and machine-readable results such as
`--file-reporter json:<path>` where supported. Say where each artifact was
written. Separate what was observed from what is inferred, and state any run you
could not perform. Apply the fix only after the cause is classified, and
verify with the narrow command first, then the owning suite.
