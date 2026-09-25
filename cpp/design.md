# C++ Design

This document defines the shared C++ programming model and the design choices we expect projects to make repeatedly.

The main goals are explicit dependencies, components that can be tested in isolation, strong compile-time composition, and an object graph that can also be tested as a complete system.

## Errors and Invariants

Assert internal invariants and states that indicate a programming error. Do not use assertions for expected runtime failures.

Error-carrying results must not be silently discardable. Mark project-owned error-carrying types `[[nodiscard]]`. When the result type cannot carry that attribute, mark the returning function instead.

These standards do not choose one error-transport mechanism for every project. Use exceptions, std::unexpected, explicit error values, or another mechanism according to the contract being expressed.

## Value Semantics

Prefer value semantics when they fit the component. Values make ownership, lifetime, and composition easier to reason about and work naturally with the static composition model used here.

Custom move constructors and move assignment operators are `noexcept`. Do not write a throwing custom move only to keep a type movable.

Implicit and defaulted moves may throw when a member's move throws. Do not assume that generic or injected types move without throwing. Require nothrow movement only at a boundary that needs it.

## Testing Model

The repository-wide testing rules in [`../testing.md`](../testing.md) apply to C++ tests.

Isolation tests exercise one component without exercising its real behavioral dependencies. Replace those dependencies with test doubles so the component can be driven directly and its observable behavior checked precisely.

Integration tests exercise the real composition. They verify that the independently tested components are assembled correctly and satisfy one another's contracts.

A behavioral dependency that cannot be replaced prevents the consumer from being tested in isolation. This is why explicit dependencies, substitution seams, and composable behavior are recurring design choices in these projects. Create those seams in the production design rather than adding test-only access paths.

Avoid test-only constructors. Do not use `friend` to give a test access to a type's internals.

## Component Boundaries

Split a component when doing so clearly improves responsibility, reuse, testability, substitution, or composition.

Prefer one coherent job per type.

If part of a type needs to be reused on its own, extract it.

If a type is hard to test because useful seams are hidden inside it, introducing another component is a valid design improvement. That can be true even when the new component has only one production use.

Testability is part of the design, not something added afterward for tests.

## Abstraction Cost

Do not add an abstraction only for symmetry or possible future extensibility.

Add a seam when it improves responsibility, reuse, testability, substitution, or composition. A parameterized type can still be useful with only one production implementation if its boundary makes the behavior easier to understand and test on its own.

The goal is not to minimize the number of types. The goal is a graph of components whose responsibilities and contracts can be understood and proven independently.

## Explicit Dependencies

Use dependency injection; consumers receive their behavioral dependencies instead of constructing or locating them. Keep those dependencies visible and replaceable at the consumer boundary.

Inject dependency types through template parameters and dependency instances through construction. When a dependency can be represented directly in the object graph, do not hide it behind globals, service locators, static state, or implicit lookup.

Do not pass the construction details of a dependency through its consumer. Construct the dependency first, then give the completed dependency to the consumer.

## Static Composition

When a dependency's concrete type does not need to vary at runtime, parameterize the consumer on that dependency type:

```cpp
template <typename Clock, typename Formatter>
class Reporter {
public:
    Reporter(Clock clock, Formatter formatter)
        : clock_(std::move(clock)),
          formatter_(std::move(formatter)) {}

private:
    Clock clock_;
    Formatter formatter_;
};
```

Prefer static composition for substitution seams rather than adding runtime polymorphism.

Use runtime polymorphism when runtime heterogeneity or runtime substitution is part of the program's actual behavior. Existing runtime-polymorphic interfaces do not need to become templates merely for consistency.

Do not add runtime polymorphism only to provide a testing seam when static composition expresses the dependency cleanly. Do not add compile-time polymorphism if existing runtime polymorphism already provides a testing seam.

### Cost of Compile-Time Composition

Compile-time composition has costs. A template definition that callers instantiate is normally visible at the point of instantiation. This can move more implementation into headers, increase rebuild work, and produce longer diagnostics.

That does not make every template header public. Header publication is still an explicit build decision. Private template headers may remain private when only project code needs them.

A templated consumer should depend on the contract of its template parameter, not on the concrete production type that normally satisfies it. Do not include the production dependency merely to obtain declarations that belong to that dependency's contract. Associated types, constants, and operations may instead be supplied by the dependency type so test doubles can provide the same contract.

Do not duplicate genuinely shared domain vocabulary merely to avoid an include. If an enum, constant, or type has meaning outside one dependency contract, give it an independent owner and include that declaration normally.

Keep structural contracts narrow. If a dependency requires many associated types, constants, or operations, reconsider the boundary. The dependency may have too many responsibilities, or some of its vocabulary may deserve an independent owner.

Explicit instantiation can move a template definition out of widely included headers when the supported specializations are intentionally closed. Do not use it when consumers are expected to instantiate the template with arbitrary types.

Keep template surfaces focused. Move work that does not depend on template parameters out of the template when doing so keeps the design clear. If compile-time cost becomes significant, treat it as an engineering constraint rather than ignoring it for the sake of uniformity.

## Behavior as Types

Prefer small composable types over namespace-scope functions for application behavior.

When an operation is useful as an independently composable or replaceable piece of behavior, prefer a functional type:

```cpp
struct FormatDiagnostic {
    auto operator()(Diagnostic const& diagnostic) const -> std::string;
};
```

rather than:

```cpp
auto formatDiagnostic(Diagnostic const& diagnostic) -> std::string;
```

Giving behavior a type lets it participate directly in dependency injection, be replaced in isolation tests, gain state or policy without changing the consumer's composition model, and compose naturally into larger object graphs.

Free functions are still a good fit when the operation is shaped by the language, required by an external API, or is a genuinely local helper with no useful substitution boundary.

Examples include operators, customization points, C ABI boundaries, and tightly scoped implementation helpers.

For ordinary application behavior, however, prefer a composable type.

Dependency names follow [`style.md`](style.md#names-describe-meaning). Keep the role expected by the consumer separate from the concrete mechanism that satisfies it.

## Composition

Prefer simple value composition.

Keep composition hierarchical. A component normally depends on a small number of direct collaborators rather than the transitive dependencies beneath them. Compose lower-level behavior first, then inject the resulting component.

A growing dependency parameter list is a reason to reconsider the component's responsibility or composition boundary. Orchestrators may legitimately coordinate more direct collaborators, but ordinary components should not flatten the dependency graph into one long parameter list.

Inject already-composed dependencies instead of making each consumer reconstruct them from lower-level pieces. The same rule applies to types and instances.

For type-level composition, build policy and component types at a composition boundary, then inject the resulting type:

```cpp
using CursorFilter = FilterPipeline<NoiseFilter, SmoothingFilter>;
using DesktopPointerController = PointerController<CursorFilter>;
```

Do not make `PointerController` take `NoiseFilter` and `SmoothingFilter` as separate template parameters only so it can rebuild `FilterPipeline` internally.

At the instance level, pass a constructed dependency to its consumer directly instead of passing the dependency's constructor arguments through the consumer. Construct object graphs in composition roots or parameterized factories.

A dependency-injection library may build the object graph at the composition boundary, but application types remain unaware of the container. The same dependency boundaries should work with direct construction in tests.

Keep the dependency graph explicit. Make it understandable from constructors, template parameters, factories, and composition code rather than hidden runtime machinery.

Prefer the Law of Demeter. Ask direct collaborators for behavior instead of reaching through them to find unrelated objects. A chain such as `controller.window().display().refresh()` is a warning sign that the consumer knows too much about another object's structure.

Apply the same idea to types. Avoid making a consumer navigate chains of nested dependency types to find a type it needs. Make that type a direct dependency or compose it at the appropriate policy boundary.

Isolation tests compose a component with test doubles. Integration tests compose the corresponding real graph. Both should use the same production dependency boundaries.

Name test doubles according to [`style.md`](style.md#testing).
