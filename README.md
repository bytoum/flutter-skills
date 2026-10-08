# Flutter Skills

Reusable agent skills for Flutter and Dart workflows. Each skill is a focused,
portable set of instructions stored in its own directory under `skills/`.

## Available skills

| Skill | Description |
| --- | --- |
| [`flutter-analyze`](skills/flutter-analyze/SKILL.md) | Diagnose and resolve Flutter and Dart analyzer findings while preserving intended behavior and the project's analysis policy. |
| [`flutter-testing`](skills/flutter-testing/SKILL.md) | Create, run, diagnose, review, and stabilize Flutter and Dart unit, widget, and golden tests; ask before changing uncovered production behavior. |
| [`flutter-unit-testing`](skills/flutter-unit-testing/SKILL.md) | Write, run, and fix Flutter and Dart unit tests for pure logic, async code, and state-management units with deterministic test doubles. |
| [`roll-dice`](skills/roll-dice/SKILL.md) | Generate random dice rolls with shell or PowerShell commands. |

## Install a skill

A skill is a directory containing a `SKILL.md` file. Copy the directory for the
skill you want to use into one of Codex's skill locations:

- `.codex/skills/` in a repository for a project-scoped skill.
- `~/.codex/skills/` for a user-scoped skill on macOS or Linux.
- `%USERPROFILE%\.codex\skills\` for a user-scoped skill on Windows.

For example, after cloning this repository, install `flutter-analyze` for your
user account on macOS or Linux:

```bash
mkdir -p ~/.codex/skills
cp -R skills/flutter-analyze ~/.codex/skills/
```

Or with PowerShell on Windows:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex\skills"
Copy-Item -Recurse skills\flutter-analyze "$env:USERPROFILE\.codex\skills\"
```

Review a skill's instructions before installing it. Restart Codex or begin a
new session if the newly copied skill is not discovered immediately.

## Use a skill

Codex can select a skill automatically from its name and description. You can
also invoke one explicitly by referencing its name with a `$` prefix:

```text
Use $flutter-analyze to run the analyzer and fix its findings.
```

### Flutter Testing example

Ask the skill to add a widget test and run it, while preserving the confirmation
gate for uncovered production behavior:

```text
Use $flutter-testing to add a widget test for LoginForm and run it. If the
behavior is uncovered, ask me which path to take before changing production code.
```

In a project that uses FVM, a focused test command might be:

```bash
fvm flutter test test/path/to/example_test.dart
```

Use the wrapper and test command configured by the project; this example is not
universal.

## Repository structure

```text
.
├── .changeset/                 Release notes and Changesets configuration
├── skills/
│   └── <skill-name>/
│       └── SKILL.md            Skill metadata and instructions
├── AGENTS.md                   Repository contribution guidance
└── package.json                Release-tool configuration
```

Skill directories may also contain supporting `references/`, `scripts/`, or
`assets/` when the workflow requires them.

## Contributing

Create each skill in a lowercase, hyphen-separated directory under `skills/`.
Its `SKILL.md` must begin with YAML frontmatter containing a quoted semantic
version, a name that matches the directory, and a trigger-oriented description:

```yaml
---
version: "1.0.0"
name: example-skill
description: Explain what the skill does and when it should be used.
---
```

Keep instructions direct, portable, and focused on one coherent capability.
Before submitting a change:

1. Validate the YAML frontmatter and confirm the name matches the directory.
2. Review commands and examples for placeholders, quoting, and platform assumptions.
3. Check the diff for unrelated changes or generated files.
4. Add a Changeset for a user-visible skill addition or behavior change:

   ```bash
   npm install
   npm run changeset
   ```

Documentation-only and repository-maintenance changes do not require a
Changeset unless they affect published skill behavior.

This repository does not currently have a functional automated test suite;
`npm test` is a placeholder that exits with an error.

## License

This project is licensed under the ISC license, as declared in `package.json`.
