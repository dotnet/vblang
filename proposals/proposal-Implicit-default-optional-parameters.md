# Implicit Default Optional Parameters

* [x] Proposed
* [x] Prototype: [Complete](https://github.com/AdamSpeight2008/master_Feature_ImplicitDefaultOptionalParameter)
* [ ] Implementation: [In Progress](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

## Summary
[summary]: #summary

One para explanation of the feature.

## Motivation
[motivation]: #motivation

Simplifies the type of the more common usage case, that of using the default value of the associated type as the default to value of the parameter.
Eg `Foo(Optional size As String = Nothing)`

## Detailed design
[design]: #detailed-design

The grammar definition for an `Optional Parameter` is something similar to this.

```
OptionalParameter ::= "Optional" ParameterName Typing? DefaultToValue?
           Typing ::= "As" TypeIdentifier
   DefaultToValue ::= "=" ( "Nothing" | ConstantValue )
```

**Examples**

```vb.net
Foo( Optional arg0 )                     ' --> Foo( Optional arg0 As Object = Nothing )
Foo( Optional arg1 As String )           ' --> Foo( Optional arg1 As String = Nothing )
Foo( Optional arg2 As String = Nothing ) ' --> Foo( Optional arg2 As String = Nothing )
Foo( Optional arg3 = "" )                ' --> Foo( Optional arg3 As String = "" )
Foo( Optional arg4 As Integer )          ' --> Foo( Optional arg4 As Integer = Nothing )
Foo( Optional arg5 As Integer = 0 )      ' --> Foo( Optional arg5 As Integer = 0 )
Foo( Optional arg6 As Integer = 1 )      ' --> Foo( Optional arg6 As Integer = 1 )
Foo( Optional arg7 = 7 )                 ' --> Foo( Optional arg7 As Integer = 7 )
```



## Potential Usage
[potential]: #potential
Using the following code authored by @AnthonyDGreen
```VB.NET
Imports System.Console
Imports System.IO

Module Program

    Sub Main()
        Dim log = New StringWriter()

        Dim nothingDefaultCount = 0,
            nonNothingDefaultCount = 0

        For Each filename In Directory.EnumerateFiles("<roslyn-repo-path>", "*.vb", SearchOption.AllDirectories)
            Dim tree = VisualBasicSyntaxTree.ParseText(File.ReadAllText(filename))

            Dim root = tree.GetRoot()

            For Each decl In root.DescendantNodes.OfType(Of ParameterSyntax)
                If decl.Default Is Nothing Then Continue For

                If decl.Default.Value.IsKind(SyntaxKind.NothingLiteralExpression) OrElse
                   decl.Default.Value.IsKind(SyntaxKind.FalseLiteralExpression) OrElse
                   decl.Default.Value.ToString() = "0" _
                Then
                    nothingDefaultCount += 1
                Else
                    nonNothingDefaultCount += 1

                    Dim text = decl.ToString()
                    WriteLine(text)
                    log.WriteLine(text)
                End If
            Next
        Next

        WriteLine(nothingDefaultCount)
        WriteLine(nonNothingDefaultCount)

        My.Computer.Clipboard.SetText(log.ToString())
    End Sub
End Module
```
Running it on my roslyn repo clone it reports that of all the optional parameter declarations in VB in Roslyn 2229 out of 2612 of them, or > 85% use the default value of the parameter type as the default value of the parameter. So your suggestion would clean up 85% of the uses of Optional in Roslyn. Sounds like value to me!

## Invariants
[invariants]: #invariants

The following invariants must be preserved to maintain compatibility with legacy / preexisting code.

`CallerInfoAttributes`

**Legacy**
```VB.net
Imports System.Runtime.CompilerServices
Module Module1

    Sub Main()
        Dim r = Foo()
        Console.WriteLine(r)
    End Sub

    Function Foo(<CallerLineNumber> Optional x As Integer = 0) As Integer
        Console.WriteLine(x)
        Return x
    End Function

End Module
```
**With Feature**
```vb.net
Imports System.Runtime.CompilerServices
Module Module1

    Sub Main()
        Dim r = Foo()
        Console.WriteLine(r)
    End Sub

    Function Foo(<CallerLineNumber> Optional x As Integer) As Integer
        Console.WriteLine(x)
        Return x
    End Function

End Module
```
**Output**
```
R=5
X=5
```




## Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

## Alternatives
[alternatives]: #alternatives

What other designs have been considered? What is the impact of not doing this?

## Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD?