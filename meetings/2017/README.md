# Visual Basic .NET Language Design Notes for 2017

Overview of meetings and agendas for 2017

**April**
* [12th](vbldm-notes-2017.04.12.md)
  * Proposal [#59](https://github.com/dotnet/vblang/issues/59): New conversion operator/syntax (`As Type`/`CVal`)

**May**
* [19th](vbldm-notes-2017.05.19.md)
  * Default Interface Implementations

**August**
* [9th](vbldm-notes-2017.08.09.md)
  * Issue [#37](https://github.com/dotnet/vblang/issues/37): Await in Catch/Finally
  * Non-trailing named arguments
* [23rd](vbldm-notes-2017.08.23.md)
  * Scenario [#135](https://github.com/dotnet/vblang/issues/135): Late-binding without `Option Strict Off`
    * Proposal [#136](https://github.com/dotnet/vblang/issues/136): `Dynamic` pseudo-type
    * Proposal [#117](https://github.com/dotnet/vblang/issues/117): Method-scoped `Option` and `Imports` statements
    * Proposal [#137](https://github.com/dotnet/vblang/issues/137): Late-bound member-access expressions
    * Proposal [#43](https://github.com/dotnet/vblang/issues/43): Variable-scoped late-binding
    * Proposal [#106](https://github.com/dotnet/vblang/issues/106): Erased interfaces
  * Proposal [#152](https://github.com/dotnet/vblang/issues/152): `Out` arguments
* [30th](vbldm-notes-2017.08.30.md)
  * Nullable reference types

**September**
* 27th

**October**
* 4th
* [18th](vbldm-notes-2017.10.18.md)
  * Proposal [#101](https://github.com/dotnet/vblang/issues/101): JSON Literals
    * Pattern matching
  * Annotated types
    * Proposal [#184](https://github.com/dotnet/vblang/issues/184): Tagged String literals (Related [#27](https://github.com/dotnet/vblang/issues/27): Guid literals)
    * JSON types for JSON IntelliSense
    * XML types for XML IntelliSense
  * Proposal [#190](https://github.com/dotnet/vblang/issues/190): `Try` assignment

**November**
* [15th](vbldm-notes-2017.11.15.md)
  * Proposal [#195](https://github.com/dotnet/vblang/issues/195) - `Static` Property Variables
  * Proposal [#196](https://github.com/dotnet/vblang/issues/196) - Implicit Property Backing Fields
  * Proposal [#197](https://github.com/dotnet/vblang/issues/197) - Inferred `Set` Parameter Type
  * Scenario [#219](https://github.com/dotnet/vblang/issues/219) - Implementing INotifyPropertyChanged is Tedious
    * Proposal [#107](https://github.com/dotnet/vblang/issues/107) - Replaceable Members
    * Proposal [#198](https://github.com/dotnet/vblang/issues/198) - Bindable Classes and Properties
    * Proposal [#107](https://github.com/dotnet/vblang/issues/194) - `WithPropertyEvents` Modifier to Enable Easy INotifyPropertyChanged Implementations
    
**December**
* [6th](vbldm-notes-2017.12.06.md)
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
