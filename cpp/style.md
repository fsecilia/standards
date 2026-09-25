# C++ Style

This document defines shared C++ naming and source-level conventions.

Where this document is silent, follow the naming rules below and the nearby code.

## Language and Toolchain

Use C++26 mode for project-owned C++. Code must compile with GCC 14.2 and Clang 17.0 unless a project documents a newer compiler requirement.

Compiler versions are build compatibility floors. Development tools are separate dependencies and may require newer LLVM versions than the supported compiler. A user who only builds the project does not need the formatting or linting tools.

## Mechanical Formatting

The `.clang-format` and `.clang-tidy` files in Standards define the shared mechanical baseline. Consuming projects normally copy and check in those files so editors, builds, and other tooling can use them directly.

A project may deliberately change its local configuration. Document that difference as a project exception instead of duplicating the mechanical rule in prose.

### Clang Tools Versions

Use clang-format no older than 17.0.6. However, do not use clang-format 22; it has a bug in `AllowShortFunctionsOnASingleLine: InlineOnly` that affects our codebases. The minimum tested version of clang-tidy is 19.1.0.

Choose a development-tool version only after verifying that it accepts the shared configuration and preserves the intended mechanical rules.

## Naming

Files and directories use `snake_case`, with `.hpp` and `.cpp` suffixes for C++ files. Types, type aliases, concepts, and enum values use `PascalCase`.

Functions, variables, parameters, and constants use `camelCase`; namespaces use `snake_case`; and macros use `UPPER_SNAKE_CASE`.

Constants use ordinary value names:

```cpp
constexpr auto maxPlayers = 16;
```

Do not decorate a name only to show that the value is constant.

Private mutable instance data members use a trailing underscore.

Public struct members use ordinary value naming:

```cpp
struct WindowDesc {
    int width;
    int height;
    bool resizable;
};
```

Treat acronyms as words:

```cpp
GpuDevice gpuDevice;
HttpClient httpClient;
XmlParser xmlParser;
```

## Names Describe Meaning

Concrete types are named for what they are. Values are named for the role they play in the current context.

Those names are often the same:

```cpp
Renderer renderer;
```

They do not have to be:

```cpp
LowerBoundBisector inverter;
```

`LowerBoundBisector` describes the mechanism. `inverter` describes the role of that instance.

Dependency type parameters follow the same rule and describe the role required by the consumer:

```cpp
template <typename Inverter>
struct Consumer {
    Inverter inverter;
};
```

The concrete dependency can then describe its actual mechanism:

```cpp
using LowerBoundBisectionConsumer = Consumer<LowerBoundBisector>;
```

Prefer names that describe meaning rather than implementation machinery.

Do not use "foo", "bar", or other placeholder names for types or instances, even in examples.

## Construction

Prefer braces when constructing objects directly:

```cpp
Renderer renderer{device, queue};
auto config = Config{width, height};
```

Use parentheses when braces would select different semantics or cannot express the intended construction. This most often matters for types with `std::initializer_list` constructors. For example, use `std::vector<int> values(8, -1);` when the intent is eight copies of `-1`; braces would instead construct a two-element vector. Ordinary function calls continue to use parentheses.

## Local Variables

Use `auto` for local variables unless it cannot express the intended declaration. Write `const` to the right of the type it qualifies. When a local value is not intended to change, prefer `auto const`.

Preserve reference semantics explicitly. Bare `auto` creates a value and drops references and top-level `const`; use `auto&` or `auto const&` when the local is intended to refer to the original object. Use `auto&&` only when its reference-collapsing behavior is intentional. The same qualifier rule composes with pointers: `auto const*` points to a const value, while `auto* const` is a const pointer.

For example:

```cpp
auto const count = values.size();
auto const& current = values.front();
auto& destination = outputs.back();
```

Explicit types are necessary for instances that are deliberately uninitialized until a later conditional assignment or used as an out parameter. For example:

```cpp
int exponent;
auto const fraction = std::frexp(value, &exponent);
```

## Function Declarations

Use concrete trailing return types for functions:

```cpp
auto size() const -> std::size_t;
auto render(Frame const& frame) -> void;
```

Do not use a deduced function return type only to avoid spelling the return type. Deduced return types are useful in some cases, but avoid them unless they are necessary.

Use another return-type form only when required by external tooling or language integration, such as declarations processed by Qt MOC or exposed to QML.

Use `[[nodiscard]]` when accidentally discarding a result would lose a resource or failure information. Do not apply it broadly to ordinary return values.

## Constexpr

Prefer `constexpr` for functions that can naturally support constant evaluation. Do not restructure an interface or complicate an implementation solely to make a function `constexpr`.

In particular, use `constexpr` freely for small value-type operations, accessors, constructors, operators, and header-defined utilities when their implementation permits it.

## Comments

Comments explain purpose, constraints, invariants, or non-obvious behavior instead of narrating code whose meaning is already clear.

Give a type or function a short summary when its purpose is not clear from its name. Write function summaries in the present tense with an implied "This function" subject: "Allocates memory for the requested elements," not "Allocate memory for the requested elements."

Describe what a function does, including its contract, constraints, and important effects. Put comments about how the implementation works in the function body. Mention implementation details at the function boundary only when callers need to know them.

Describe what a type represents or is responsible for rather than listing its members.

Use block-level comments when several statements implement one non-obvious operation. Explain the approach once instead of commenting each line.

Use `//` for ordinary comments, including comments that span several lines. Reserve `/* ... */` for comments embedded in C++ syntax where a line comment would not fit naturally, such as an omitted parameter name or a short argument annotation.

Use `///` for Doxygen documentation comments. Use backslash commands such as `\param` and `\returns` rather than the equivalent `@` forms. Do not use `/** ... */` or `/*! ... */` for ordinary project documentation comments.

For example:

```cpp
// Preserve the previous value until every validation step succeeds.
// This keeps a failed update from changing visible state.
auto const candidate = parseConfig(input);

auto setCallback(Callback callback, int /*priority*/) -> void;
auto result = parse(input, /*allowTrailing=*/false);

/// Opens the requested asset.
///
/// \param path Path to the asset.
/// \returns The loaded asset.
auto openAsset(Path const& path) -> Asset;
```

## Headers

Use `#pragma once` in project-owned headers.

Header self-containment and include ownership are defined in [`structure.md`](structure.md#headers-are-self-contained).

## Interfaces and Implementations

An abstract runtime interface uses the natural name of the role it represents:

```cpp
class Renderer {
public:
    virtual ~Renderer() = default;

    virtual auto render(Frame const& frame) -> void = 0;
};
```

Do not add `I`, `Interface`, or `Abstract` just because the type is an interface.

Concrete implementations add the qualifier that distinguishes them:

```cpp
VulkanRenderer
SoftwareRenderer
RemoteRenderer
```

If there is truly no useful qualifier and the implementation needs a separate name, an `-Impl` suffix may be used locally as an escape hatch. It is not a general naming convention.

## Functional Types

A type that represents one focused operation may be named directly for that operation:

```cpp
// GOOD
FormatDiagnostic formatDiagnostic;
ValidateMesh validateMesh;
LoadTexture loadTexture;
```

Do not invent a noun-form type name just so the instance can use the natural verb name:
```cpp
// BAD
MeshValidator validateMesh;
```

When the contextual role differs from the operation itself, name the instance for that role:

```cpp
// GOOD
LowerBoundBisector inverter;
```

## Concepts

Concepts use natural names for the category or requirement they express:

```cpp
Renderable
Contiguous
Arithmetic
```

Prefer an adjective when the category has a natural adjective. When a concept structurally models an existing noun-named abstraction, a `-Like` suffix is appropriate:

```cpp
Renderer
RendererLike
```

## Template Parameters

Name template parameters according to what they represent:

```cpp
template <typename Allocator, std::size_t capacity>
class Container;
```

When a parameter is redeclared inside the type body, add its language category to avoid the name collision:

```cpp
template <typename AllocatorType, std::size_t capacityValue>
class Container {
public:
    using Allocator = AllocatorType;
    static constexpr auto capacity = capacityValue;
};
```

Use `Type` for type parameters and `Value` for non-type template parameters when the names would otherwise collide. Do not add these suffixes when there is no collision.

## Testing

Generally, test doubles are named like regular types and instances. There is no need to draw attention to the fact that an instance is a test double when it is the only object serving that role in the test.

Use a test-double category in the name only when the distinction itself matters or avoids a collision.

### Mocking with GMock

When GMock is used with a value-semantic dependency, keep the value-semantic test double named normally and delegate its behavior to a separate mock object. In this pattern, prefix the GMock type with `Mock` and the mock instance with `mock`. This keeps the testing machinery distinct from the dependency that the production component sees.

Give any mock type wrapped in a GMock decorator such as `StrictMock` a defaulted virtual destructor. GMock decorators derive from the mock type, and GoogleTest requires a virtual destructor for reliable decorator behavior.

```cpp
struct MockGenerator {
    virtual ~MockGenerator() = default;

    MOCK_METHOD(void, set, (int), (noexcept));
    MOCK_METHOD(int, call, (), (const, noexcept));
};

struct Generator {
    MockGenerator* mock = nullptr;

    auto set(int value) noexcept -> void { mock->set(value); }
    auto operator()() const noexcept -> int { return mock->call(); }
};

StrictMock<MockGenerator> mockGenerator;
Generator generator{&mockGenerator};
```
