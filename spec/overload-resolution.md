# Overloaded Method Resolution

In practice, the rules for determining overload resolution are intended to find the overload that is "closest" to the actual arguments supplied. If there is a method whose parameter types match the argument types, then that method is obviously the closest. Barring that, one method is closer than another if all of its parameter types are narrower than (or the same as) the parameter types of the other method. If neither method's parameters are narrower than the other, then there is no way for to determine which method is closer to the arguments.

__Note.__ Overload resolution does not take into account the expected return type of the method. 

Also note that because of the named parameter syntax, the ordering of the actual and formal parameters may not be the same.

Given a method group, the most applicable method in the group for an argument list is determined using the following steps. If, after applying a particular step, no members remain in the set, then a compile-time error occurs. If only one member remains in the set, then that member is the most applicable member. The steps are:

1.  First, if no type arguments have been supplied, apply type inference to any methods which have type parameters. If type inference succeeds for a method, then the inferred type arguments are used for that particular method. If type inference fails for a method, then that method is eliminated from the set.

2.  Next, eliminate all members from the set that are inaccessible or not applicable (Section [Applicability To Argument List](overload-resolution.md#applicability-to-argument-list)) to the argument list

3.  Next, if one or more arguments are `AddressOf` or lambda expressions, then calculate the *delegate relaxation levels* for each such argument as below. If the worst (lowest) delegate relaxation level in `N` is worse than the lowest delegate relaxation level in `M`, then eliminate `N` from the set. The delegate relaxation levels are as follows:

    31. *Error delegate relaxation level* -- if the `AddressOf` or lambda cannot be converted to the delegate type.
  
    32. *Narrowing delegate relaxation of return type or parameters* -- if the argument is `AddressOf` or a lambda with a declared type and the conversion from its return type to the delegate return type is narrowing; or if the argument is a regular lambda and the conversion from any of its return expressions to the delegate return type is narrowing, or if the argument is an async lambda and the delegate return type is `Task(Of T)` and the conversion from any of its return expressions to `T` is narrowing; or if the argument is an iterator lambda and the delegate return type `IEnumerator(Of T)` or `IEnumerable(Of T)` and the conversion from any of its yield operands to `T` is narrowing.

    33. *Widening delegate relaxation to delegate without signature* -- if delegate type is `System.Delegate` or `System.MultiCastDelegate` or `System.Object`.

    34. *Drop return or arguments delegate relaxation* -- if the argument is `AddressOf` or a lambda with a declared return type and the delegate type lacks a return type; or if the argument is a lambda with one or more return expressions and the delegate type lacks a return type; or if the argument is `AddressOf` or lambda with no parameters and the delegate type has parameters.

    35. *Widening delegate relaxation of return type* -- if the argument is `AddressOf` or a lambda with a declared return type, and there is a widening conversion from its return type to that of the delegate; or if the argument is a regular lambda where the conversion from all return expressions to the delegate return type is widening or identity with at least one widening; or if the argument is an async lambda and the delegate is `Task(Of T)` or `Task` and the conversion from all return expressions to `T`/`Object` respectively is widening or identity with at least one widening; or if the argument is an iterator lambda and the delegate is `IEnumerator(Of T)` or `IEnumerable(Of T)` or `IEnumerator` or `IEnumerable` and the conversion from all return expressions to `T`/`Object` is widening or identity with at least one widening.

    36. *Identity delegate relaxation* -- if the argument is an `AddressOf` or a lambda which matches the delegate exactly, with no widening or narrowing or dropping of parameters or returns or yields.Next, if some members of the set do not requiring narrowing conversions to be applicable to any of the arguments, then eliminate all members that do. For example:

    ```vb
    Sub f(x As Object)
    End Sub

    Sub f(x As Short)
    End Sub

    Sub f(x As Short())
    End Sub

    f("5") ' picks the Object overload, since String->Short is narrowing
    f(5)   ' picks the Object overload, since Integer->Short is narrowing
    f({5}) ' picks the Object overload, since Integer->Short is narrowing
    f({})  ' a tie-breaker rule subsequent to [3] picks the Short() overload

    ```

4.  Next, elimination is done based on narrowing as follows. (Note that, if Option Strict is On, then all members that require narrowing have already been judged inapplicable (Section [Applicability To Argument List](overload-resolution.md#applicability-to-argument-list)) and removed by Step 2.)

    41. If some instance members of the set only require narrowing conversions where the argument expression type is `Object`, then eliminate all other members.
    42. If the set contains more than one member which requires narrowing only from `Object`, then the invocation target expression is reclassified as a late-bound method access (and an error is given if the type containing the method group is an interface, or if any of the applicable members were extension members).
    43. If there are any candidates that only require narrowing from numeric literals, then chose the most specific among all remaining candidates by the steps below. If the winner requires only narrowing from numeric literals, then it is picked as the result of overload resolution; otherwise it is an error.

    __Note.__ The justification for this rule is that if a program is loosely-typed (that is, most or all variables are declared as `Object`), overload resolution can be difficult when many conversions from `Object` are narrowing. Rather than have the overload resolution fail in many situations (requiring strong typing of the arguments to the method call), resolution the appropriate overloaded method to call is deferred until run time. This allows the loosely-typed call to succeed without additional casts. An unfortunate side-effect of this, however, is that performing the late-bound call requires casting the call target to `Object`. In the case of a structure value, this means that the value must be boxed to a temporary. If the method eventually called tries to change a field of the structure, this change will be lost once the method returns. Interfaces are excluded from this special rule because late binding always resolves against the members of the runtime class or structure type, which may have different names than the members of the interfaces they implement.

5.  Next, if any instance methods remain in the set which do not require narrowing, then eliminate all extension methods from the set. For example:

    ```vb
    Imports System.Runtime.CompilerServices

    Class C3
        Sub M1(d As Integer)
        End Sub
    End Class

    Module C3Extensions
        <Extension> _
        Sub M1(c3 As C3, c As Long)
        End Sub

        <Extension> _
        Sub M1(c3 As C3, c As Short)
        End Sub
    End Module

    Module Test
        Sub Main()
            Dim c As New C3()
            Dim sVal As Short = 10
            Dim lVal As Long = 20

            ' Calls C3.M1, since C3.M1 is applicable.
            c.M1(sVal)

            ' Calls C3Extensions.M1 since C3.M1 requires a narrowing conversion
            c.M1(lVal)
        End Sub
    End Module
    ```

    __Note.__ Extension methods are ignored if there are applicable instance methods to guarantee that adding an import (that might bring new extension methods into scope) will not cause a call on an existing instance method to rebind to an extension method. Given the broad scope of some extension methods (i.e. those defined on interfaces and/or type parameters), this is a safer approach to binding to extension methods.

6.  Next, if, given any two members of the set `M` and `N`, `M` is more *specific* (Section [Specificity of members/types given an argument list](overload-resolution.md#specificity-of-memberstypes-given-an-argument-list)) than `N` given the argument list, eliminate `N` from the set. If more than one member remains in the set and the remaining members are not equally specific given the argument list, a compile-time error results.

7.  Otherwise, given any two members of the set, `M` and `N`, apply the following tie-breaking rules, in order:

    71. If `M` does not have a ParamArray parameter but `N` does, or if both do but `M` passes fewer arguments into the ParamArray parameter than `N` does, then eliminate `N` from the set. For example:

        ```vb
        Module Test
            Sub F(a As Object, ParamArray b As Object())
                Console.WriteLine("F(Object, Object())")
            End Sub

            Sub F(a As Object, b As Object, ParamArray c As Object())
                Console.WriteLine("F(Object, Object, Object())")
            End Sub

           Sub G(Optional a As Object = Nothing)
              Console.WriteLine("G(Object)")
           End Sub

           Sub G(ParamArray a As Object())
              Console.WriteLine("G(Object())")
           End Sub    Sub Main()
                F(1)
                F(1, 2)
                F(1, 2, 3)
              G()
            End Sub
        End Module
        ```

        The above example produces the following output:

        ```
        F(Object, Object())
        F(Object, Object, Object())
        F(Object, Object, Object())
        G(Object)
        ```

        __Note.__ When a class declares a method with a paramarray parameter, it is not uncommon to also include some of the expanded forms as regular methods. By doing so it is possible to avoid the allocation of an array instance that occurs when an expanded form of a method with a paramarray parameter is invoked.

    72. If `M` is defined in a more derived type than `N`, eliminate `N` from the set. For example:

        ```vb
        Class Base
            Sub F(Of T, U)(x As T, y As U)
            End Sub
        End Class

        Class Derived
            Inherits Base

            Overloads Sub F(Of T, U)(x As U, y As T)
            End Sub
        End Class

        Module Test
            Sub Main()
                Dim d As New Derived()

                ' Calls Derived.F
                d.F(10, 10)
            End Sub
        End Module
        ```

        This rule also applies to the types that extension methods are defined on. For example:

        ```vb
        Imports System.Runtime.CompilerServices

        Class Base
        End Class

        Class Derived
            Inherits Base
        End Class

        Module BaseExt
            <Extension> _
            Sub M(b As Base, x As Integer)
            End Sub
        End Module

        Module DerivedExt
            <Extension> _
            Sub M(d As Derived, x As Integer)
            End Sub
        End Module

        Module Test
            Sub Main()
                Dim b As New Base()
                Dim d As New Derived()

                ' Calls BaseExt.M
                b.M(10)

                ' Calls DerivedExt.M 
                d.M(10)
            End Sub
        End Module
        ```

    73. If `M` and `N` are extension methods and the target type of `M` is a class or structure and the target type of `N` is an interface, eliminate `N` from the set. For example:

        ```vb
        Imports System.Runtime.CompilerServices

        Interface I1
        End Interface

        Class C1
            Implements I1
        End Class

        Module Ext1
            <Extension> _
            Sub M(i As I1, x As Integer)
            End Sub
        End Module

        Module Ext2
            <Extension> _
            Sub M(c As C1, y As Integer)
            End Sub
        End Module

        Module Test
            Sub Main()
                Dim c As New C1()

                ' Calls Ext2.M, because Ext1.M is hidden since it extends
                ' an interface.
                c.M(10)

                ' Calls Ext1.M
                CType(c, I1).M(10)
            End Sub
        End Module
        ```

    74. If `M` and `N` are extension methods, and the target type of `M` and `N` are identical after type parameter substitution, and the target type of `M` before type parameter substitution does not contain type parameters but the target type of `N` does, then has fewer type parameters than the target type of `N`, eliminate `N` from the set. For example:

        ```vb
        Imports System.Runtime.CompilerServices

        Module Module1
            Sub Main()
                Dim x As Integer = 1
                x.f(1) ' Calls first "f" extension method

                Dim y As New Dictionary(Of Integer, Integer)
                y.g(1) ' Ambiguity error
            End Sub

            <Extension()> Sub f(x As Integer, z As Integer)
            End Sub

            <Extension()> Sub f(Of T)(x As T, z As T)
            End Sub

            <Extension()> Sub g(Of T)(y As Dictionary(Of T, Integer), z As T)
            End Sub

            <Extension()> Sub g(Of T)(y As Dictionary(Of T, T), z As T)
            End Sub
        End Module
        ```

    75. Before type arguments have been substituted, if `M` is *less generic* (Section [Genericity](overload-resolution.md#genericity)) than `N`, eliminate `N` from the set.

    76. If `M` is not an extension method and `N` is, eliminate `N` from the set.

    77. If `M` and `N` are extension methods and `M` was found before `N` (Section [Extension Method Collection](expressions.md#extension-method-collection)), eliminate `N` from the set. For example:

        ```vb
        Imports System.Runtime.CompilerServices

        Class C1
        End Class

        Namespace N1
            Module N1C1Extensions
                <Extension> _
                Sub M1(c As C1, x As Integer)
                End Sub
            End Module
        End Namespace

        Namespace N1.N2
            Module N2C1Extensions
                <Extension> _
                Sub M1(c As C1, y As Integer)
                End Sub
            End Module
        End Namespace

        Namespace N1.N2.N3
            Module Test
                Sub Main()
                    Dim x As New C1()

                    ' Calls N2C1Extensions.M1
                    x.M1(10)
                End Sub
            End Module
        End Namespace
        ```

        If the extension methods were found in the same step, then those extension methods are ambiguous. The call may always be disambiguated using the name of the standard module containing the extension method and calling the extension method as if it was a regular member. For example:

        ```vb
        Imports System.Runtime.CompilerServices

        Class C1
        End Class

        Module C1ExtA
            <Extension> _
            Sub M(c As C1)
            End Sub
        End Module

        Module C1ExtB
            <Extension> _
            Sub M(c As C1)
            End Sub
        End Module

        Module Main
            Sub Test()
                Dim c As New C1()

                C1.M()               ' Ambiguous between C1ExtA.M and BExtB.M
                C1ExtA.M(c)          ' Calls C1ExtA.M
                C1ExtB.M(c)          ' Calls C1ExtB.M
            End Sub
        End Module
        ```

    78. If `M` and `N` both required type inference to produce type arguments, and `M` did not require determining the dominant type for any of its type arguments (i.e. each the type arguments inferred to a single type), but `N` did, eliminate `N` from the set.

        __Note.__ This rule ensures that overload resolution that succeeded in previous versions (where inferring multiple types for a type argument would cause an error), continue to produce the same results.

    79. If overload resolution is being done to resolve the target of a delegate-creation expression from an `AddressOf` expression, and both the delegate and `M` are functions while `N` is a subroutine, eliminate `N` from the set. Likewise, if both the delegate and `M` are subroutines, while `N` is a function, eliminate `N` from the set.

    710. If `M` did not use any optional parameter defaults in place of explicit arguments, but `N` did, then eliminate `N` from the set.

    711. Before type arguments have been substituted, if `M` has *greater depth of genericity* (Section [Genericity](overload-resolution.md#genericity)) than `N`, then eliminate `N` from the set.

8. Otherwise, the call is ambiguous and a compile-time error occurs.

#### Specificity of members/types given an argument list

A member `M` is considered *equally specific* as `N`, given an argument-list `A`, if their signatures are the same or if each parameter type in `M` is the same as the corresponding parameter type in `N`.

__Note.__ Two members can end up in a method group with the same signature due to extension methods. Two members can also be equally specific but not have the same signature due to type parameters or paramarray expansion.

A member `M` is considered *more specific* than `N` if their signatures are different and at least one parameter type in `M` is more specific than a parameter type in `N`, and no parameter type in `N` is more specific than a parameter type in `M`. Given a pair of parameters `Mj` and `Nj` that matches an argument `Aj`, the type of `Mj` is considered *more specific* than the type of `Nj` if one of the following conditions is true:

* There exists a widening conversion from the type of `Mj` to the type `Nj`. (__Note.__ Because parameters types are being compared without regard to the actual argument in this case, the widening conversion from constant expressions to a numeric type the value fits into is not considered in this case.)

* `Aj` is the literal `0`, `Mj` is a numeric type and `Nj` is an enumerated type. (__Note.__ This rule is necessary because the literal `0` widens to any enumerated type. Since an enumerated type widens to its underlying type, this means that overload resolution on `0` will, by default, prefer enumerated types over numeric types. We received a lot of feedback that this behavior was counterintuitive.)

* `Mj` and `Nj` are both numeric types, and `Mj` comes earlier than `Nj` in the list
`Byte`, `SByte`, `Short`, `UShort`, `Integer`, `UInteger`, `Long`, `ULong`, `Decimal`, `Single`, `Double`. (__Note.__ The rule about the numeric types is useful because the signed and unsigned numeric types of a particular size only have narrowing conversions between them. The above rule breaks the tie between the two types in favor of the more "natural" numeric type. This is particularly important when doing overload resolution on a type that widens to both the signed and unsigned numeric types of a particular size, for example, a numeric literal that fits into both.)

* `Mj` and `Nj` are delegate function types and the return type of `Mj` is more specific than the return type of `Nj` If `Aj` is classified as a lambda method, and `Mj` or `Nj` is `System.Linq.Expressions.Expression(Of T)`, then the type argument of the type (assuming it is a delegate type) is substituted for the type being compared.

* `Mj` is identical to the type of `Aj`, and `Nj` is not. (__Note.__ It is interesting to note that the previous rule differs slightly from C#, in that C# requires that the delegate function types have identical parameter lists before comparing return types, while Visual Basic does not.)

#### Genericity

A member `M` is determined to be *less generic* than a member `N` as follows:

1. If, for each pair of matching parameters `Mj` and `Nj`, `Mj` is less or equally generic than `Nj` with respect to type parameters on the method, and at least one `Mj` is less generic with respect to type parameters on the method.
2. Otherwise, if for each pair of matching parameters `Mj` and `Nj`, `Mj` is less or equally generic than `Nj` with respect to type parameters on the type, and at least one `Mj` is less generic with respect to type parameters on the type, then `M` is less generic than `N`.

A parameter `M` is considered to be equally generic to a parameter `N` if their types `Mt` and `Nt` both refer to type parameters or both don't refer to type parameters. `M` is considered to be less generic than `N` if `Mt` does not refer to a type parameter and `Nt` does.

For example:

```vb
Class C1(Of T)
    Sub S1(Of U)(x As U, y As T)
    End Sub

    Sub S1(Of U)(x As U, y As U)
    End Sub

    Sub S2(x As Integer, y As T)
    End Sub

    Sub S2(x As T, y As T)
    End Sub
End Class

Module Test
    Sub Main()
        Dim x As C1(Of Integer) = New C1(Of Integer)

        x.S1(10, 10)    ' Calls S1(U, T)
        x.S2(10, 10)    ' Calls S2(Integer, T)
    End Sub
End Module
```

Extension method type parameters that were fixed during currying are considered type parameters on the type, not type parameters on the method. For example:

```vb
Imports System.Runtime.CompilerServices

Module Ext1
    <Extension> _
    Sub M1(Of T, U)(x As T, y As U, z As U)
    End Sub
End Module

Module Ext2
    <Extension> _
    Sub M1(Of T, U)(x As T, y As U, z As T)
    End Sub
End Module

Module Test
    Sub Main()
        Dim i As Integer = 10

        i.M1(10, 10)
    End Sub
End Module
```

#### Depth of genericity

A member `M` is determined to have *greater depth of genericity* than a member `N` if, for each pair of matching parameters  `Mj` and `Nj`, `Mj` has greater or equal *depth of genericity* than `Nj`, and at least one `Mj` is has greater depth of genericity. Depth of genericity is defined as follows:

* Anything other than a type parameter has greater depth of genericity than a type parameter;

* Recursively, a constructed type has greater depth of genericity than another constructed type (with the same number of type arguments) if at least one type argument has greater depth of genericity and no type argument has less depth than the corresponding type argument in the other.

* An array type has greater depth of genericity than another array type (with the same number of dimensions) if the element type of the first has greater depth of genericity than the element type of the second.

For example:

```vb
Module Test

    Sub f(Of T)(x As Task(Of T))
    End Sub

    Sub f(Of T)(x As T)
    End Sub

    Sub Main()
        Dim x As Task(Of Integer) = Nothing
        f(x)            ' Calls the first overload
    End Sub
End Module
```

### Applicability To Argument List

A method is *applicable* to a set of type arguments, positional arguments, and named arguments if the method can be invoked using the argument lists. The argument lists are matched against the parameter lists as follows:

1. First, match each positional argument in order to the list of method parameters. If there are more positional arguments than parameters and the last parameter is not a paramarray, the method is not applicable. Otherwise, the paramarray parameter is expanded with parameters of the paramarray element type to match the number of positional arguments. If a positional argument is omitted that would go into a paramarray, the method is not applicable.
2. Next, match each named argument to a parameter with the given name. If one of the named arguments fails to match, matches a paramarray parameter, or matches an argument already matched with another positional or named argument, the method is not applicable.
3. Next, if type arguments have been specified, they are matched against the type parameter list . If the two lists do not have the same number of elements, the method is not applicable, unless the type argument list is empty. If the type argument list is empty, type inference is used to try and infer the type argument list. If type inferencing fails, the method is not applicable. Otherwise, the type arguments are filled in the place of the type parameters in the signature.If parameters that have not been matched are not optional, the method is not applicable.
4. If the argument expressions are not implicitly convertible to the types of the parameters they match, then the method is not applicable.
5. If a parameter is ByRef, and there is not an implicit conversion from the type of the parameter to the type of the argument, then the method is not applicable.
6. If type arguments violate the method's constraints (including the inferred type arguments from Step 3), the method is not applicable. For example:

```vb
Module Module1
    Sub Main()
        f(Of Integer)(New Exception)
        ' picks the first overload (narrowing),
        ' since the second overload (widening) violates constraints 
    End Sub

    Sub f(Of T)(x As IComparable)
    End Sub

    Sub f(Of T As Class)(x As Object)
    End Sub
End Module
```

If a single argument expression matches a paramarray parameter and the type of the argument expression is convertible to both the type of the paramarray parameter and the paramarray element type, the method is applicable in both its expanded and unexpanded forms, with two exceptions. If the conversion from the type of the argument expression to the paramarray type is narrowing, then the method is only applicable in its expanded form. If the argument expression is the literal `Nothing`, then the method is only applicable in its unexpanded form. For example:

```vb
Module Test
    Sub F(ParamArray a As Object())
        Dim o As Object

        For Each o In a
            Console.Write(o.GetType().FullName)
            Console.Write(" ")
        Next o
        Console.WriteLine()
    End Sub

    Sub Main()
        Dim a As Object() = { 1, "Hello", 123.456 }
        Dim o As Object = a

        F(a)
        F(CType(a, Object))
        F(o)
        F(CType(o, Object()))
    End Sub
End Module
```

The above example produces the following output:

```
System.Int32 System.String System.Double
System.Object[]
System.Object[]
System.Int32 System.String System.Double
```

In the first and last invocations of `F`, the normal form of `F` is applicable because a widening conversion exists from the argument type to the parameter type (both are of type `Object()`), and the argument is passed as a regular value parameter. In the second and third invocations, the normal form of `F` is not applicable because no widening conversion exists from the argument type to the parameter type (conversions from `Object` to `Object()` are narrowing). However, the expanded form of `F` is applicable, and a one-element `Object()` is created by the invocation. The single element of the array is initialized with the given argument value (which itself is a reference to an `Object()`).

### Passing Arguments, and Picking Arguments for Optional Parameters

If a parameter is a value parameter, the matching argument expression must be classified as a value. The value is converted to the type of the parameter and passed in as the parameter at run time. If the parameter is a reference parameter and the matching argument expression is classified as a variable whose type is the same as the parameter, then a reference to the variable is passed in as the parameter at run time.

Otherwise, if the matching argument expression is classified as a variable, value, or property access, then a temporary variable of the type of the parameter is allocated. Before the method invocation at run time, the argument expression is reclassified as a value, converted to the type of the parameter, and assigned to the temporary variable. Then a reference to the temporary variable is passed in as the parameter. After the method invocation is evaluated, if the argument expression is classified as a variable or property access, the temporary variable is assigned to the variable expression or the property access expression. If the property access expression has no `Set` accessor, then the assignment is not performed.

For optional parameters where an argument has not been provided, the compiler picks arguments as described below. In all cases it tests against the parameter type after generic type substitution.

* If the optional parameter has the attribute `System.Runtime.CompilerServices.CallerLineNumber`, and the invocation is from a location in source code, and a numeric literal representing that location's line number has an intrinsic conversion to the parameter type, then the numeric literal is used. If the invocation spans multiple lines, then the choice of which line to use is implementation-dependent.

* If the optional parameter has the attribute `System.Runtime.CompilerServices.CallerFilePath`, and the invocation is from a location in source code, and a string literal representing that location's file path has an intrinsic conversion to the parameter type, then the string literal is used. The format of the file path is implementation-dependent.

* If the optional parameter has the attribute `System.Runtime.CompilerServices.CallerMemberName`, and the invocation is within the body of a type member or in an attribute applied to any part of that type member, and a string literal representing that member name has an intrinsic conversion to the parameter type, then the string literal is used. For invocations that are part of property accessors or custom event handlers, then the member name used is that of the property or event itself. For invocations that are part of an operator or constructor, then an implementation-specific name is used.

If none of the above apply, then the optional parameter's default value is used (or `Nothing` if no default value is supplied). And if more than one of the above apply, then the choice of which to use is implementation-dependent.

The `CallerLineNumber` and `CallerFilePath` attributes are useful for logging. The `CallerMemberName` is useful for implementing `INotifyPropertyChanged`. Here are examples.

```vb
Sub Log(msg As String,
        <CallerFilePath> Optional file As String = Nothing,
        <CallerLineNumber> Optional line As Integer? = Nothing)
    Console.WriteLine("{0}:{1} - {2}", file, line, msg)
End Sub

WriteOnly Property p As Integer
    Set(value As Integer)
        Notify(_p, value)
    End Set
End Property

Private _p As Integer

Sub Notify(Of T As IEquatable(Of T))(ByRef v1 As T, v2 As T,
        <CallerMemberName> Optional prop As String = Nothing)
    If v1 IsNot Nothing AndAlso v1.Equals(v2) Then Return
    If v1 Is Nothing AndAlso v2 Is Nothing Then Return
    v1 = v2
    RaiseEvent PropertyChanged(Me, New PropertyChangedEventArgs(prop))
End Sub
```

In addition to the optional parameters above, Microsoft Visual Basic also recognizes some additional optional parameters if they are imported from metadata (i.e. from a DLL reference):

* Upon importing from metadata, Visual Basic also treats the parameter `<Optional>` as indicative that the parameter is optional: in this way it is possible to import a declaration which has an optional parameter but no default value, even though this can't be expressed using the `Optional` keyword.

* If the optional parameter has the attribute `Microsoft.VisualBasic.CompilerServices.OptionCompareAttribute`, and the numeric literal 1 or 0 has a conversion to the parameter type, then the compiler uses as argument either the literal 1 if `Option Compare Text` is in effect, or the literal 0 if `Optional Compare Binary` is in effect.

* If the optional parameter has the attribute `System.Runtime.CompilerServices.IDispatchConstantAttribute`, and it has type `Object`, and it does not specify a default value, then the compiler uses the argument `New System.Runtime.InteropServices.DispatchWrapper(Nothing)`.

* If the optional parameter has the attribute `System.Runtime.CompilerServices.IUnknownConstantAttribute`, and it has type `Object`, and it does not specify a default value, then the compiler uses the argument `New System.Runtime.InteropServices.UnknownWrapper(Nothing)`.

* If the optional parameter has type `Object`, and it does not specify a default value, then the compiler uses the argument `System.Reflection.Missing.Value`.

### Conditional Methods

If the target method to which an invocation expression refers is a subroutine that is not a member of an interface and if the method has one or more `System.Diagnostics.ConditionalAttribute` attributes, evaluation of the expression depends on the conditional compilation constants defined at that point in the source file. Each instance of the attribute specifies a string, which names a conditional compilation constant. Each conditional compilation constant is evaluated as if it were part of a conditional compilation statement. If the constant evaluates to `True`, the expression is evaluated normally at run time. If the constant evaluates to `False`, the expression is not evaluated at all.

When looking for the attribute, the most derived declaration of an overridable method is checked.

__Note.__ The attribute is not valid on functions or interface methods and is ignored if specified on either kind of method. Thus, conditional methods will only appear in invocation statements.

### Type Argument Inference

When a method with type parameters is called without specifying type arguments, *type argument inference* is used to try and infer type arguments for the call. This allows a more natural syntax to be used for calling a method with type parameters when the type arguments can be trivially inferred. For example, given the following method declaration:

```vb
Module Util
    Function Choose(Of T)(b As Boolean, first As T, second As T) As T
        If b Then
            Return first
        Else
            Return second
        End If
    End Function
End Class
```

it is possible to invoke the `Choose` method without explicitly specifying a type argument:

```vb
' calls Choose(Of Integer)
Dim i As Integer = Util.Choose(True, 5, 213)
' calls Choose(Of String)
Dim s As String = Util.Choose(False, "a", "b") 
```

Through type argument inference, the type arguments `Integer` and `String` are determined from the arguments to the method.

Type argument inference occurs *before* expression reclassification is performed on lambda methods or method pointers in the argument list, since reclassification of those two kinds of expressions may require the type of the parameter to be known.  Given a set of arguments `A1,...,An`, a set of matching parameters `P1,...,Pn` and a set of method type parameters `T1,...,Tn`, the dependencies between the arguments and method type parameters are first collected as follows:

* If `An` is the `Nothing` literal, no dependencies are generated.

* If `An` is a lambda method and the type of `Pn` is a constructed delegate type or `System.Linq.Expressions.Expression(Of T)`, where `T` is a constructed delegate type,

* If the type of a lambda method parameter will be inferred from the type of the corresponding parameter `Pn`, and the type of the parameter depends on a method type parameter `Tn`, then `An` has a dependency on `Tn`.

* If the type of a lambda method parameter is specified and the type of the corresponding parameter `Pn` depends on a method type parameter `Tn`, then `Tn` has a dependency on `An`.

* If the return type of `Pn` depends on a method type parameter `Tn`, then `Tn` has a dependency on `An`.

* If `An` is a method pointer and the type of `Pn` is a constructed delegate type,

* If the return type of `Pn` depends on a method type parameter `Tn`, then `Tn` has a dependency on `An`.

* If `Pn` is a constructed type and the type of `Pn` depends on a method type parameter `Tn`, then `Tn` has a dependency on `An`.

* Otherwise, no dependency is generated.

After collecting dependencies, any arguments that have no dependencies are eliminated. If any method type parameters have no outgoing dependencies (i.e. the method type parameter does not depend on an argument), then type inference fails. Otherwise, the remaining arguments and method type parameters are grouped into strongly connected components. A strongly connected component is a set of arguments and method type parameters, where any element in the component is reachable via dependencies on other elements.

The strongly connected components are then topologically sorted and processed in topological order:

* If the strongly typed component contains only one element,

  * If the element has already been marked complete, skip it.

  * If the element is an argument, then add type hints from the argument to the method type parameters that depend on it and mark the element as complete. If the argument is a lambda method with parameters that still need inferred types, then infer `Object` for the types of those parameters.

  * If the element is a method type parameter, then infer the method type parameter to be the dominant type among the argument type hints and mark the element as complete. If a type hint has an array element restriction on it, then only conversions that are valid between arrays of the given type are considered (i.e. covariant and intrinsic array conversions). If a type hint has a generic argument restriction on it, then only identity conversions are considered. If no dominant type can be chosen, inference fails. If any lambda method argument types depend on this method type parameter, the type is propagated to the lambda method.

* If the strongly typed component contains more than one element, then the component contains a cycle.

  * For each method type parameter that is an element in the component, if the method type parameter depends on an argument that is not marked complete, convert that dependency into an assertion that will be checked at the end of the inference process.

  * Restart the inference process at the point at which the strongly typed components were determined.

If type inference succeeds for all of the method type parameters, then any dependencies that were changed into assertions are checked. An assertion succeeds if the type of the argument is implicitly convertible to the inferred type of the method type parameter. If an assertion fails, then type argument inference fails.

Given an argument type `Ta` for an argument `A` and a parameter type `Tp` for a parameter `P`, type hints are generated as follows:

* If `Tp` does not involve any method type parameters then no hints are generated.

* If `Tp` and `Ta` are array types of the same rank, then replace `Ta` and `Tp` with the element types of `Ta` and `Tp` and restart this process with an array element restriction.

* If `Tp` is a method type parameter, then `Ta` is added as a type hint with the current restriction, if any.

* If `A` is a lambda method and `Tp` is a constructed delegate type or `System.Linq.Expressions.Expression(Of T)`, where `T` is a constructed delegate type, for each lambda method parameter type `TL` and corresponding delegate parameter type `TD`, replace `Ta` with `TL` and `Tp` with `TD` and restart the process with no restriction. Then, replace `Ta` with the return type of the lambda method and:

  * if `A` is a regular lambda method, replace `Tp` with the return type of the delegate type;
  * if `A` is an async lambda method and the return type of the delegate type has form `Task(Of T)` for some `T`, replace `Tp` with that `T`;
  * if `A` is an iterator lambda method and the return type of the delegate type has form `IEnumerator(Of T)` or `IEnumerable(Of T)` for some `T`, replace `Tp` with that `T`.
  * Next, restart the process with no restriction.

* If `A` is a method pointer and `Tp` is a constructed delegate type, use the parameter types of `Tp` to determine which method pointed is most applicable to `Tp`. If there is a method that is most applicable, replace `Ta` with the return type of the method and `Tp` with the return type of the delegate type and restart the process with no restriction.

* Otherwise, `Tp` must be a constructed type. Given `TG`, the generic type of `Tp`,

  * If `Ta` is `TG`, inherits from `TG`, or implements the type `TG` exactly once, then for each matching type argument `Tax` from `Ta` and `Tpx` from `Tp`, replace `Ta` with `Tax` and `Tp` with `Tpx` and restart the process with a generic argument restriction.

  * Otherwise, type inference fails for the generic method.

The success of type inference does not, in and of itself, guarantee that the method is applicable.
