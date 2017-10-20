# Visual Basic Language Design Meeting
October 18th, 2017

### Agenda
* Proposal [#101](https://github.com/dotnet/vblang/issues/101): JSON Literals
  * Pattern matching
* Annotated types
  * Proposal [#184](https://github.com/dotnet/vblang/issues/184): Tagged String literals (Related [#27](https://github.com/dotnet/vblang/issues/27): Guid literals)
  * JSON types for JSON IntelliSense
  * XML types for XML IntelliSense
* Proposal [#190](https://github.com/dotnet/vblang/issues/190): `Try` assignment

## Proposal [#101](https://github.com/dotnet/vblang/issues/101) - JSON Literals

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
Related: [#139](https://github.com/dotnet/vblang/issues/139) - XML Patterns

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

### Proposal [#184](https://github.com/dotnet/vblang/issues/184)
Related: [#27](https://github.com/dotnet/vblang/issues/27) - GUID Literals

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

## Proposal [#190](https://github.com/dotnet/vblang/issues/190) - `Try` assignment
Related: [#175](https://github.com/dotnet/vblang/issues/175), [#159](https://github.com/dotnet/vblang/issues/159), [#132](https://github.com/dotnet/vblang/issues/132), [#67](https://github.com/dotnet/vblang/issues/67), [#60](https://github.com/dotnet/vblang/issues/60)

**Feedback**
* Are side-effects really that bad?
* Syntax is weird because the right-hand side of the assignment can assign to the thing it's initializing; should it instead be forced to return a different value?
  * But how would we represent the case where a failed "Try" should result in throwing or early exit?
* Swift accomplishes this design with `if let` for positive cases and `guard let` for negative cases. Might that be a better design to explore?
