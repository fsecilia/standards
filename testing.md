# Testing

This document defines testing rules that apply across languages and build systems. More specific standards may add requirements for a particular language, tool, or lifecycle boundary.

## Test the Contract

Test observable behavior at the smallest boundary that makes the claim meaningful. A useful test communicates which contract failed without requiring unrelated implementation details to diagnose the result.

Prefer focused tests. One tested condition per test is a useful target when the behavior separates cleanly; avoid monolithic tests whose unrelated assertions make failures difficult to localize.

Prefer assertions about externally visible contracts over assertions coupled to private implementation details. Tests may inspect internal state when that state is itself the contract under test, but convenience alone is not a reason to create that coupling.

## Isolation and Integration

Isolation tests exercise one component with its dependencies controlled or replaced. They show that the component satisfies its contract for selected inputs and collaborator behavior.

Integration and end-to-end tests exercise real compositions. They show that components are assembled correctly, satisfy one another's contracts, and cooperate across system boundaries.

The two scopes answer different questions. Neither replaces the other, and passing in one scope does not stand in for coverage in the other.

## Regression Coverage

Add a focused regression test for a behavior change when practical.

When fixing a defect, reproduce the failure at the narrowest useful boundary before changing the implementation when practical. Keep the regression test after the correction so the affected contract remains protected against recurrence.

## Failure Behavior

A test must fail clearly when required setup, dependencies, tools, or expected results are missing. Keep environmental or orchestration failures distinguishable from failures in the behavior under test.

Do not replace a failed test path with a weaker path without making that reduction visible. If a fallback or reduced-coverage mode is intentional, identify it in the test output so the resulting evidence cannot be mistaken for full coverage.

Check a failure at the boundary where the contract says it occurs. Do not move the assertion to a later phase only because the later phase is easier to drive or instrument.
