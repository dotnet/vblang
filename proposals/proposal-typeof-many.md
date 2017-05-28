# TypeOf Many

* [x] Proposed
* [ ] Prototype: [Complete](https://github.com/PROTOTYPE_OWNER/roslyn/BRANCH_NAME)
* [ ] Implementation: [In Progress](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

## Summary
[summary]: #summary
Extend the capability of `TypeOf obj Is ...` and `TypeOf obj IsNot ...` to check against many types. 

## Motivation
[motivation]: #motivation

**What cases does it support?**

This proposal is tightly focused on the two following forms, commonly used to check an object's type against multiple possible types.
```vb.net
dim result0 = (TypeOf obj Is T0) OrElse (TypeOf obj Is T1) OrElse (TypeOf obj Is T2) OrElse (TypeOf obj Is T3) 
dim result1 = (TypeOf obj IsNot T0) AndAlso (TypeOf obj IsNot T1) AndAlso (TypeOf obj IsNot T2) AndAlso (TypeOf obj IsNot T3) 
```
**With Propose Syntax**
```vb.net
dim result0 = (TypeOf obj Is {T0,T1,T2,T3}) 
dim result1 = (TypeOf obj IsNot {T0,T1,T2,T3}) 
```
**What is the expected outcome?**   
The proposed syntax is semantically equivalent to writing the previous form.

**Why are we doing this?**    
Reduces the visual noise when expressing this intent.

## Detailed design
[design]: #detailed-design
>*This is the bulk of the proposal. Explain the design in enough detail for somebody familiar
with the language to understand, and for somebody familiar with the compiler to implement,  and include examples of how the feature is used. This section can start out light before the prototyping phase but should get into specifics and corner-cases as the feature is iteratively designed and implemented.*

The grammar of a `TypeOf` expression is similar to the following BNF implementation
```
TypeOfExpression ::= TypeOfKeyword ws+ Expression TypeOfOperand ws+ Target
TypeOfKeyword    ::= "TypeOf"
TypeOfOperand    ::= (IsOperand | IsNotOperand )
IsOperand        ::= "Is"
IsNotOperand     ::= "IsNot"
Target		 ::= TypeIdentifer
```
The proposal is to extend the rule `Target` to 
```
Target ::= TypeIdentifer | TypeArray
```


#### TypeArray *(name may change)*    

A `typeArray` is `collection initializer list` consisting only `type identifiers`.
```
TypeArray     ::= BraceOpening ws* TypeIdentifer (WS* Comma WS* TypeIdentifer )* ws* Brace_Closing
Brace_Opening ::= '{'
Brace_Closing ::= '}'
Comma         ::= ','
```


## Drawbacks
[drawbacks]: #drawbacks
> Why should we *not* do this?

## Alternatives
[alternatives]: #alternatives
> What other designs have been considered? What is the impact of not doing this?

## Unresolved questions
[unresolved]: #unresolved-questions
> What parts of the design are still TBD?