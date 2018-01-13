# Feature: Flags Enum Operators

* [x] Proposed
* [ ] Prototype: **Very Experimental** [Proof of Concept](https://github.com/AdamSpeight2008/roslyn-AdamSpeight2008/tree/EnumFlagExpression)
* [ ] Implementation: [In Progress](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

## Summary
[summary]: #summary

Simplifyy the usage of flags enums.

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


## Detailed design
[design]: #detailed-design

### Overview of the operators

This is proposal introduces the concept of a **Flags Enum Operator**.
A Flags Enum operator consist of three parts
  * `Flags`
    * The flags enum *(or variable of a flags enum)* being work on.
  * `!`
    * The operation being performed.
  * 'flag`
    * The flag to work with.
    * Note: `flag` is a literal referencing the member of the flags to use.
    * Note: It isn't a reference to variable.

Initially one will be supported
  * `flags!flag` 
    * This borrows syntax from them dictionary lookup operator `!`.
    * This usage is analogous to `IsSet`, which returns a `Boolean`. 
    * `IsClr` is supported via negation `Not flags!flag` or comparing against `flags!flag = false`.

Two other additional operator, are to be consider also;-
  * `flags!+flag`
    * This is analogous to `[Set](Of Fe As <Flags>Enum)(flags As Fe, Flag As Fe) As Fe`
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
Dim flagsA = myFlags!+Red!+Green!+Blue
Dim flagsB = flagsA!-Green
' Explict comparision used for clarity.
If flagsA!Green = True Then
  ' ...
ElseIf flagsB!Green = True Then
  ' ...
End If
' ...
```

### Parsing

Should be relatively simple to extend existing support for dictionary lookup operator.

Existing usage `Dim results = Flags!Flag` results in an error.    
`Error	BC30103	'!' requires its left operand to have a type parameter, class or interface type, but this operand has the type 'Main.Flags`

Would be enable via a feature flag, to preserve compatability.

The operator `!+` and `!-` will require additional parsing code to support.

### SyntaxNode
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

### BoundNode

```
Bound BoundFlagsEnumOperationExpressionSyntax Inherits ExpressionSyntax
  Overrides Type       As TypeSymbol            (Null:= Disallow)
            EnumFlags  As BoundExpression       (Null:= Disallow)
            Op         As FlagsEnumOperatorKind (Null:= NotApplicable)
            EnumFlag   As BoundExpression       (Null:= Disallow)
End Bound
```

## Advantages

  * Simplified usage.
  * Compiler-Time verification of supported enums. *As they are essentially constants.*

## Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

## Alternatives
[alternatives]: #alternatives

What other designs have been considered? What is the impact of not doing this?

## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?
  * Shape of SyntaxNodes
  * Shape of BoundNode