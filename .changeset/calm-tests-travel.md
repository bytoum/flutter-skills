---
"flutter-skills": minor
---

Add a `flutter-testing` skill for Dart and Flutter unit, widget, and golden
tests, plus diagnosing, reviewing, and stabilizing those tests. It selects the
repository's FVM, Puro, Melos, or script toolchain before running anything, adds
a focused test for uncovered code by default instead of asking, treats golden
updates as explicit reviewed actions, asks before changing uncovered production
behavior, and reports evidence and incomplete checks without implying full-suite
success. Integration and plugin/native platform testing are outside its scope.
