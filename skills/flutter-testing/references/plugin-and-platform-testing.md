# Plugin and Platform Testing

Use this workflow for code that crosses into platform channels, Flutter plugins,
or native implementations. First decide which side of the boundary the request is
on.

## Application Using a Plugin vs. a Plugin Package

- **App code that uses a plugin.** Test your code, not the plugin. Prefer an
  application-owned wrapper interface and fake it in unit and widget tests. Only
  when no wrapper exists, use the plugin's platform interface or its published
  test fakes, and mock a method channel as the last resort.
- **A plugin package.** It has layers; test each where it lives: the Dart API,
  the platform interface and federated packages, the method channel, the native
  implementation, and the example app.

## Test the Dart Side

- Unit-test the public Dart API against a fake platform interface (set
  `<Plugin>Platform.instance` to a fake) before touching channels.
- When testing the channel layer itself, register a mock handler with
  `TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger.setMockMethodCallHandler`
  and clear it in `addTearDown`. Assert the method name and arguments sent and
  the Dart value returned. Check the SDK version's current API with the analyzer
  because this surface has changed across releases.
- A `MissingPluginException` in a test means no platform implementation is
  registered on that runner: the unit test host VM has no plugins. Fix it by
  faking the wrapper or interface, not by catching the exception.

## Native Implementation Tests

Route native behavior to the platform's own harness and its established command
instead of translating it into Dart:

- Android: JUnit/Robolectric unit tests or Espresso, run through Gradle in
  `android/` or the example app.
- iOS and macOS: XCTest or XCUITest, run through `xcodebuild test` against the
  example app's workspace.
- Linux and Windows: GoogleTest (or the project's runner) through CMake/CTest.

Find the checked-in command in the plugin's README, scripts, or CI; do not invent
schemes, targets, or paths. If the harness does not exist, describe what it would
cover and ask before adding native test targets, since that changes platform
project files.

## Integration Through the Example App

Exercise real plugin behavior on a target with an `integration_test` in the
plugin's example app, following [integration-testing.md](integration-testing.md).
Name the platform and device actually used; a passing Android run says nothing
about iOS or web.

## Native System UI and Device State

Permission dialogs, notifications, and platform views are outside Flutter's
standard harness. Prefer setting the precondition (grant the permission through
device configuration) when the dialog is not itself the contract. When the dialog
is the contract, use an established native-capable harness such as Patrol, or
propose one before adding it.

## Report

State which layer was tested (Dart API, channel, native, example app), the
platform and target, and which other platforms remain unverified.
