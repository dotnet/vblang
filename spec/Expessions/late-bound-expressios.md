^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Specification]("vblang/spec/VisualBasic-Specification.md") / [Expressions](Expressions.md) / **Late-Bound Expressions**

## Late-Bound Expressions

When the target of a member access expression or index expression is of type `Object`, the processing of the expression may be deferred until run time. Deferring processing this way is called *late binding*. Late binding allows `Object` variables to be used in a *typeless* way, where all resolution of members is based on the actual run-time type of the value in the variable. If strict semantics are specified by the compilation environment or by `Option Strict`, late binding causes a compile-time error. Non-public members are ignored when doing late-binding, including for the purposes of overload resolution. Note that, unlike the early-bound case, invoking or accessing a `Shared` member late-bound will cause the invocation target to be evaluated at run time. If the expression is an invocation expression for a member defined on `System.Object`, late binding will not take place.

In general, late-bound accesses are resolved at run time by looking up the identifier on the actual run-time type of the expression. If late-bound member lookup fails at run time, a `System.MissingMemberException` exception is thrown. Because late-bound member lookup is done solely off the run-time type of the associated target expression, an object's run-time type is never an interface. Therefore, it is impossible to access interface members in a late-bound member access expression.

The arguments to a late-bound member access are evaluated in the order they appear in the member access expression: not the order in which parameters are declared in the late-bound member. The following example illustrates this difference:

```vb
Class C
    Public Sub f(ByVal x As Integer, ByVal y As Integer)
    End Sub
End Class

Module Module1
    Sub Main()
        Console.Write("Early-bound: ")
        Dim c As C = New C
        c.f(y:=t("y"), x:=t("x"))

        Console.Write(vbCrLf & "Late-bound: ")
        Dim o As Object = New C
        o.f(y:=t("y"), x:=t("x"))
    End Sub

    Function t(ByVal s As String) As Integer
        Console.Write(s)
        Return 0
    End Function
End Module
```

This code displays:

```
Early-bound: xy
Late-bound: yx
```

Because late-bound overload resolution is done on the run-time type of the arguments, it is possible that an expression might produce different results based on whether it is evaluated at compile time or run time. The following example illustrates this difference:

```vb
Class Base
End Class

Class Derived
    Inherits Base
End Class

Module Test
    Sub F(b As Base)
        Console.WriteLine("F(Base)")
    End Sub

    Sub F(d As Derived)
        Console.WriteLine("F(Derived)")
    End Sub

    Sub Main()
        Dim b As Base = New Derived()
        Dim o As Object = b

        F(b)
        F(o)
    End Sub
End Module
```

This code displays:

```
F(Base)
F(Derived)
```