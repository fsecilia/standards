# Standards

This repository defines shared engineering standards for a group of related projects. Each consuming project includes Standards as a Git submodule named `standards/` and pins the revision whose rules it follows.

The standards are grouped by the scope in which they apply:

- `repository.md` applies to every project.
- `testing.md` applies to projects that maintain automated tests.
- `cpp/` applies to projects that contain C++.
- `cmake/` applies to projects that contain CMake.

## Reading the Standards

Direct instructions such as "Use," "Do not," and "Keep" state an engineering rule rather than a suggestion.

`Prefer` identifies the normal engineering choice. Use another approach when a concrete project constraint or design requirement outweighs that default.

Each consuming project's `CONTRIBUTING.md` identifies the standards that apply and records any project-specific additions or exceptions. Document a deliberate exception when future contributors would otherwise reasonably apply the shared rule.

The shared `.clang-format` and `.clang-tidy` files define the mechanical baseline for C++ projects. Consuming projects normally copy and check in those files so their tools can use them directly. A project may deliberately change its copy, but the difference belongs in that project's documented exceptions rather than remaining unexplained configuration drift.

Unless a formatter, linter, compiler check, or automated test enforces a rule, enforcement happens during code review. These standards do not assume that every engineering rule has a dedicated automated checker.

## Adopting Changes

Changing this repository does not automatically change the rules for consuming projects. A project adopts a change when it updates the pinned `standards/` submodule revision and any checked-in copies of shared tool configuration that changed with it.
