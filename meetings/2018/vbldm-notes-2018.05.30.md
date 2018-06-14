# Visual Basic Language Design Meeting

May 30, 2018

We have been largely on break because of travel, Build and other issues. 

## Agenda

* Europe trip report, stability, possible survey
* Issue labels (marking what we've reviewed)
* Issue 305: In and Out operators
* Issue 304: Select TypeOf
* Issue 303: Null conditional operators for add/removeHandler
* Issue 301: Modify how literal strings are parsed
* Issue 167: Support for Return? construct

## Europe trip report, stability, possible survey

We discussed stability, the current VB strategy and input I heard on my Europe trip. We believe the majority of Visual Basic customers (there are hundreds of thousands of quiet customers each month) primarily want VB to keep doing what it does now. It remains striking how few Visual Basic programmers we're able to hear from. We have a small group of important friends and a large number of silent users. We don't know what this larger group wants, they aren't asking us for features.

We will discuss with folks at Microsoft that know about such things the possibility of a survey to try to gather better information. There is no plan to change the strategy, although we'll continue to discuss how we communicate it effectively so that people know Microsoft remains committed to Visual Basic. The survey needs to go to VB programmers only (not a Twitter survey)

## Issue labels

Kathleen will put together a document for review that clarifies how we communicate our review of issues. Initial thoughts: 

* "LDM Reviewed: No plans" as opposed to closing or rejected to avoid shutting down further discussion but communicating that we don't plan to move forward
* "LDM Thinking"
* Orthogonally, group issues that are related to larger areas, such as "Pattern Matching" and "Flagged Enums"

## Issue #305 [In and Out operators](https://github.com/dotnet/vblang/issues/305)

Considered in three pieces: 

### In against a normal list (not types)

This does not seem to have much value - is not significantly more expressive even if shorter - than .Contains against the list. Not moving forward for this reason. 

### In against a list of types

Kathleen's alternative syntax is wrong. TypeOf...Is is not an exact match, but whether a cast can occur. This is a concern with this feature since there is already some confusion around how TypeOf...Is.

The correct syntax is a bit involved, but when would this be useful. You would branch, but would have to do additional casts. It seems much more likely that you'd do a series of If statements or similar.

Not moving forward unless we see significant real world cases, and still have concerns. 

### Out

This is similar to Pattern Matching and Out variables in C#. Out variables are probably covered better in [#60](https://github.com/dotnet/vblang/issues/60) and pattern matching by [#124](https://github.com/dotnet/vblang/issues/124)

Labels: Pattern Matching and LDM Reviewed: No Plans

## Issue #304 [Select TypeOf](https://github.com/dotnet/vblang/issues/304)

Will consider part of pattern matching. 

Labels: Pattern Matching and LDM Reviewed: No Plans

## Issue #303 [Null conditional operators for add/removeHandler](https://github.com/dotnet/vblang/issues/303)

As suggested this would only work if special case lowering or similar occurs because the null is more or less on the assignment side. 

Considered in relation to .NET 3 focus on WinForms/WPF. However, even in that context this would be quite rare (generally humans don't do the AddHandlers thing and generally all controls are instantiated.)

Really seems like a side case.

Labels: LDM Reviewed: No Plans

## Issue #301 [Modify how literal strings are parsed.](https://github.com/dotnet/vblang/issues/301)

To summarize: Unicode is ugly. It's maybe a bit extra ugly in VB. 

We considered `\u3264`. VB has a bit of an advantage of C# here because it can avoid the `u`/`U` ugliness. We considered special casing in interpolated strings, but it seems more logical to just make it a literal if we did it. But it looks so not-VB!

Considered variations with trailing type character like `&H3264c`. That's only nominally better than `ChrW`. People are likely to use constants if they want clarity.

Labels: LDM Thinking

## Issue #167 [Proposal: Support for Return? construct](https://github.com/dotnet/vblang/issues/167)

We flipped over to #167 while looking at #303.

We think this is a bad idea. Control flow would be altered by a very subtle character. It's not the same meaning as other uses as ? (any alteration in control flow)

Labels: LDM Reviewed: No Plans