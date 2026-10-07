---
"flutter-skills": minor
---

Add a `flutter-testing` skill for Flutter and Dart testing. It routes requests
by intent (create, run, diagnose, review, stabilize) and by layer, with focused
workflows for unit, widget, golden, integration, and plugin or platform tests.
It selects the repository's FVM, Puro, Melos, or script toolchain before running
anything, adds a focused test for uncovered code by default instead of asking,
treats golden updates as explicit reviewed actions, diagnoses CI-only and flaky
failures from evidence, and reports the toolchain, target, and any unverified
checks instead of implying cross-platform or full-suite success.
