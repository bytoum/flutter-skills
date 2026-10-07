---
version: "1.0.0"
name: flutter-test-coverage
description: Use when measuring, interpreting, enforcing, or improving Flutter test coverage, including LCOV reports, uncovered lines, changed-code coverage, coverage regressions, or CI threshold failures. Not for general test authoring, static analysis, or treating a percentage alone as proof of test quality.
---

# Flutter Test Coverage

Use coverage as evidence about which code executed, then connect uncovered code
to behavior and risk. Preserve the project's existing coverage scope, filters,
and CI policy so comparisons use the same denominator.

## Workflow

### 1. Discover the coverage contract

Find the Flutter package or workspace root and inspect `pubspec.yaml`, test
directories, package scripts, CI workflows, and existing coverage configuration.
Resolve:

- the repository-selected toolchain (for example FVM, Puro, Melos, or a script);
- the command and working directory that produce coverage;
- included packages, test tags, shards, and platform assumptions;
- exclusions, path rewrites, merge steps, and threshold or changed-code rules;
- the report format and artifact consumed by CI.

Before invoking a version manager, check without launching Flutter that its
pinned SDK is installed. Ask before allowing a wrapper to download an SDK. Use
the global Flutter SDK only when the repository selects no wrapper.

### 2. Collect fresh evidence

Run the repository command. If none exists, confirm supported flags with the
selected SDK's `flutter test --help`, then use the smallest command that covers
the requested scope; the ordinary single-package starting point is:

```bash
flutter test --coverage
```

Record the exact command, working directory, SDK, selected tests, and output
path. Treat the report as valid only when the coverage-producing test run
completes successfully. Distinguish a focused run from package-wide or
workspace-wide coverage.

For a monorepo, keep package reports separate unless the repository already has
a merge convention. Combining LCOV files without path normalization can merge
different files or count the same source twice.

### 3. Interpret the report

Prefer the repository's report or summary command. If no viewer is configured,
inspect the LCOV file directly or use an already-installed LCOV-compatible
tool. Do not add a dependency merely to render a report without user approval.

For every reported percentage, state its metric and scope. Line, branch, and
function coverage are different denominators. Check source-file records and
hit counts before concluding that a file is uncovered; generated files,
re-export libraries, and path mismatches can distort summaries.

When comparing runs, hold the command, filters, SDK, tests, and source revision
constant. If they differ, describe the comparison as non-equivalent rather
than a regression or improvement.

### 4. Turn gaps into test targets

Prioritize uncovered behavior by failure cost, complexity, recent change, and
boundary risk. Map candidate lines back to observable behavior, then select the
smallest test layer that can prove it. Use `flutter-testing` when writing,
diagnosing, reviewing, or stabilizing unit, widget, or golden tests.

Exercise meaningful outcomes rather than adding calls or weak assertions only
to increase a percentage. Cover error paths and state transitions when they
carry risk; leave unreachable or generated code governed by the repository's
documented exclusion policy.

### 5. Handle gates and regressions

Reproduce a CI failure with the same command, report transformation, baseline,
rounding, and threshold. Identify whether the failure comes from changed
behavior, changed scope, a stale artifact, path normalization, or genuinely
uncovered code.

Keep the configured gate intact while adding meaningful coverage. Changing a
threshold, exclusion, baseline, or CI policy is a separate policy decision and
requires explicit user authorization.

### 6. Verify and report

After test changes, rerun the narrowest affected tests and then the exact
coverage command used for the comparison. Report:

- toolchain, working directory, and exact commands;
- test and source scope represented by the artifact;
- metric totals and material file-level gaps;
- whether the requested gate passed;
- behavior newly covered, rather than only percentage movement;
- incomplete verification, unavailable tooling, or non-equivalent baselines.

## Decision Guide

| Situation | Default |
| --- | --- |
| No repository coverage command exists | Confirm the selected SDK's supported flags, then start with `flutter test --coverage`. |
| Coverage percentage is high but a risky path is uncovered | Test the risky behavior; do not optimize the aggregate number. |
| Generated code dominates uncovered lines | Apply only the repository's established exclusion policy, or ask before changing it. |
| CI and local percentages differ | Compare commands, SDKs, selected tests, path rewriting, exclusions, and rounding. |
| A monorepo produces several LCOV files | Report per package unless an established merge step defines a shared denominator. |
| User asks for “100% coverage” | Clarify scope and metric, then expose remaining risk and maintenance tradeoffs. |

## Common Mistakes

| Mistake | Correction |
| --- | --- |
| Reporting a percentage without its metric or scope | Name the metric, selected tests, packages, and exclusions. |
| Trusting an LCOV file after failed tests | Regenerate it from a successful coverage run. |
| Comparing artifacts created by different commands | Reproduce both with an equivalent command and denominator. |
| Adding superficial assertions to cover lines | Test observable behavior and meaningful failure modes. |
| Lowering a gate to make CI green | Preserve policy and ask before changing thresholds or exclusions. |
