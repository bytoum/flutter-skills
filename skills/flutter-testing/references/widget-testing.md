# Widget Testing

Use this workflow to test rendered Flutter widgets with `testWidgets` and
`flutter_test`, without launching the full app on a device. Choose it over a
unit test when the behavior needs layout, gestures, focus, semantics, or inherited
widgets. Choose a [golden test](golden-testing.md) only when pixels are the
contract. This workflow covers widget behavior in the test binding, not full-app
flows on a device or real platform services.

## Build the Smallest Environment

Pump only the wrappers the behavior needs. Each extra layer is a place for
unrelated failures to come from.

- A bare widget that needs directionality or text usually needs
  `MaterialApp` or `WidgetsApp` around it, with `Scaffold` only if it needs one.
- Add a `Theme`, localization delegates, `Navigator`, or `MediaQuery` only when
  the widget reads them. Reuse the project's existing test wrapper or pump
  helper before writing a new one.
- Provide dependencies through the project's mechanism (Provider, Riverpod
  `ProviderScope`, `BlocProvider`, a service locator) with fakes at the
  boundary. Assert rendered output, not framework internals.
- Set the surface through the binding (`tester.view.physicalSize`,
  `devicePixelRatio`, text scale) and reset it in `addTearDown`.

## Interact and Synchronize

- Find by stable keys or semantics first, then by type or text. Avoid
  coordinates and incidental widget-tree structure; avoid localized text when
  a key or semantic label exists.
- Drive input with `tester.tap`, `enterText`, `drag`, `fling`, and
  `scrollUntilVisible`. Use `ensureVisible` before tapping off-screen targets.
- After an action, advance time deliberately: `pump()` for one frame,
  `pump(duration)` for a known transition, `pumpAndSettle()` only when the
  screen reaches a rest state. A looping animation or a repeating timer makes
  `pumpAndSettle()` hang, so pump an exact duration or stop the animation.
- Handle focus and forms through the focus tree and `TextEditingController`
  state, and assert validation messages that users see.
- Use fake async or the binding's clock for timers and debounce. Dispose
  controllers and finish owned animations before the test ends so no timer is
  left pending.
- Assert the observable result: found text, enabled state, scroll position,
  navigation outcome, or emitted callback value. Verify that an interaction
  happened only when the call is itself the contract.

## Accessibility and Layout Variants

Add these only when relevant to the request, and use APIs the project's Flutter
version supports:

- Semantics: assert labels, roles, and actions with `find.bySemanticsLabel` or
  `tester.getSemantics`, and use `meetsGuideline` checks
  (`androidTapTargetGuideline`, `iOSTapTargetGuideline`,
  `labeledTapTargetGuideline`, `textContrastGuideline`) through
  `expectLater(tester, meetsGuideline(...))`. Create semantics with
  `tester.ensureSemantics()` and dispose the handle.
- Text scale and long content: pump with a large `textScaler` and long
  translations, and assert no overflow exception and that key controls remain
  reachable.
- RTL and locales: wrap with `Directionality.rtl` or a locale and assert
  mirrored layout or localized strings.
- Viewports: test one narrow and one wide size when the layout is responsive,
  not every device.

## Verify

Run the focused file or named test first, using the selected toolchain, then the
owning package suite:

```bash
flutter test test/widgets/example_widget_test.dart
flutter test --plain-name "shows error when email is invalid"
```

Replace `flutter` with the repository's wrapper (`fvm flutter`, a Melos script,
and so on). Confirm flags with `flutter test --help`. Report that widget tests
run in the test binding on the host, not on a device or browser.
