# Visual Basic Language Design Meeting
December 6th, 2017

## Agenda
* Proposal [#25](https://github.com/dotnet/vblang/issues/25) - Range `1 To 10 Step 2` Expressions
    * _Related: [#180](https://github.com/dotnet/vblang/issues/180) - For-loop should use larger type to avoid overflow exception_
* Proposal [#215](https://github.com/dotnet/vblang/issues/215) - Allow Attributes on Generic Type Parameters
* Proposal [#170](https://github.com/dotnet/vblang/issues/170) - New Tie-Breaker Rule for Resolving Ambiguity Between Overloads Which Differ By Refness of Arguments
* Proposal [#218](https://github.com/dotnet/vblang/issues/218) - Expression tree rewrite for string comparison in VB should product an operator invocation, not a method call to Operators.CompareString
    * _Related: [#193](https://github.com/dotnet/vblang/issues/193) - Option Compare Ordinal (and OrdinalIgnoreCase?)_
* Proposal [#48](https://github.com/dotnet/vblang/issues/48) - Multiple `For` or `For Each` Control Variables Per Statement
    * _Related: [#104](https://github.com/dotnet/vblang/issues/104) - Extend `For Each` Statement with Query Comprehensions_
* Proposal [#186](https://github.com/dotnet/vblang/issues/48) - `Exit For j` Statement to Break Out of Nested `For` and `For Each` Loops
* Proposal [#102](https://github.com/dotnet/vblang/issues/102) - Support Top-Level Statements in a Single Entry-Point File
* Scenario [#219](https://github.com/dotnet/vblang/issues/219) - Implementing INotifyPropertyChanged is Tedious
    * Proposal [#107](https://github.com/dotnet/vblang/issues/107) - Replaceable Members
    * Proposal [#198](https://github.com/dotnet/vblang/issues/198) - Bindable Classes and Properties
    * Proposal [#107](https://github.com/dotnet/vblang/issues/194) - `WithPropertyEvents` Modifier to Enable Easy INotifyPropertyChanged Implementations

## Proposal [#25](https://github.com/dotnet/vblang/issues/25) - Range `1 To 10 Step 2` Expressions

_Related: [#180](https://github.com/dotnet/vblang/issues/180) - For-loop should use larger type to avoid overflow exception_

This proposal was envisioned specifically for VB and from the `For` loop down, so to speak. Independently, [the C# Range proposal](https://github.com/dotnet/roslyn/blob/features/range/docs/features/range.md) was envisioned for C# and designed from `Span<T>`/slices up. After this morning's C# LDM it's clear that we'll need to reconcile the two designs to make sure both languages can share `Range` types and inter-operate nicely. As this is likely our last LDM in either language for the rest of 2017 due to holiday/vacations, we will review the broader VB design in the new year. 

**Decisions**
* **Speclet needed**.

## Proposal [#215](https://github.com/dotnet/vblang/issues/215) - Allow Attributes on Generic Type Parameters

**Decisions**
* **Approved-in-Principle**. This isn't on our backlog but provide it was of high quality we'd likely accept this as a community PR.

## Proposal [#170](https://github.com/dotnet/vblang/issues/170) - New Tie-Breaker Rule for Resolving Ambiguity Between Overloads Which Differ By Refness of Arguments

**Decisions**
* **Approved-in-Principle**. We will likely need to do this to protect VB customers from breaks when API authors retro-fit `in` (readonly reference) parameter overloads on to existing methods.

## Proposal [#218](https://github.com/dotnet/vblang/issues/218) - Expression tree rewrite for string comparison in VB should product an operator invocation, not a method call to Operators.CompareString

_Related: [#193](https://github.com/dotnet/vblang/issues/193) - Option Compare Ordinal (and OrdinalIgnoreCase?)_

Eh, back compat concerns, trees, operators with methods, new VB runtime, fallbacks, LINQ providers should fix their stuff, CInt doesn't support binary literals or digit group separators, late-binding won't support the tie-breaker discussed in #170, needa solution.

**Decisions**
* **Speclet needed**.

## Proposal [#48](https://github.com/dotnet/vblang/issues/48) - Multiple `For` or `For Each` Control Variables Per Statement

_Related: [#104](https://github.com/dotnet/vblang/issues/104) - Extend `For Each` Statement with Query Comprehensions_

VB already has a strong precedent for multiple consecutive statements/constructs of the same kind being combinable into a comma-separated list: `Imports`, `Dim`, `Next`, `From`, `Let`, `Case` (required in this case).

**Decisions**
* **Approved-in-Principle**. None of us could think of a good reason why this doesn't already work. Allowing the `For Each` case is virtually required by #104.

## Proposal [#186](https://github.com/dotnet/vblang/issues/48) - `Exit For j` Statement to Break Out of Nested `For` and `For Each` Loops

The real motivator for this is that `Goto` is a code-smell and people want a less smelly solution to this problem.

**Decisions**
* **Approved-in-Principle, also do this for `Continue For` statement**.

### Discussion

We use `Goto` all over the scanners and parsers in both languages and we don't feel bad about those uses because `Goto` seems the most appropriate way of expressing that control flow. And because you're going from a likely more descriptive label name to a likely brief control variable name this could actually be a less readable construct in many cases. But, VB already has a bit of a precedent here insofar as:

1. The programmer can specify the specific block they want to exit at any time, e.g. `Exit Do`, `Exit While` to jump out of an outer loop or even out of an outer block via `Exit Select`.
2. The `Next` statement already lets you specify multiple control variables to close multiple loops in one go.

**Would it be better to have labeled loops like in Java?**
 
Maybe, but doing the proposal doesn't preclude us doing what Java does later. And if your control variables are well named, e.g. `row` and `column` then requiring the programmer to label the entire loop might very well be redundant and people would complain about that.

**Should we require or allow a comma-separated list of control variables like `Next`, e.g. `Continue For x, y`?**
Seems reasonable

**No**. In fact, for the `Continue` case that's not at all what's happening. One isn't continuing the inner loop _then_ continuing the outer one; one is simply skipping to the next iteration of the outer loop.

## Proposal [#102](https://github.com/dotnet/vblang/issues/102) - Support Top-Level Statements in a Single Entry-Point File

Last time we discussed this proposal we decided to take a wait-and-see approach with https://try.dot.net/. Today they use the scripting dialect of C# by virtue of using the Scripting API but based on discussions with them it would be beneficial to have the simplicity of top-level statements in the language(s) proper. For one, it means that all of the documentation sites/pages that are making use of trydotnet are technically teaching a slightly different version of the language with slightly different semantics. Let's try to reconcile the standard and scripting dialects in the new year.

**Decisions**
* **Deferred to January 2018**. 

## Scenario [#219](https://github.com/dotnet/vblang/issues/219) - Implementing INotifyPropertyChanged is Tedious

In our last deign meeting (two weeks ago) we looked at these three proposals on a spectrum from most general (#107) to most specific (#198), with #194 falling somewhere in between. The decision in that meeting was to take another look at our implementation concerns with [Source Generators](https://github.com/dotnet/roslyn/blob/features/range/docs/features/generators.md) to see if we could scope it down in such a way that it was achievable in the VB 16/C# 8 timeframe. Nope. While we bounced ideas around on how to constrain the behavior of generators to minimize the complexity and performance concerns it still feels like a _long_ lead to get it right both in design and implementation and especially iteration with the community for feedback. We absolutely cannot rush full meta-programming solution and our plate is pretty full already with HUGE ticket items for the next major version that generators are extremely unlikely to fit in either developer-time or calendar-time.

Proposal #194 seems to strike a good balance. We like that it could support a few more scenarios than just `INotifyPropertyChanged`, e.g. logging, "Freezable" objects.

### Proposal [#107](https://github.com/dotnet/vblang/issues/107) - Replaceable Members

**Decisions**
* **Deferred to next release**.

### Proposal [#198](https://github.com/dotnet/vblang/issues/198) - Bindable Classes and Properties

**Decisions**
* **Rejected**.

### Proposal [#194](https://github.com/dotnet/vblang/issues/194) - `WithPropertyEvents` Modifier to Enable Easy INotifyPropertyChanged Implementations

**Decisions**
* **Approved for prototype/speclet**.
