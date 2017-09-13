# Visual Basic Language Design Meeting
August 23rd, 2017

## Agenda
* Late-binding without `Option Strict Off`
    * `Dynamic` pseudo-type
    * Method-scoped `Option` and `Imports` statements
    * Late-bound Member-Access expressions
    * Variable-scoped late-binding
    * Erased interfaces
* `Out` arguments
* Pipe-forward (`->`) operator

## `Dynamic` pseudo-type

Feels like it could be a light-weight solution.

Q: What's the stack-ranking on this feature?

Q: Is it just a safe alias for `Object` or is it more like C# `dynamic` in that it always late-binds even if a static member is available.

## Method-scoped `Option` and `Imports` statements

Q: Do we really want `Imports`? Seems kinda crazy.
Could be useful, should break it out into another proposals.

Between this and a `Dynamic` type, I like this better.

This could dove-tail with an `Option Checked`.

`Option`/`End Option` block. Blocked-scoped support types and namespaces in with block

## Late-bound Member-Access expressions

! means so much.

There's a feature here, but it might not add its weight.
The JSON thing is great. Oh, but it already works with ! so all the value just evaporated.

## Erased interfaces

Needs more design

## `Out` Arguments

Investigate C# -> VB -> C# and C# -> VB -> VB

New warning on a VB override

Recognize the OutAttribute, require no In.
Don't add an Out argument modifier
Out parameter separately.

