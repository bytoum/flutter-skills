# Golden Testing

Use this workflow for visual regression tests that compare a rendered widget to
a checked-in baseline image with `matchesGoldenFile`. Build the widget
environment with the rules in [widget-testing.md](widget-testing.md) first.

## When a Golden Is Appropriate

Use a golden when the visual result is the contract: a design-system component,
a themed state, a custom painter, or a layout that is hard to assert with
finders. Use ordinary widget assertions for behavior, text, and state. A golden
that protects behavior also fails on every incidental visual change and trains
reviewers to approve diffs without reading them.

## Make Rendering Deterministic

A golden that differs between machines is a flake. Control every input the
pixels depend on:

- Fonts: Flutter tests render with a placeholder font unless fonts are loaded.
  Load the project's real fonts through its existing loader (for example a
  `loadAppFonts` helper or a `flutter_test_config.dart`) and do not invent one.
- Assets and images: use in-memory or bundled assets, never the network, and
  wait for decoding to finish before the comparison.
- Surface: fix the logical size and `devicePixelRatio`, and reset them in
  `addTearDown`.
- Theme, locale, text scale, and platform: set them explicitly; do not inherit
  defaults.
- Time and animation: pump a fixed duration or stop the animation, and use a
  fixed clock for anything date-dependent.
- Platform drift: text and anti-aliasing differ by operating system. Generate
  and compare baselines on the platform the repository designates.

## When a Golden Fails

Look at the failure images (`test/failures/` holds the masked and isolated
diffs) before changing anything, then classify the cause:

1. **Intended UI change.** The diff matches the change being made. Update the
   baseline deliberately.
2. **Environmental drift.** Fonts, platform, Flutter version, or pixel ratio
   differ from the baseline's generator. Fix the environment, not the baseline.
3. **Missing asset or font.** Blank boxes or placeholder glyphs. Fix loading.
4. **True regression.** Fix the production code.

Do not widen a comparator tolerance or regenerate baselines to make a failing
check pass.

## Update Baselines Deliberately

Regenerating is an intentional output change. Scope it to the affected files,
then review the image diff before finishing:

```bash
flutter test --update-goldens test/golden/example_golden_test.dart
```

Use the repository's wrapper and the exact path of the test you mean to update.
Updating the whole suite hides unrelated regressions. Report which baselines
changed and what was reviewed. Do not commit regenerated images you did not
inspect.

## CI Ownership

Use the repository's canonical generation platform and comparison policy. If a
checked-in script, a Docker image, or a CI job owns baseline generation, run or
reference that. When the local machine cannot reproduce the CI environment (for
example baselines are generated on Linux and you are on macOS), say so and report
the golden result as unverified against CI rather than updating baselines locally.
