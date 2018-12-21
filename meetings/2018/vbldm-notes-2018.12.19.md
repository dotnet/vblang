# Visual Basic Language Design Meeting

December 19, 2018

Last week's meeting subsumed with other issues.

## Agenda

* Pattern Matching discussion/proposal from community

It's been a while since we've had Language Design in our weekly meetings because folks on the LDM have been distracted with the VB runtime and have not been doing language work, since [it will be a while before we can implement new features](https://blogs.msdn.microsoft.com/vbteam/2018/11/12/visual-basic-in-net-core-3-0/). These realities and timeline haven't changed, but we had a fun meeting on pattern matching in relation to the [VBLang issue #337](https://github.com/dotnet/vblang/issues/337).

In general, we like to watch community issues - our involvement can be "leading the witness" and potentially miss exploring an avenue. In this case, the conversation had made significant progress and it was a good time to review, and it was requested.

It was really gratifying that we started with the initial issue and early comments, then had a very free-wheeling discussion on what it might look like, then found our conclusions were in very close alignment with [this summary of the community conversation to date](https://github.com/dotnet/vblang/issues/337#issuecomment-448427815). Fantastic job by the community at VBLang looking at the problems from many directions. 

We all want to do pattern matching, it's the thing we are most excited about after C# interop issues. The time-frame we think is practical remains around VB.NET 16.2. And, while we are optimistic, like all such discussions, we are not making a commitment to doing this.

For background: a pattern is not an expression, but a thing that when matched results in an expression, in this context a Boolean expression.

## Overall thoughts and criteria

* No breaking changes.
* Think ahead so the initial design doesn't box us away from things we might want later.
* Where we need to make a decision, we will follow C# unless there is a compelling reason to avoid adding more subtle differences between the languages (many programmers work in C# and VB.NET).
* Balancing the previous, we want pattern matching to be VB-like as possible - for example, C# restrictions regarding constant expressions may not be appropriate.
* A phased release allows the community to understand concepts individually (this concept is not VB.NET specific, this was done in C#). The likely order (in separate, not necessarily sequential versions):
  * Declaration pattern - type match with assignment equivalent and When clause
  * Recursive patterns (including tuple patterns)
  * Maybe _and_ and _or_ and _not_ patterns later (still uncertain on this)

## Comments on summary posted Dec-18, 2018

It seems most appropriate to structure our thoughts in relation to [this comment's summary](https://github.com/dotnet/vblang/issues/337#issuecomment-448427815). 

#1. Both `Matches` and `Is` were discussed. We think `Is` will have ambiguity issues with the existing use for reference equality. We might be able to finesse the use of `Is` with operators, by expanding thinking about what a pattern is, but other scenarios are more problematic. `Matches` is the proposed keyword.

```VB
If o Matches x As String
   ' ...
End If

Select Case o
   Case x As String
      ' ...
End Select
```

Not all of us are happy with the reading of the `If` syntax. The human English wording would be more like "If o matches string, put it in x as a string." While we didn't find an alternative in the meeting, we've since found [this suggestion by @AdamSpeight2008](https://github.com/dotnet/vblang/issues/119#issuecomment-313021036) which leads to considering the alternative:

```VB
If o Matches String Into x
   ' ...
End If

Select Case o
   Case String Into x
      ' ...
End Select
```
  
This will lead to a discussion about whether it is more important to read like English or to look like a declaration here.

We are not ready for a negation pattern, but think that would work better than a `DoesntMatch` keyword. 

We agree that an additional keyword is not required in all `Case` cases, we haven't removed the possibility to remove ambiguity or to increase readibility.

#2. We are a little confused. The effect of [#119](https://github.com/dotnet/vblang/issues/119#issue-239288726) seems desirable, but not sure whether this is a pattern or evaluation (or what distinctions matter here).

#3. We thought about commas. If patterns are evaluated in certain scenarios, like method parameters, commas are a problem. For example, similar to a pattern in another thread was:

```VB
Select Case o
   Case 1 To 10, 47
      ' stuff
End Select
```

But this syntax can't be used as a method argument, because it would be unclear whether 47 is a pattern or a second argument.

```VB
f(Matches 1 To 10, 47)
```

Our resolution was for the comma to remain a special feature of `Case`, not a part of the pattern syntax.

This also results in a natural answer for whether When should apply to each part of a multi-part Case match. This syntax might be expressed like (whether or not parens are ultimately legal syntax here):

```VB
Select Case o
   Case (x As String [When <expression>]), (y As Object [When <expression >])
      ' ...
End Select
```

Certainly everything that works in the context of a `Case` today should continue to work in a `Case`. But hesitate on moving syntax from `Case` to other places patterns can be used.

#4. The linked "full range of potential patterns" is for F# and several of these are not available in other .NET languages. 

It's not clear how variable introduction without typechecking would work, or how type checking without assignment differs from the available `TypeOf x  Is <type>`.

#5. We like `When`. 

#6. Can we get clarity on this. Is this basically saying there can't be ambiguity and we can't break existing code? 

#7. Do you mean `Case` should not require `Is` where it does not require it today?

8# i) We need some compelling cases for non-matches. The most compelling case for patterns is TypeCheck/assignment, and that doesn't seem to make sense for non-matches. 

#8 ii) If you mean exhaustiveness of cases in `Select Case`, we don't have this today and introducing it is backwards breaking. 

#8 iii) Need clarity on what this is saying

#8 iv) Agree. We don't usually let variables be used before they are declared, so it would look quite weird to allow this in one case (`DoBottomLoopStatement`)

## Current state of the grammar

We are very, very early in this process and this is open to a great deal of iteration, but the best current expression of the grammar is this, which is copied from [@zpitz summary of the community conversation here](https://github.com/dotnet/vblang/issues/337#issuecomment-448427815) with the changes noted:

```
// LDM: Throughout: "PatternExpression" has been replaced with "Pattern". 
// This probably needs a bit more work, because the implication is that 
// When is part of the pattern and some confusion in that part below. 
// The resolution needs to be clear that a pattern is not an expression. 

BooleanExpressionOrPattern
    : BooleanExpression
    | Expression 'Matches' Pattern
    ;


// If...Then..ElseIf blocks
// LDM: The scope of introduced variables is the surrounding scope

BlockIfStatement
    : 'If' BooleanExpressionOrPattern 'Then'? StatementTerminator
      Block?
      ElseIfStatement*
      ElseStatement?
      'End' 'If' StatementTerminator
    ;

ElseIfStatement
    : ElseIf BooleanExpressionOrPattern 'Then'? StatementTerminator
      Block?
    ;

LineIfThenStatement
    : 'If' BooleanExpressionOrPattern 'Then' Statements ( 'Else' Statements )? StatementTerminator
    ;


// Loops
// LDM thinks introduced variables scope to the While block
WhileStatement
    : 'While' BooleanExpressionOrPattern StatementTerminator
      Block?
      'End' 'While' StatementTerminator
    ;

// introducing variables with either While or Until could only be used 
// by the When clause, not within the block
// LDM thinks probably within the block as well
DoTopLoopStatement
    : 'Do' ( WhileOrUntil BooleanExpressionOrPattern )? StatementTerminator
      Block?
      'Loop' StatementTerminator
    ;

// introducing variables with either While or Until could only be used 
// by the When clause, not within the block
DoBottomLoopStatement
    : 'Do' StatementTerminator
      Block?
      'Loop' WhileOrUntil BooleanExpressionOrPattern StatementTerminator
    ;

ConditionalExpression
    : 'If' OpenParenthesis BooleanExpressionOrPattern Comma Expression Comma Expression CloseParenthesis
    | 'If' OpenParenthesis Expression Comma Expression CloseParenthesis
    ;


// Within a Case clause

CaseStatement
    : 'Case' Pattern StatementTerminator
      Block?
    ;
What are the parts of a pattern expression?

Pattern
    : Pattern ('When' BooleanExpression)?
    ;
What patterns should be supported from the start?


// LDM reworded: 
Multiple Patterns
    // multiple patterns are separately evaluated and each has it's own when clause
    // multiple patterns are only supported in case
    : Pattern ',' Pattern                    // OR pattern (already supported in Case)

Patterns
    // LDM wants to understand need for TypeCheck without assignemnt.
    | 'As' TypeName                          // Type check pattern -- matches when subject is of TypeName
    // LDM wants to understand usage without type
    | 'Dim' Identifier ('As' TypeName)?      // Variable pattern -- introduces a new variable in child scope; as TypeName or Object
    // LDM: Tentatively thinking of other Case syntax as a pattern may be useful, but these might remain specific to Case
    | 'Is'? ComparisonOperator Expression    // Comparison pattern
    | 'Like' StringExpression                // Like pattern
    | Expression 'To' Expression             // Range pattern
    | Expression                             // Expression pattern -- value/reference equality test against Expression
    ;
```

## Other Notes

* What does "nested patterns" mean? Maybe not V1
* `If obj Is String Then Console.WriteLine("obj is a string")` already available with `TypeOf..Is`
* If we do _and_ and _or_ later, AndAlsoMatches/OrElseMatches is one option. Conjunctions are probably not in first or second version, C# thinking is these are much lower need/usage.
* Probably add a discard identifier and a discard pattern
