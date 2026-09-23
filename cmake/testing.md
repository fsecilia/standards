# Testing CMake Behavior

These standards apply when the project tests CMake code. This includes configure, generation, install, package discovery, and downstream use.

The repository-wide testing standards in [`../testing.md`](../testing.md) also apply.

## Test the Real Lifecycle

Use separate CMake processes when process boundaries are part of the behavior under test.

Install, reconfigure, package discovery, and downstream use often cross configure trees. Do not force these cases into one configure process just to make them look like ordinary unit tests.

When practical, prefer a lifecycle test that crosses the same boundary that real consumers cross.

## Fixtures and Orchestration

Keep fixture projects small and focused on the behavior under test.

Prefer separating fixture source trees from scripts that run configure, build, install, or downstream use when this makes the lifecycle easier to follow.

## Failure Boundaries

Apply the general failure rules at the CMake lifecycle boundary where the contract says the failure occurs.

Test a configure-time contract failure as a configure failure. Do not turn it into a later build-time assertion only because the later phase is easier to drive.

## Testing Installation

When a project exports CMake packages, test installed use separately from source-tree use when practical.

Installed-package tests use the same interface intended for downstream users. They do not reach back into the original build tree.

If package identity, target names, version requirements, or visibility are part of the contract, assert them directly.

## Version and Generator Coverage

Some CMake behavior changes across supported versions. When that matters to a test, run it at the relevant version boundaries.

Generators can also change behavior. Test the relevant generators when the contract depends on multi-config behavior, install layout, generated targets, or similar generator-specific behavior.

Passing with one version or generator does not prove behavior that is known to vary across the supported range.
