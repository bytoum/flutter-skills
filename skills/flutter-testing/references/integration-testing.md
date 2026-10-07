# Integration Testing

Use this workflow for a complete Flutter app or a large app flow running on a
target device. Keep application integration tests distinct from end-to-end
tests that require native system UI or uncontrolled external systems.

## Inspect Before Setup

Find the project's existing integration directory, harness, CI job, device ID,
platform configuration, application reset strategy, and backend fixture
strategy. Preserve an established `integration_test`, Patrol, or custom runner.
Add the Flutter SDK's `integration_test` dev dependency only when setup is
missing and dependency changes are in scope.

Derive application entrypoints, keys, reset hooks, fixture APIs, package IDs,
permissions, and runner flags from project code or tool output. When those facts
are unavailable, describe the capability the project needs and mark the command
or verification unresolved instead of presenting invented code as executable.

Standard SDK tests normally live under `integration_test/`, initialize
`IntegrationTestWidgetsFlutterBinding.ensureInitialized()`, and use
`flutter_test` APIs. Treat the host machine and target device as separate even
when web or desktop runs both roles on one machine.

When `integration_test/` does not exist, create the narrowest tree for the
requested flow. Follow package and feature boundaries from the application,
but organize tests by user journey rather than copying presentation, data, and
domain layers. Add support, fixture, or robot directories only when creating a
concrete reusable helper. Select the app bootstrap or test seam from repository
usage and configuration, not from a plausible filename.

## Make the Flow Deterministic

1. Start from explicit app, authentication, storage, and backend state.
2. Give fixtures unique identities and clean them up. Avoid shared staging
   users or records unless the requested test is explicitly end-to-end.
3. Select controls by stable semantics or keys rather than coordinates,
   localized text, or incidental widget structure.
4. Wait for an observable condition with a bounded diagnostic timeout. Prefer
   exact pumps for known transitions. Broad `pumpAndSettle()` can hang on
   continuous animations and should not replace a known readiness signal.
5. On failure, retain the useful artifacts already supported by the project,
   such as logs, screenshots, traces, and the device identifier.

Retries, longer sleeps, and larger timeouts may help reproduce a failure, but a
stable fix isolates shared state, real time, live network, animations, and
ambiguous readiness.

## Native and External Boundaries

Flutter's SDK integration APIs cannot control native system UI such as runtime
permission dialogs or notification surfaces. When the flow requires that UI,
use the project's native-capable harness. If none exists, explain the boundary
and propose a tool such as Patrol before adding dependencies or changing CI.
If the flow only requires the resulting permission, configure the test device
state rather than turning every app test into a native-UI test.

Use fake or controlled services for ordinary integration coverage. Real APIs,
credentials, and staging environments require explicit scope, deterministic
fixtures, cleanup, and clear failure reporting.

## Run on the Intended Target

Prefer the exact project or CI command. A common SDK command for a connected
mobile or desktop target is:

```bash
flutter test integration_test/checkout_test.dart -d <device-id>
```

Web driver setup and commands are especially version-sensitive; follow the
project's checked-in runner and current Flutter documentation rather than
hardcoding a ChromeDriver workflow. Confirm supported syntax with local command
help.

Run the narrow flow first, then the relevant integration directory or CI job on
the same platform and explicit device. Choose repeated runs in proportion to
the flake's frequency and risk; repetition can expose nondeterminism but does
not replace removing its cause. Report unavailable targets or services as
unverified.

## Official References

- [Flutter integration testing](https://docs.flutter.dev/testing/integration-tests)
- [`integration_test` introduction](https://docs.flutter.dev/cookbook/testing/integration/introduction)
