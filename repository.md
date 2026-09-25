# Repository Structure

This document defines the common repository layout and conceptual ownership model shared across projects.

## Conceptual Ownership

Use directory structure to communicate which component conceptually owns a file. Keep files that evolve together structurally close unless another boundary provides a stronger reason to separate them.

Do not create parallel directory trees merely to separate public files, tests, build metadata, or similar concerns when those files belong to the same component.

Language-specific standards may define more detailed source-layout rules for their own files and tooling.

## Build Structure

Directory structure communicates conceptual ownership. Build files communicate meaningful build and target boundaries, and those two structures do not need to correspond one-to-one.

Do not add a build file merely because a directory exists. A subtree may own its own build file when it forms an independently useful library or another meaningful target.

Keep ordinary subdirectories in the parent target unless they need their own target.

## Tests

The behavioral testing rules in [`testing.md`](testing.md) apply across projects. This section defines repository placement rather than test behavior.

Place a test at the lowest conceptual boundary that contains everything it exercises.

Place a component test with that component. Place an integration or end-to-end test at the lowest boundary containing the complete composition it exercises. This does not require every directory to contain tests, and it does not require every test to own a separate build target.

Do not create a mirrored test tree merely to keep tests physically separate from the production code they exercise.

## External Projects

Reusable external projects that participate in the build live under:

```text
external/
```

When a dependency is maintained as an independent project, keep its repository history independent and place its working copy under `external/`. This allows a self-contained build workspace without combining unrelated repository histories.

`external/` is for genuine external source projects used by the build. Do not use it as a catch-all for copied third-party files or unrelated foreign material.

## Shared Standards

Shared engineering standards live in the repository's:

```text
standards/
```

submodule.

`standards/` is repository policy rather than a build dependency, so it does not belong under `external/`.

## Revision Control

Use [Conventional Commits](https://www.conventionalcommits.org/).

Keep the subject to 50 characters when practical. Wrap commit message bodies at 80 characters.

## Prose and Comments

When writing documentation and comments, aim for a Flesch-Kincaid grade level of about 10–12.
