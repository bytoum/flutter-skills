---
"flutter-skills": patch
---

Change `flutter-unit-testing` so a failing test is never skipped or disabled.
When a failure can't be fixed in the test itself, or tests can't compile or run,
the agent now stops and reports the failing test, the reason, and options to
resolve it, then waits for the user's choice.
