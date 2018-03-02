# Visual Basic Language Design Meeting

February 28,2018

## Agenda

* Implications of C# Range on Visual Basic
* Issue [#37 -Champion "Await in Catch and Finally"](https://github.com/dotnet/vblang/issues/37)
* C# Issue [#967 Proposal for tuple equality/inequality comparisons](https://github.com/dotnet/csharplang/pull/967)

## Community Update

### MVP Summit

MVP Summit is March 5-8.

## Implications of C# Range on Visual Basic

The C# LDM is discussing possibility adding operators to make range access easier. 

This is to support programs that will use some of the new .NET Core API features for direct memory access and manipulation. These aren't expected to be high usage workflows in VB, but we don't want to block VB programmers from those scenarios.

The work to supply this feature will be about the same for Visual Basic, after the C# feature is specced. There is a high probability that there are more important features for VB. But, let's still take a look at syntax possibilities and more deeply at alternate approaches. 

### Potential Syntax

Since the hat is currently used as a binary operator, it would be available as a unary operator.

`.:` could be used for length.
 
Open ended `x..` and `..x` and `..` also work.

The To as a Range separator would be weird if the range was exclusive as anticipated. Not using that. `..` would work.

```
Dim x = s.Slice(0..11) ' seems pretty VB like
```

We believe that there is non-trivial amounts of VB code that are imprecise about the length of arrays - Dimming with the length instead of 1 minus the length. Consistent usually would generally allow this code to work fine, but if we allow counting from the end, and code has an array one to long, things will not work as expected. Nothing we can do about that.

Wait to see what C# does and whether we need syntax.

### Using API Support

If index provides an implicit conversion from int, then VB can access the API. Or if integers are used, conversions to Range.
 
```
Index i = Index.FromEnd(Int32 i)
 
Range = New Range(2, Index.FromEnd(2))
Dim z1 = Slice(Range)
Dim z2 = Slice(New Range(2, Index.FromEnd(2))

Dim z3 = Slice(Range.StartAt(2))
Dim z4 = Slice(Range.Open)  ' maybe Range.All
Dim z5 = Slice(Range.EndAt(8))
Dim z6 = Slice(Range.FromEnd(2))
```

It might be even cooler if there are conversions from tuples.

```
' Conversion from tuple of Index or integer
Dim z7 = Slice((2,3))
Dim s8 = Slice((2,Index.FromEnd(2)))
```

If the API support is good enough, don't do the language work in VB. These APIs will allow older versions of Visual Basic (and C#) access these APIs and may come out reading more "VB like" than the dot dot range syntax. 

If we can't get good enough for VB users, then reconsider.

Kathleen will discuss possibilities for API with API folks.

NOTE: Following discussions with Immo on 3/1.

The API already has methods of the style discussed here. They do not have final names, so at the point that is firmed up, we can ensure it makes sense for VB. Conversions are possible, although Immo thought it was weird to go just one direction with the conversion and Range to Integer tuple is probably not sensible. 

## Issue [#37 -Champion "Await in Catch and Finally"](https://github.com/dotnet/vblang/issues/37)

In VB you can't Await in a Catch block. This makes it pretty much impossible to do some things after an exception that we assume user's would like to do. The language is blocking these scenarios, so can we fix it. 

@VSadov was present so we can complete last week's discussion.

Await in try catch is implemented by rewriting the code to use a complex state machine.

It is legal in VB (but not C#) to jump from inside a Catch to inside the Try. Await in catch in C# was implemented without support for this (obviously), and we aren't quite sure how to do it because nested Try/Catch create some interesting possibilities. 

If we drop the difficult scenarios and provide support and disallow Await and these jumps to happen together, this is doable. 

Let's do it!

Kathleen to discuss resourcing.

## C# Issue [#967 Proposal for tuple equality/inequality comparisons](https://github.com/dotnet/csharplang/pull/967)

Tuple equality is being considered for C#. Given that x and y are tuples

```VB
If x.Equals(y) Then ' works today if they are of the same type

' The following is not supported
If x = y Then ' in general probably means...
If x.First = y.First AndAlso x.Second = y.Second Then
```

Equality operator is a convenient and a manner of nicer style.

Since the equality operator more different than Equals. It is not a reference equals, for example. Nullable value types also works some place in VB where it doesn't in C#.

This is likely to need different deep thought than C# (more than a port) to find VB behavior (quirks in a good way).

Much of the work in the feature comes when the types are different. Most of the value to the programmer occur when the types are the same.

Ran out of time to complete. 
