---
version: "1.0.0"
name: flutter-analyze
description: Diagnose and resolve Flutter and Dart analyzer findings. Use when asked to run flutter analyze, reduce analyzer errors or warnings, or make analyzer-based CI checks pass without changing intended behavior.
---

# Flutter Analyze

Use analyzer output as a bounded repair queue. Preserve runtime behavior and the
project's analysis policy unless the user explicitly requests broader changes.
Requests to fix "everything" do not authorize behavior changes, public API
changes, suppressions, or analysis-policy changes.

## Workflow

1. Identify the package root, `analysis_options.yaml`, and the exact command,
   working directory, and flags used by CI when CI is in scope.
2. Run that command. When output is large, retain the complete diagnostics with
   `flutter analyze --write=<temporary-file>` rather than relying on truncated
   terminal output. If adding `--write` changes the original command, treat the
   capture run as a diagnostic companion, not as final verification.
3. Group findings by diagnostic rule and root cause. Fix errors first, then
   warnings, then info-level findings, unless the user's goal or CI flags impose
   a different threshold. Re-run after fixes that may remove many downstream
   findings.
4. Prefer narrow, mechanical changes. Preserve public APIs and observable
   behavior unless a diagnostic cannot be resolved safely without changing
   them.
5. Re-run the original command with the original flags. Report the final result
   and any remaining diagnostics; do not claim the analyzer is clean from a
   partial run.

## Boundaries

- Edit project-owned source. Do not hand-edit generated files such as
  `*.g.dart`, `*.freezed.dart`, `build/`, package-cache content, or vendored
  code. Change the source or generator inputs and regenerate only when that work
  is in scope.
- Do not weaken `analysis_options.yaml`, add blanket ignores, or disable fatal
  warning or info flags merely to produce a zero exit code.
- Stop for direction when a fix requires a behavior change, public API change,
  dependency migration, or a policy choice between keeping and suppressing a
  diagnostic. Report the remaining analyzer or CI failure with that request.

## Common Mistakes

| Mistake | Correction |
| --- | --- |
| Treating every nonzero exit as the same failure | Inspect diagnostic severity and the command's fatal flags. |
| Fixing findings in terminal order | Group repeated rules and repair shared root causes first. |
| Reporting success after a narrower local command | Verify with the same command and working directory used by the requested workflow. |
