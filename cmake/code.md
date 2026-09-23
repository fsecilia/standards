# CMake Coding Standards

These standards apply to CMake code maintained by the project. Project-specific rules may add stricter requirements.

## CMake Baseline

Require CMake 3.31.6 or newer. A project may require a newer version when it depends on newer CMake behavior.

## Targets and Directory Scope

Prefer target-scoped commands for properties of a target. Use `target_include_directories()`, `target_compile_definitions()`, `target_compile_options()`, `target_link_libraries()`, and similar target commands with explicit `PUBLIC`, `PRIVATE`, or `INTERFACE` scope where that scope is part of the command.

Do not use directory-scope commands such as `include_directories()`, `add_definitions()`, `add_compile_options()`, or `link_libraries()` when the setting belongs to a particular target and a target-scoped form can express it.

Use directory or project-wide state only for behavior that is genuinely shared at that scope. Keep every change to ambient CMake state deliberate, visible, and attributable to the code that owns the policy.

## Functions and Scope

Prefer `function()` when local scope is enough. Use `macro()` only when the implementation must affect caller scope.

Avoid unnecessary changes to caller or global state. When shared state is required, make its lifetime and ownership clear.

## Variables and Lists

Quote variable expansions unless list expansion is intentional and clear from the call. Make list behavior visible at the call site.

Do not rely on accidental splitting, empty arguments, or unclear scalar-to-list conversions when the distinction affects the command's semantics.

## Conditionals

CMake conditionals are a sharp edge. Take care when choosing to quote an expression because CMake gives unquoted condition arguments additional interpretation. A value can be treated as a boolean constant, variable name, keyword, or another part of the condition syntax instead of as the value the caller supplied. Quoting prevents this extra interpretation where the condition grammar accepts a value.

Do not turn this into a rule that every condition must use quoted expansion. Forms such as `if (variable)`, `DEFINED`, `TARGET`, `COMMAND`, `EXISTS`, and `IN_LIST` have their own operand rules. Use the form that expresses the intended test.

Write conditions so it is clear whether an argument is a variable name or a value.

For a boolean variable, test the variable directly:

```cmake
if (feature_enabled)
    ...
endif()
```

Use explicit, quoted expansion when comparing values:

```cmake
if ("${actual}" STREQUAL "${expected}")
    ...
endif()
```

Do not expand a value into an unquoted `if ()` argument when that argument is meant to remain a value:

```cmake
# Avoid: the expanded value may be interpreted again by if ().
if (${value})
    ...
endif()
```

Take extra care with `macro()` arguments. Macro arguments are textual substitutions, not normal CMake variables. An expression such as:

```cmake
if (argument)
```

tests a variable named `argument`; it does not test the macro argument. Expand the argument when its value is required:

```cmake
if ("${argument}")
    ...
endif()
```

Do not rely on `AND` or `OR` to short-circuit. CMake evaluates both sides of these operators. Split a condition into nested `if ()` statements when evaluating one part safely depends on another part being true.

## Naming

Name functions for their role rather than their implementation details when practical.

Use a project-specific prefix for public APIs and private names to reduce naming collisions: `prefix_perform_action()`

Use an `_` prefix for private names: `_prefix_perform_private_action()`

Public namespaced targets use `Project::Target`. When a project exposes one primary target, use the project name for both parts, such as `Axiom::Axiom`.

Give the concrete target the matching `EXPORT_NAME` and provide an in-tree alias so build-tree and installed-package consumers use the same public name:

```cmake
add_library(axiom ...)
set_target_properties(axiom PROPERTIES EXPORT_NAME Axiom)
add_library(Axiom::Axiom ALIAS axiom)
```

When exporting the target, use the matching namespace, such as `NAMESPACE Axiom::`.

## Comments

Comments explain purpose, constraints, invariants, or non-obvious CMake behavior instead of narrating statements whose meaning is already clear from the commands themselves.

Give a function a short summary when its purpose is not clear from its name. Write function summaries in the present tense with an implied "This function" subject: "Locates the requested package," not "Locate the requested package."

Describe what the function does, including its contract, constraints, and important effects. Put comments about how the implementation works in the function body. Mention implementation details at the function boundary only when callers need to know them.

Use block-level comments when several commands implement one non-obvious operation. Explain the approach once instead of commenting each line.

## Errors and Fallbacks

Fail when the requested contract cannot be preserved. Do not silently continue with behavior that only looks similar to the requested result.

A supported fallback must be intentional and documented. When practical, show the fallback in configure output so users can tell which path was selected.

Do not create placeholder targets, fake metadata, or weaker behavior just to let configuration continue.

## C++ Header Publication

For C++ targets, declare installed headers with CMake file sets instead of maintaining a separate source tree only for installation.

For a target owned by `src/axiom/CMakeLists.txt`, for example:

```cmake
target_sources(
    axiom
    PUBLIC
        FILE_SET HEADERS
        BASE_DIRS "${PROJECT_SOURCE_DIR}/src"
        FILES
            platform/window.hpp
            platform/renderer.hpp
            platform/platform.hpp
)
```

`FILES` paths are relative to the `CMakeLists.txt` that owns the target. `BASE_DIRS` identifies the source include root so the installed layout preserves root-qualified includes such as `#include "axiom/platform/window.hpp"`.

Publication is explicit build metadata, not a consequence of where a header happens to live.

## Wrappers and Compatibility

Prefer direct CMake commands over wrappers that only rename a command or property. A wrapper should add policy, validation, or a useful abstraction.

A wrapper normally preserves the scope and visibility of the native operation. If it changes those rules, make the different contract explicit. Consider target visibility, variable scope, directory scope, and package discovery.

If downstream projects can observe a CMake behavior, treat that behavior as part of the wrapper contract. This includes package discovery, target identity, target visibility, installation behavior, exported configuration, and important directory-scope effects.

When changing a wrapper, check its result and any changes it makes to the surrounding configure state.
