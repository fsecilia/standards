# Banner Comments

Prefer source files that do not need banner comments. Use banners only when they make meaningful groups within a file easier to see and splitting the file would reduce cohesion.

Treat banner strength as a hierarchy allocated from the bottom up. Start with the weakest form that is sufficient. Sibling sections at the same nesting level use the same banner strength.

When a section must be subdivided, use its current banner strength for the new child sections and promote that section and each enclosing banner by one level. Do not reserve stronger banners for possible future use.

If the available hierarchy is no longer sufficient, consider splitting the file.

For CMake, use these banner forms, from strongest to weakest:

```cmake
# #####################################################################################################################
# Section title
# #####################################################################################################################

# =====================================================================================================================
# Section title
# =====================================================================================================================

# ---------------------------------------------------------------------------------------------------------------------
# Section title
# ---------------------------------------------------------------------------------------------------------------------

#
# Section title
#
```

For C++, use the equivalent forms:

```cpp
// ####################################################################################################################
// Section title
// ####################################################################################################################

// ====================================================================================================================
// Section title
// ====================================================================================================================

// --------------------------------------------------------------------------------------------------------------------
// Section title
// --------------------------------------------------------------------------------------------------------------------

//
// Section title
//
```

The hierarchy should grow only when the structure of the file requires it. For example, a file may begin without banners:

```text
A

B
```

If `A` and `B` become distinct sections, introduce the weakest banner:

```text
weak A

weak B
```

If those sections later belong to a larger group, use the next-stronger banner for the group:

```text
medium Group
    weak A
    weak B
```

If `A` must then be split into child sections, promote the existing hierarchy and reuse the previous strength for the new children:

```text
strong Group
    medium A
        weak A1
        weak A2

    medium B
```

Banner strength reflects nesting depth, not the perceived importance of a section.
