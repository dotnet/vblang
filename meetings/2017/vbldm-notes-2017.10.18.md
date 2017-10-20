# Visual Basic Language Design Meeting - 2017.10.18

### Agenda
* Proposal #101 - JSON Literals
    * Pattern matching
* Annotated types
    * Proposal #184 - Tagged String literals (#27)
    * JSON types for JSON IntelliSense
    * XML types for XML IntelliSense
* Proposal #XXX - `Try` assignment

## Proposal #101 - JSON Literals

JSON is the _lingua franca_ of the cloud. First-class JSON support could be a strong attractant for first-time developers. The power of copy/paste/modify to jump start a project can't be overstated.

``` VB.NET
        Dim dow =
            {
               "name": NameOf(DayOfWeek),
               "type": "enum",
               "subtype": GetType(DayOfWeek).GetEnumUnderlyingType().Name,
               "members": [
                    (From f In GetType(DayOfWeek).GetFields(Reflection.BindingFlags.Public Or Reflection.BindingFlags.Static)
                     Select {f.Name: CInt(f.GetValue(Nothing))})
               ]
            }
```

**Feedback**
* If you want a top-level array we should drop the square brackets; the above code reads like "member" should be a one-element array whose first element is another array.
  * The explicit syntax does allow interleaving fixed and generated elements in the list, but is that scenario worth it?
* Embedded syntax, should there be any special syntax like XML's `<%= %>`? _No strong feelings_.
* "I like that it's so close to a general dictionary init syntax that it may be better thought of as that" - I would say that new dictionary - followed by that syntax it should work. It could be a superset of JSON.
  * The problem is the nested case. How do we know the List type? Do you restate the type or have a separate list syntax.

### JSON Pattern Matching?
Related: #139 - XML Patterns

I think the principle for making a new "literal" or construction pattern is when the ceremony of creating an object(-graph) obscures the structure of the data itself. Therefore, I proposal the principle for introducing a pattern is when the ceremony of _inspecting_ and object(-graph) obscures the structure of the data.

I haven't actually heard feedback that `.`, `!`, `.<>`, `.@`, and `...<>` obscure the structure of the underlying data when inspecting homogeneous collections. Therefore I believe the motivating example for any specialized patterns for JSON or XML would be dispatching over multiple possible structures nested within a larger one:

``` VB.NET
For Each contact In account!contacts
    Select Case contact
        Case Match { "type": "business",
                     "company_name": company,
                     "address": address }

            ' Handling for businesses.

        Case Match { "type": "individual",
                     "email": email }

            ' Handling for individuals.

        Case Match { "type": "government",
                     "agency": agency,
                     "jurisdiction": jurisdiction }

            ' Handle government agency.

    End Select
Next
``` 

In the above example, each possible contact kind has different content. Inspecting and dispatching correctly would greatly obscure the actual structure of the data.

* Decision: Table, wait for feedback/scenarios and more matching.

## Annotated types

### Proposal #184
Related: #27 - GUID Literals

After discussing the scenario for GUID literals with customers I don't believe they are necessary. There isn't a good case to be made for an expression typed as `System.Guid` as the dominant use case (attributes) can only take compile-time/CLR constants, i.e. strings. The only notable benefit I've heard from users is _validation_. This naturally leads one to consider an analyzer. The problem with the analyzer approach today is that 1) such an analyzer must inspect every string literal in the program, 2) such an analyzer would have to determine if a erroneously typed string were _intended_ to be a GUID before validating it.

A simple general solution is a syntax that lets a user decorate a (magic) string literal with a bit of metadata. This metadata would (like a comment) have absolutely no effect on compilation. It would however make it trivial for a set of validation analyzers to target string literals _intended_ for validation _and_ to know what validation is required.  

Example:

``` VB.NET
? "{123e4567-e89b-12d3-a456-426655440000}"#Guid
```

In all ways such an annotated string literal would behave exactly as a string literal does today and the set of annotation identifiers would be open-ended. However, illustrative examples include: GUIDs, dates, URIs, IP addresses, paths, database connection strings, and format strings.

**Feedback**
* Using attributes would be better than a special tag.
* Attribute should be on the producing APIs rather than the strings themselves. The advantage is that everyone can benefit without changing their code.
  * Can you apply attributes to constants?
* Could this be done with an analyzer? Does IOperation have the perf it needs to do this? _Test it_.

**Bullets dodged**
* **Discussion**: If the compiler understood annotations it could preserve them through type-inference at least.
* **Question**: Could we add semantics that are generalizable? In a manner, nullable and tuple names are "annotations" applied to an underlying type. They enable us to report certain warnings, particularly around implicit conversions when, for example, element names are mismatched. Could be valuable in general to report conversion warnings between strings with different annotations or strings with and without annotations?
* **Question**: Could this be further generalized to annotating any type?

## XML and JSON

Is there an even richer notion of an annotated type which could be used to add and track simple information about XML and JSON literals to better enable the IDE to provide completion when using XML-axis properties or the dictionary-access operator?

``` VB.NET
Dim x As <contact>     ' Type: <contact>
Dim y = x.<address>    ' Type: <contact>.<address>
Dim z = y.<city>.Value
```

``` VB.NET
Dim a As {"contact"} ' Type: {"contact"}
Dim b = a!address    ' Type: {"contact"}.{"address"}
Dim c = b!city 
```

**Feedback**
* This is one end of the spectrum of providing a better tooling experience for untyped data over the wire. Type providers are on the other end.
* We might be able to do a lot with no compiler changes and should investigate with IDE team.

## Proposal #XXX - `Try` assignment
Related: #175, #159 #132 #67 #60

A general solution that solves both multiple return values and the popular .NET `TryXYZ` pattern eludes me. Though it would seem that the "Try" pattern would benefit from using either nullable value types or tuples, neither approach is satisfactory.

Nullables
``` VB.NET
Dim i As Integer? = Integer.TryParseNullable(s)

If i IsNot Nothing Then
    Dim i2 = i.Value ' To avoid the on-going tax of unwrapping the null.
End If

myDictionary(myKey) = Nothing

Dim myValue = myDictionary.TryGetValueNullable(s)
' Null was a valid value here.
```

Tuples
``` VB.NET
Dim r = Integer.TryParseTuple(s)

If r.Succeeded Then
    Dim i = r.Value ' To avoid the on-going tax of unwrapping the tuple.
End If

Dim (succeeded, result) = myDictionary.TryGetValueTuple(s)

If succeeded Then
    ' Do whatever.
End If

' succeeded just hangs around forever despite its transient role having long ago ended. Lamentations. 
```

It turns out that `ByRef` and `Out` remain the preferred solution to this problem. This leads customers to ask us for better support for `Out` vars like C# has. However, VB has long shied away in general from inline assignment and inline declarations and I am loathe to break from this. I find keeping declarations and assignments at the statement level to simplify code and indeed the use of inline assignment trickery in particular is frowned upon even in curly brace languages. 

After some thought, I realize that the reason general data-based solutions don't address this problem is because the "Try" pattern simultaneously conveys _both_ data-flow _and_ control-flow. It's meant to be used in conjunction with an `If` or other control-flow statement and the assignment of a value is relegated to a **side-effect** of the control-flow. I propose that a special solution is needed for this pattern that recognizes the duality: `Try` assignment/initialization.

Single-line
``` VB.NET
Dim offset = Try Integer.TryParse(offsetString) Else Throw New FormatException
Dim count = Try Integer.TryParse(countString) Else Throw New FormatException
```

Fallback
``` VB.NET
Function Factorial(n As ULong) As ULong
    Static cache = New Dictionary(Of ULong, ULong)

    Dim result As ULong

    If n = 0 Then
        result = 1
    Else
        result = Try cache.TryGetValue(n) Else
                     result = n * Factorial(n - 1)
                     cache(n) = result
                 End Try
    End If

    Return result
End Function
```

Positive consequence/early-out
``` VB.NET
Dim root = Try document.TryGetRoot(cancellationToken) Then
               Return root ' Early return.
           End Try

' Do something heavy.
```

The above examples reverse the inversion the "Try" pattern forces on developers by making the control-flow aspect subordinate to the data-flow aspect. This allows type-inference to work as normal and keep variable declarations and assignments at the statement level. I considered other designs made this a new kind of statement but all of them lost those benefits.

The specifics of what patterns the feature would work on (nullables, tuples, out-vars) haven't been specified. But if designed correctly this could yield a very natural transformation for an asynchronous try:

``` VB.NET
Dim customer = Try Await repository.TryGetCustomer(id) Else Return "Not Found"

Return customer.Name 
```

Such a feature, combined with tuples completely eliminates the need for `ByRef` and `Out` arguments outside of performance-based structure copy-avoidance; a very uncommon scenario.

**Feedback**
* Are side-effects really that bad?
* Syntax is weird because the right-hand side of the assignment can assign to the thing it's initializing; should it instead be forced to return a different value?
  * But how would we represent the case where a failed "Try" should result in throwing or early exit?
