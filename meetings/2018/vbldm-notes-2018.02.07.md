# Visual Basic Language Design Meeting
February 7,2018

## Agenda
* [C# Issue #1208 - C# Design Notes for nullable reference types](https://github.com/dotnet/csharplang/issues/1208)
* [Issue #117 - Block-scoped Option Statements](https://github.com/dotnet/vblang/issues/117)
* [Issue #211 - Discussion / proposal: usage of any installed programming language](https://github.com/dotnet/vblang/issues/211)

## C# Issue [#1208](hhttps://github.com/dotnet/csharplang/issues/1208) - C# Design Notes for nullable reference types

### Part 1: Implementing analysis

C# has a prototype for "nullable reference types" which uses flow analysis to establish whether a reference is intended to allow null. 

The user opts in per assembly. For other than a greenfield project, this results in a wall of compiler warnings, the user then works through these to resolve.

There are probably a significant number of VB (and C#) projects where retrofitting this behavior is not desirable. While these warnings are valuable, it's unclear how popular this feature will be. And it may feel like a "not VB" thing as we understand its usage.

We'll postpone this until we understand the uptake in C#.

### Part 2: Marking an assemblies surface area with null intention

If a project opts into nullable reference warnings, it assumes all values received from assemblies that do not opt in may contain nulls. 

There will be a technique for annotating assemblies' surface area for nullability. This is important for the .NET frameworks. It is not clear how this will work, and at least at the implementation level it won't be as simple as managing attributes. 

Implementing this would allow VB projects to express their nullability to C# projects, even if the VB project did not do null checks internally.

It is possible that VB could add the emitting attributes based on a simpler attributing system

We'll postpone this until we know how the framework will handle this scenario and how much interest there is in this cross language support.

## Issue [#117](https://github.com/dotnet/vblang/issues/117) - Block-scoped Option Statements
_Related: [#255](https://github.com/dotnet/vblang/issues/255) - Localised Compiler Options_

These issues suggest being able to limit the scope of Option Strict to a smaller block than the file.

This is a convenience feature. Options are set per file, not per class. This means that any code that needs looser Options could be refactored into another method and placed in a Partial class.

Considering all the options

* Option Strict: Would provide value, possible to do
* Option Explicit: Not seeing value, difficult to do
* Option Compare: Not seeing high value, could create confusing code
* Option Infer: This doesn't really make sense to us

Leaving these two issues open for now, and requesting info on why #255 is not a duplicate. 

## Issue [#211](https://github.com/dotnet/vblang/issues/211) - Discussion / proposal: usage of any installed programming language

Fantastic idea, and too hard to do.

Lots of thought has gone into this over the years. It was considered in depth when Roslyn was designed, and has been considered since. It would require rewriting considerable parts of Roslyn.

It could be simplified somewhat if each type declaration was fully in one language or another. But, it would still need a new compiler and significant IDE work. 

And that would still leave open resolution of the many differences between C# and Visual Basic. The easiest of these differences to visualize are case sensitivity and overloads resolution.

After rethinking again, the previous assessment that this is not practical stands. 

We believe it is important to communicate this clearly to the community so that we can, move toward other tractable issues. We will not close the issue for now to allow further discussion.
