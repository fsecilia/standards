# C++ Source Structure

This document defines how C++ source, namespaces, includes, tests, and build boundaries are organized.

The repository-wide placement rules in [`../repository.md`](../repository.md) also apply.

## Source Layout

C++ source lives under the project's root namespace directory:

```text
src/<root_namespace>/
```

For a project whose root namespace is `axiom`:

```text
src/axiom/
```

Physical layout follows conceptual ownership, not whether a header is public or private.

Public and private headers live beside the code they belong to. Do not split the project into parallel `include/`, `src/`, and `tests/` trees.

For example:

```text
src/axiom/platform/
    window.hpp
    renderer.hpp
    platform.hpp

    sdl/
        window.hpp
        window.cpp
        window_test.cpp

    vulkan/
        instance.hpp
        instance_test.cpp
        renderer.hpp
        renderer_test.cpp

    vulkan_sdl/
        platform.hpp
        platform.cpp
        platform_test.cpp
```

Installing a header does not decide where it lives. The filesystem shows ownership; build metadata shows publication.

## Namespaces

Project C++ code lives under the project's root namespace.

Source directories may map to nested namespaces when the namespace is a useful domain boundary. For example:

```text
src/axiom/platform/vulkan_sdl/
```

may contain code in:

```cpp
namespace axiom::platform::vulkan_sdl {}
```

Directories and namespaces do not need to match one-to-one.

Add a nested namespace because it means something to the code, not simply because a directory exists.

## Includes

Use root-qualified quoted includes for project headers:

```cpp
#include "axiom/platform/renderer.hpp"
#include "axiom/platform/window.hpp"
```

This keeps the include stable when the including file moves elsewhere in the repository while following the usual distinction between project-owned headers and system or externally supplied headers.

Do not use relative traversal such as:

```cpp
#include "../window.hpp"
#include "../../common/status.hpp"
```

During a source-tree build, configure include search paths so the current project's source include root resolves before any installed copy of the same project. Do not let an older installed header shadow the source being built.

## Headers are Self-Contained

Every header must be self-contained.

This alone must be enough to compile the header's declarations:

```cpp
#include "axiom/platform/renderer.hpp"
```

A header must not rely on another project header having been included first.

A published header must not include a private header from the same project.

## Principal Headers

The principal header of an implementation or isolation-test translation unit is the header for the component that file implements or tests. Use the same basename unless a different mapping is clearer.

A `.cpp` or `_test.cpp` file beside its principal header may include that header directly:

```cpp
#include "window.hpp"
```

For example:

```text
window.hpp
window.cpp
window_test.cpp
```

Both `window.cpp` and `window_test.cpp` may use:

```cpp
#include "window.hpp"
```

This exception applies only to the principal header.

Use root-qualified quoted includes for other project headers.

Keep the principal header first in an implementation or isolation-test translation unit. This helps expose missing dependencies in that header.

## Include Ownership

Each source or test file includes the headers that declare what it uses, even when its principal header already includes them.

For example, if `renderer.cpp` uses a declaration from `window.hpp`, include `window.hpp` directly instead of relying on `renderer.hpp` to provide it.

This keeps each file's dependencies explicit. It also lets a header remove an include it no longer needs without breaking an implementation or test that used it indirectly.

The principal-header rule still applies: include the principal header first, then the other direct dependencies according to the project's formatting configuration.

## Tests

C++ isolation and ordinary regression tests live beside the production component they exercise.

Use the `_test.cpp` suffix:

```text
renderer.hpp
renderer.cpp
renderer_test.cpp
```

Higher-level integration and end-to-end tests follow the placement rules in [`../repository.md`](../repository.md).

Use the `_integration_test.cpp` suffix for integration tests:

```text
renderer_integration_test.cpp
```

Use a more specific suffix for another distinct test class only when the distinction is useful to the project.

## Build Structure

The repository-wide rules in [`../repository.md#build-structure`](../repository.md#build-structure) define build-file and target boundaries. They apply unchanged to C++ source subtrees.
