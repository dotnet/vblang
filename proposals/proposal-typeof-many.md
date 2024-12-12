# TypeOf Many & TypeOf Into Expressions

* [x] VBLang Issue [#23](https://github.com/dotnet/vblang/issues/23)
* [x] Proposed
* [x] Prototype: [Proof Of Concept](https://github.com/AdamSpeight2008/roslyn-AdamSpeight2008/tree/TypeOf_Into)
  * [x] Syntax and Bound Nodes
  * [x] Parser
  * [x] Binder
  * [x] Lowering
  * [ ] Tooling Support  
* [ ] Implementation: [In Progress](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

---------

## TypeOfManyExpression Summary
[typeofmanyexpressionsummary]: #typeofmanyexpressionsummary
Extend the capability of `TypeOf obj Is ...` and `TypeOf obj IsNot ...` to check against many types. 

## Motivation
[typeofmanyexpressionmotivation]: #typeofmanyexpressionmotivation

**What cases does it support?**

This proposal is tightly focused on the two following forms, commonly used to check an object's type against multiple possible types.
```vb.net
dim result0 = (TypeOf obj Is T0) OrElse (TypeOf obj Is T1) OrElse (TypeOf obj Is T2) OrElse (TypeOf obj Is T3) 
dim result1 = (TypeOf obj IsNot T0) AndAlso (TypeOf obj IsNot T1) AndAlso (TypeOf obj IsNot T2) AndAlso (TypeOf obj IsNot T3) 
```
**With Propose Syntax**
```vb.net
dim result0 = (TypeOf obj Is (Of T0,T1,T2,T3)) 
dim result1 = (TypeOf obj IsNot (Of T0,T1,T2,T3)) 
```
**What is the expected outcome?**   
The proposed syntax is semantically equivalent to writing the previous form.

**Why are we doing this?**    
Reduces the visual noise when expressing this intent.

## Detailed design
[typeofmanyexpressiondesign]: #typeofmanyexpressiondetailed-design
>*This is the bulk of the proposal. Explain the design in enough detail for somebody familiar
with the language to understand, and for somebody familiar with the compiler to implement,  and include examples of how the feature is used. This section can start out light before the prototyping phase but should get into specifics and corner-cases as the feature is iteratively designed and implemented.*

The grammar of a `TypeOf` expression is similar to the following BNF implementation
```
TypeOfExpression ::= TypeOfKeyword ws+ Expression TypeOfOperand ws+ Target
TypeOfKeyword    ::= "TypeOf"
TypeOfOperand    ::= (IsOperand | IsNotOperand )
IsOperand        ::= "Is"
IsNotOperand     ::= "IsNot"
Target           ::= TypeIdentifer
```
The proposal is to extend the rule `Target` to 
```
Target ::= TypeIdentifer | TypeArgumentList
```

`TypeArgumentList` is the form used for when specifying generic type arguments `(Of T0, T1 ...)`.

------

## TypeOfIntoExpression Summary
[typeofintoexpressionsummary]: #typeofintoexpressionsummary
Extends `TypeOf expression Is type` to `TypeOf expression Is type Into variable`.
Where if the first part of the expression is true, also cast the expression to type.

## Motivation

```vbnet
Dim variable As T = Nothing
If TypeOf expression Is T Then
    variable = DirectCast(expression, T)
End If
``` 

This also has the benefit of the allowing use the resultant cast further with in the expression. eg.
```vb
If TypeOf obj Is Int32 Into value AndAlso value > 0 Then

```

**What cases does it support?**

The only form of `TypeOfExpression` supported is `TypeOf expression Is type`.

*Question*   
Should we also support `TypeOfManyExpression`, where the target type is dominate type, of the types in the list?
```vbnet
 Dim variable As ExpressionSyntax
 If TypeOf expression Is (Of TryCastExpression, DirectCastExpression) Into variable Then
```

## Lowering

When lowering `IntoVariableExpression` that have `TypeOfExpression` on the left.
Then the one of the following lowerings are use.

If the `targettype` is `String` the expression is equivalent to calling the following function
` If [Is](Of type)( expression, variable ) Then` 
```vbnet
Function [Is](Of T As Class)( exression As Object, <Out> ByRef variable As T) As Boolean
  variable = TryCast( expression, T)
  Return variable IsNot Nothing
End Function
``` 

Otherwise 
```vbnet
Function [Is](Of T As Structure)( expression As Object, variable As T ) As Boolean
  Dim tmp As T? = DirectCast(expression, T?)
  variable = tmp.GetValueOrDefault()
  Return tmp IsNot Nothing
End Function
```


## Detailed design
[typeofintoexpressiondesign]: #detailed-design

Strictly speaking `TypeOfIntoExpression` isnot an expression but an 'IntoVariableExpression` that has a `TypeOfExpression` as its `Expression` argument.

**Grammar**    ` IntoVariableExpression ::= expression "Info" expression ; `

Acts like a binary operator expression
` [Into](expression As ExpressionSyntax, ByRef variable As ExpressionSyntax ) As ExpressionSyntax `

The type of returned expression, is the same as the return type as the expression on the left.
This allows it to act transparently in expressions. 

In the prototype its usage is restricted to typeofexpression.

