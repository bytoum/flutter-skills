# AGENTS.md

## Repository overview

This repository contains reusable Flutter-related agent skills. Each skill lives
in its own directory under `skills/` and is defined by a `SKILL.md` file.

The repository currently uses npm only for Changesets-based release management;
there is no application build or automated test suite.

## Repository structure

- `skills/<skill-name>/SKILL.md`: instructions and metadata for one skill.
- `.changeset/`: Changesets configuration and release notes.
- `package.json`: repository metadata and release-tool dependencies.

Keep skill-specific files inside the corresponding `skills/<skill-name>/`
directory. Do not add generated output, dependency directories, or unrelated
application code to the repository.

## Working on skills

When adding or updating a skill:

1. Use a lowercase, hyphen-separated directory name.
2. Put the primary instructions in `skills/<skill-name>/SKILL.md`.
3. Begin `SKILL.md` with YAML frontmatter containing at least:

   ```yaml
   ---
   name: example-skill
   description: Explain what the skill does and when it should be used.
   ---
   ```

4. Keep `name` identical to the skill directory name.
5. Quote the semantic version and increment it when publishing a changed skill.
6. Make the description trigger-oriented: state both the capability and the
   user requests or situations that should activate it.
7. Write instructions as direct, executable guidance. Include commands or
   examples only when they materially help the agent perform the task.
8. Prefer portable instructions. If commands differ by platform, label each
   platform explicitly.

Keep each skill focused on one coherent capability. Avoid duplicating general
agent behavior that does not belong to the skill.

## Changesets and versioning

For a user-visible skill addition or behavior change, add a Changeset with:

```bash
npm run changeset
```

Describe the change from the user's perspective and select the appropriate
semantic version bump. Documentation-only or repository-maintenance changes may
omit a Changeset unless the change affects a published skill.

The Changesets base branch is `main`, and packages are configured as restricted.

## Validation

Before finishing a change:

- Confirm every changed `SKILL.md` has valid YAML frontmatter.
- Confirm its `name` matches its parent directory.
- Check examples and shell commands for placeholders, quoting, and platform
  assumptions.
- Review the diff for unrelated edits and generated files.
- Add a Changeset when the published behavior or version should change.

There is currently no functional test command. `npm test` is a placeholder that
always exits with an error, so do not treat it as a validation step or report it
as a passing test. If automated validation is added later, update this file and
`package.json` together.

## Change discipline

- Preserve existing content and conventions unless the task requires changing
  them.
- Keep edits narrowly scoped to the requested skill or repository tooling.
- Do not commit `node_modules/` or `dist/`.
- Do not change release configuration, package metadata, or unrelated skills as
  part of a routine skill edit.
