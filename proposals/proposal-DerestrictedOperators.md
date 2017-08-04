# Derestricted Operators

* [x] Proposed: [Complete](https://github.com/dotnet/vblang/issues/26)
* [ ] Prototype: [Complete](https://github.com/PROTOTYPE_OWNER/roslyn/BRANCH_NAME)
* [ ] Implementation: [In Progress](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

## Summary
[summary]: #summary

Remove some of the restriction placed on operators.

## Motivation
[motivation]: #motivation

Under the current rules of the VB.net language, some operators require to be implementation in complement pairs.

[]

| Op  | Comp |
|----|----|
|  <   |  =>  |
|  <=  |  >   |
|  >   |  <=  |
|  >=  |  <   |
|  <>  |  =   |
|  =   |  <>  |
| IsTrue | IsFalse |
| IsFalse | IsTrue |

```vb
Public Shared Operator <( x As Foo, y As Foo) As Boolean

End Sub

Public Shared Operator >=( x As Foo, y As Foo) As Boolean

End Sub
```

Or in proof of concept, commenting out the production of the error.
```
    diagnostics.Add(ErrorFactory.ErrorInfo(ERRID.ERR_MatchingOperatorExpected2,
      SyntaxFacts.GetText(OverloadResolution.GetOperatorTokenKind(nameOfThePair)),
      method), method.Locations(0))
```


---------------
## Detailed design
[design]: #detailed-design


This proposal is to remove that restriction, by skip the following section of under a language feature conditional.
[Source](https://github.com/dotnet/roslyn/blob/508c229005f4f6c556a54a78905971b1bc3bb9c7/src/Compilers/VisualBasic/Portable/Symbols/Source/SourceMemberContainerTypeSymbol.vb#L3630)



This is the bulk of the proposal. Explain the design in enough detail for somebody familiar
with the language to understand, and for somebody familiar with the compiler to implement,  and include examples of how the feature is used. This section can start out light before the prototyping phase but should get into specifics and corner-cases as the feature is iteratively designed and implemented.

Example code:

```vbnet
Module Module1

    Sub Main()
        Dim lower_limit = 0
        Dim upper_limit = 100
        Dim value_A = 50
        Dim value_B = -10
        Dim value_C = 180
        Dim res_A = lower_limit.__ < value_A <= upper_limit
        Dim res_B = lower_limit.__ < value_B <= upper_limit
        Dim res_C = lower_limit.__ < value_C <= upper_limit

    End Sub

End Module

Public Class CMPOP_A(Of T As IComparable(Of T))

    Private _Value As T

    Sub New(value As T)
        _Value = value
    End Sub

    ' First part of Range Check is done here. Also note we have to remember what the middle value is over to the next part.
    Public Shared Operator <(x As CMPOP_A(Of T), y As T) As CMPOP_B(Of T)   ' <-- Not a Boolean
        Return New CMPOP_B(Of T)(y, x._Value.CompareTo(y) < 0)
    End Operator

    Public Shared Operator <=(x As CMPOP_A(Of T), y As T) As CMPOP_B(Of T)   ' <-- Not a Boolean
        Return New CMPOP_B(Of T)(y, x._Value.CompareTo(y) <= 0)
    End Operator

End Class

Public Class CMPOP_B(Of T As IComparable(Of T))

    Private _Value As T
    Private _State As Boolean

    Sub New(Value As T, state As Boolean)
        _Value = Value
        _State = state
    End Sub

    ' If the State is False already it skips the comparison and returns False.
    Public Shared Operator <(x As CMPOP_B(Of T), y As T) As Boolean
        Return x._State AndAlso (x._Value.CompareTo(y) < 0)
    End Operator

    Public Shared Operator <=(x As CMPOP_B(Of T), y As T) As Boolean
        Return x._State AndAlso (x._Value.CompareTo(y) <= 0)
    End Operator

End Class

Public Module Exts
    <Runtime.CompilerServices.Extension>
    Public Function __(Of T As IComparable(Of T))(value As T) As CMPOP_A(Of T)
        Return New CMPOP_A(Of T)(value)
    End Function
End Module


```

When trying to use a now non-existent operator.

eg
```vb
Dim res_A = lower_limit.__ > value_A <= upper_limit
```

`Error	BC30452	Operator '>' is not defined for types 'CMPOP_A(Of Integer)' and 'Integer'.`






## Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

## Alternatives
[alternatives]: #alternatives

Change the error, to a warning.

## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?