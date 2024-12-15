# Feature: Flags Enum Operators

* [x] Proposed
* [x] Prototype: [Proof of Concept](https://github.com/AdamSpeight2008/roslyn-AdamSpeight2008/tree/EnumFlagExpression
  * [x] Syntax & Bound Nodes 
  * [x] Parser
  * [x] Binding 
  * [x] Lowering
  * [x] Error Messages *(Will need updating)*
  * [x] Syntax Highlighting
  * [ ] Constant Folding
* [ ] Implementation: [In Progress](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

## Summary ^([VBLANG Issue#228](https://github.com/dotnet/vblang/issues/228))^
[summary]: #summary

Simplifyy the usage of flags enums, by providing a set of operators.

## Motivation
[motivation]: #motivation

Imagine we are using the follow set of enum flags.
```vbnet
<System.Flags>
Enum Flags
  Node  = 0
  Red   = 1
  Green = 2
  Blue  = 4
End Enum
```
* There are four basic operations we do on flags
  * **IsSet**
    * `IsSet = (MyFlags And Flags.Red) = Flags.Red`
  * **IsClear**
    * `IsClr = (MyFlags And Flags.Red) <> Flags.Red` 
  * **Set**
    * `MyFlags = (MyFlags Or Flags.Red)` 
  * **Clear**
    * `MyFlags = (MyFlags And Not Flags.Red)`
  
It would good be if we could define a set of generic extension methods that implement those functions.

```vbnet
Imports System.Runtime.CompilerServices

Public Module FlagsEnumExts

  <Extension>
  Public Function IsSet(Of Fe As <Flags>Enum)( Flags As Fe, Flag As Fe) As Boolean
    Return (Flags And Flag) = Flag
  End Function

  <Extension>
  Public Function IsClr(Of Fe As <Flags>Enum)( Flags As Fe, Flag As Fe) As Boolean
    Return (Flags And Flag) <> Flag
  End Function

  <Extension>
  Public Function [Set](Of Fe As <Flags>Enum)( Flags As Fe, Flag As Fe) As Fe
    Return (Flags Or Flag)
  End Function

  Public Function Clr(Of Fe As <Flags>Enum)( Flags As Fe, Flag As Fe) As Fe
    Return (Flags And Not Flag)
  End Function

End Module
```

Unfortunatly this isn't currently possible in VB.net (as of VB15.5).
We could us a non generic implementation, but that incurs a runtime reflection cost.

  
### Overview of the operators

This is proposal introduces the concept of a **Flags Enum Operator**.
A Flags Enum operator consist of three parts
  * `Flags`
    * The flags enum *(or variable of a flags enum)* being work on.
  * `!`
    * The operation being performed.
  * 'Flag`
    * The flag to work with.
    * Note: `flag` is a literal referencing the member of the flags to use.
    * Note: It isn't a reference to variable. *Maybe support in the future?*

##### `!` IsFlagSet Operator
  * `Flags ! Flag` 
    * This borrows syntax from them dictionary lookup operator `!`.
    * This usage is analogous to `IsSet`, which returns a `Boolean`. 
    * `IsClr` is supported via negation `Not flags!flag` or comparing against `flags!flag = false`.

##### `!+` SetFlag Operator
  * `Flags !+ flag`
    * This is analogous to `[Set](Of Fe As <Flags>Enum)(flags As Fe, Flag As Fe) As Fe`

##### `!-` ClearFlag Operator
  * `flags!-flag`
    * This is analogous to `Clr(Of Fe As <Flags>Enum)(flags As Fe, Flag) As Fe`
  * These two operators will have to special case in the compiler.
    * As not to mean;-
        * `!+`  `(flags : Single) + DirectCast(flag : Fe.UnderlayingType)`
        * `!-`  `(flags : Single) - DirectCast(flag : Fe.UnderlayingType)`    
    respectively.
	* Ie an *identifier* followed by the *single_type_character* 

All of the operators act immutablly, returning a new instance of the flags enum.


Example code:

```VB.NET
' ...
Dim myFlags = Flags.None
Dim flagsA = myFlags !+ Red !+ Green !+ Blue
Dim flagsB = flagsA !- Green
' Explict comparision used for clarity.
If flagsA ! Green = True Then
  ' ...
ElseIf flagsB ! Green = True Then
  ' ...
End If
' ...
```

--------

## Advantages

  * Simplified usage.
  * Compiler-Time verification of
     * supported enums.
     * Compile-Time Constant Folding. *As they are essentially constants.*
  * Could also provide language support the proposed CoreFX Api Update ([CoreFX Enum Improvements](https://github.com/dotnet/corefx/issues/15453))
    * By changing the lowering.
    * Akin to ValueTuples

## Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

## Alternatives
[alternatives]: #alternatives

  * Extension Method
    * Still Require a runtime reflection.
    * Not restricted to just `<Flags> Enum`
    * VB don't support *(as of this proposal)* attributes on generic type parameters
    
  * @AnthonyDGreen
    * Non-Immutable.
      * Looks to mutate the existing variable. 
      * Breaking change.
        * >This could technically be a very minor breaking change because VB today lets you define an "extension readonly default property" on any type by defining an ElementAtOrDefault extension function and a Select extension method to make it a queryable type. There is no way to emulate the case of making the default property writeable.
          >This entire scenario seems extremely unlikely but the breaking change could be avoided entirely by binding this way only after the queryable/ElementAtOrDefault check fails.
 
   * CoreFX Api Update ([CoreFX Enum Improvements](https://github.com/dotnet/corefx/issues/15453))
     * Requires a change at the core library level.


-------------------------

### Implementation Details

## Visual Basic Specification Changes

`DictionaryAccessExpession ::= (Expr : Expression?) '!' (Key : IdentifierOrKeyword);`

+ If `Expr` evaluates to an `Enum` 
  + **True:** If `Expr` has the `<Flags>` attribute.
    + **True:** Then parse `Key` as a member of that enumeration. eg `Red`, `Blue`, `Green`.
      + If `Key` is valid member then 
         + If the `Key` is the default value (`None` in our example) mark the key as a warning.
 As usage should result in the value not being changed.
         + Otherwise this is valid representation
      + Otherwise Mark the `key` as an error *`Not a member of enumeration`*
    + **False:** Mark `expr` as an error. *`Enum doesn't have the attribute <Flags>`*
  + **False** Continue as a normal dictionary lookup expression.

#### Parsing

Should be relatively simple to extend existing support for dictionary lookup operator.

Existing usage `Dim results = Flags!Flag` results in an error.    
`Error	BC30103	'!' requires its left operand to have a type parameter, class or interface type, but this operand has the type 'Main.Flags`

Would be enable via a feature flag, to preserve compatability.

The operator `!+` and `!-` will require peeking at the next character.
  * The parser already does this to see if is an identifier, for the dictionary lookup
  * So adding a couple of additional check
    * Is next character "+"c, parses as a potential FlagSet operator
    * Is next character "-"c, parses as a potential FlagClear operator

#### SyntaxNode
```
Syntax FlagsEnumOperatorToken Inherits SyntaxToken
  DefaultFactory        = True
  DefaultTrailingTrivia = None

  NodeKinds
    "!+" -> FlagsEnumSetToken
    "!-" -> FlagsEnumClearToken
    "!"  -> FlagsEnumIsSetToken
  End NodeKinds


End Syntax

Partial Syntax FlagsEnumOperationExpressionSyntax Inherits ExpressionSyntax

  NodeKinds
    FlagsEnumOperationExpression
  End NodeKinds

   <Description>Then expression on the left-hand-side of the flags enum operator.</Description>
   .EnumFlags?    As ExpressionSyntax 
  
   <Description></Description>
  .OperatorToken As FlagsEnumOperatorTokrn

   <Description>The member of the flags enum, after the flags enum operator.</Description>
  .EnumFlag      As SimpleNameSyntax 
End Syntax

Enum FlagsEnumOperatorKind
  .None  = 0
   <Description>The "!" FlagsEnum Operator.</Description>
  .IsSet = 1
   <Description>The "!+" FlagsEnum Operator.</Description>
  .[Set] = 2
   <Description>The "!-" FlagsEnum Operator.</Description>
  .Clear = 3
End Enum
```

#### BoundNode

```
Bound BoundFlagsEnumOperationExpressionSyntax Inherits ExpressionSyntax
  Overrides Type       As TypeSymbol            (Null:= Disallow)
            EnumFlags  As BoundExpression       (Null:= Disallow)
            Op         As FlagsEnumOperatorKind (Null:= NotApplicable)
            EnumFlag   As BoundExpression       (Null:= Disallow)
End Bound
```

