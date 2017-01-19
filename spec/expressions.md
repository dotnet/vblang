# Expressions

An expression is a sequence of operators and operands that specifies a computation of a value, or that designates a variable or constant. This chapter defines the syntax, order of evaluation of operands and operators, and meaning of expressions.

```antlr
Expression
    : SimpleExpression
    | TypeExpression
    | MemberAccessExpression
    | DictionaryAccessExpression
    | InvocationExpression
    | IndexExpression
    | NewExpression
    | CastExpression
    | OperatorExpression
    | ConditionalExpression
    | LambdaExpression
    | QueryExpression
    | XMLLiteralExpression
    | XMLMemberAccessExpression
    ;
```

## Expression Classifications

Every expression is classified as one of the following:

* *A value.* Every value has an associated type.

* *A variable.* Every variable has an associated type, namely the declared type of the variable.

* *A namespace.* An expression with this classification can only appear as the left side of a member access. In any other context, an expression classified as a namespace causes a compile-time error.

* *A type.* An expression with this classification can only appear as the left side of a member access. In any other context, an expression classified as a type causes a compile-time error.

* *A method group,* which is a set of methods overloaded on the same name. A method group may have an associated target expression and an associated type argument list.

* *A method pointer,* which represents the location of a method. A method pointer may have an associated target expression and an associated type argument list.

* *A lambda method,* which is an anonymous method.

* *A property group,* which is a set of properties overloaded on the same name. A property group may have an associated target expression.

* *A property access.* Every property access has an associated type, namely the type of the property. A property access may have an associated target expression.

* *A late-bound access,* which represents a method or property access deferred until run-time. A late-bound access may have an associated target expression and an associated type argument list. The type of a late-bound access is always `Object`.

* *An event access.* Every event access has an associated type, namely the type of the event. An event access may have an associated target expression. An event access may appear as the first argument of the `RaiseEvent`, `AddHandler`, and `RemoveHandler` statements. In any other context, an expression classified as an event access causes a compile-time error.

* *An array literal,* which represents the initial values of an array whose type has not yet been determined.

* *Void.* This occurs when the expression is an invocation of a subroutine, or an await operator expression with no result. An expression classified as void is only valid in the context of an invocation statement or an await statement.

* *A default value.* Only the literal `Nothing` produces this classification.

The final result of an expression is usually a value or a variable, with the other categories of expressions functioning as intermediate values that are only permitted in certain contexts.

Note that expressions whose type is a type parameter can be used in statements and expressions that require the type of an expression to have certain characteristics (such as being a reference type, value type, deriving from some type, etc.) if the constraints imposed on the type parameter satisfy those characteristics.

### Expression Reclassification

Normally, when an expression is used in a context that requires a classification different from that of the expression, a compile-time error occurs -- for example, attempting to assign a value to a literal. However, in many cases it is possible to change an expression's classification through the process of *reclassification*.

If reclassification succeeds, then the reclassification is judged as widening or narrowing. Unless otherwise noted, all the reclassifications in this list are widening.

The following types of expressions can be reclassified:

* A variable can be reclassified as a value. The value stored in the variable is fetched.

* A method group can be reclassified as a value. The method group expression is interpreted as an invocation expression with the associated target expression and type parameter list, and empty parentheses (that is, `f` is interpreted as `f()` and `f(Of Integer)` is interpreted as `f(Of Integer)()`). This reclassification may result in the expression being further reclassified as void.

* A method pointer can be reclassified as a value. This reclassification can only occur in the context of a conversion where the target type is known. The method pointer expression is interpreted as the argument to a delegate instantiation expression of the appropriate type with the associated type argument list. For example:
    
    ```vb
    Delegate Sub D(i As Integer)
    
    Module Test
        Sub F(i As Integer)
        End Sub
    
        Sub Main()
            Dim del As D
    
            ' The next two lines are equivalent.
            del = AddressOf F
            del = New D(AddressOf F)
        End Sub
    End Module
    ```

* A lambda method can be reclassified as a value. If the reclassification occurs in the context of a conversion where the target type is known, then one of two reclassifications can occur:
    
  1. If the target type is a delegate type, the lambda method is interpreted as the argument to a delegate-construction expression of the appropriate type.
    
  2. If the target type is `System.Linq.Expressions.Expression(Of T)`, and `T` is a delegate type, then the lambda method is interpreted as if it was being used in delegate-construction expression for `T` and then converted to an expression tree.
    
  An async or iterator lambda method may only be interpreted as the argument to a delegate-construction expression, if the delegate has no ByRef parameters.
    
  If conversion from any of the delegate's parameter types to the corresponding lambda parameter types is a narrowing conversion, then the reclassification is judged as narrowing; otherwise it is widening.
    
  __Note.__ The exact translation between lambda methods and expression trees may not be fixed between versions of the compiler and is beyond the scope of this specification. For Microsoft Visual Basic 11.0, all lambda expressions may be converted to expression trees subject to the following restrictions: (1) 1.  Only single-line lambda expressions without ByRef parameters may be converted to expression trees. Of the single-line `Sub` lambdas, only invocation statements may be converted to expression trees. (2) Anonymous type expressions cannot be converted to expression trees if an earlier field initializer is used to initialize a subsequent field initializer, e.g. `New With {.a=1, .b=.a}`. (3) Object initializer expressions cannot be converted to expression trees if a member of the current object being initialized is used in one of the field initializers, e.g. `New C1 With {.a=1, .b=.Method1()}`. (4) Multi-dimensional array creation expressions can only be converted to expression trees if they declare their element type explicitly. (5) Late-binding expressions cannot be converted to expression trees. (6) When a variable or field is passed ByRef to an invocation expression but does not have exactly the same type as the ByRef parameter, or when a property is passed ByRef, normal VB semantics are that a copy of the argument is passed ByRef and its final value is then copied back into the variable or field or property. In expression trees, the copy-back does not happen. (7) All these restrictions apply to nested lambda expressions as well.
    
  If the target type is not known, then the lambda method is interpreted as the argument to a delegate instantiation expression of an anonymous delegate type with the same signature of the lambda method. If strict semantics are being used and the type of any of the parameters are omitted, a compile-time error occurs; otherwise, `Object` is substituted for any missing parameter type. For example:
    
  ```vb
  Module Test
      Sub Main()
          ' Type of x will be equivalent to Func(Of Object, Object, Object)
          Dim x = Function(a, b) a + b
  
          ' Type of y will be equivalent to Action(Of Object, Object)
          Dim y = Sub(a, b) Console.WriteLine(a + b)
      End Sub
  End Module
  ```

* A property group can be reclassified as a property access. The property group expression is interpreted as an index expression with empty parentheses (that is, `f` is interpreted as `f()`).

* A property access can be reclassified as a value. The property access expression is interpreted as an invocation expression of the `Get` accessor of the property. If the property has no getter, then a compile-time error occurs.

* A late-bound access can be reclassified as a late-bound method or late-bound property access. In a situation where a late-bound access can be reclassified both as a method access and as a property access, reclassification to a property access is preferred.

* A late-bound access can be reclassified as a value.

* An array literal can be reclassified as a value. The type of the value is determined as follows:

  1. If the reclassification occurs in the context of a conversion where the target type is known and the target type is an array type, then the array literal is reclassified as a value of type T(). If the target type is `System.Collections.Generic.IList(Of T)`, `IReadOnlyList(Of T)`, `ICollection(Of T)`, `IReadOnlyCollection(Of T)`, or `IEnumerable(Of T)`, and the array literal has one level of nesting, then the array literal is reclassified as a value of type `T()`.

  2. Otherwise, the array literal is reclassified to a value whose type is an array of rank equal to the level of nesting is used, with element type determined by the dominant type of the elements in the initializer; if no dominant type can be determined, `Object` is used. For example:

     ```vb
     ' x Is GetType(Double(,,))
     Dim x = { { { 1, 2.0 }, { 3, 4 } }, { { 5, 6 }, { 7, 8 } } }.GetType()
        
     ' y Is GetType(Integer())
     Dim y = { 1, 2, 3 }.GetType()
        
     ' z Is GetType(Object())
     Dim z = { 1, "2" }.GetType()
        
     ' Error: Inconsistent nesting
     Dim a = { { 10 }, { 20, 30 } }.GetType()
     ```

  __Note.__ There is a slight change in behavior between version 9.0 and version 10.0 of the language. Prior to 10.0, array element initializers did not affect local variable type inference and now they do. So `Dim a() = { 1, 2, 3 }` would have inferred `Object()` as the type of `a` in version 9.0 of the language and `Integer()` in version 10.0.

  The reclassification then reinterprets the array literal as an array-creation expression. So the examples:

  ```vb
  Dim x As Double = { 1, 2, 3, 4 }
  Dim y = { "a", "b" }
  ```

  are equivalent to:

  ```vb
  Dim x As Double = New Double() { 1, 2, 3, 4 }
  Dim y = New String() { "a", "b" }
  ```

  The reclassification is judged as narrowing if any conversion from an element expression to the array element type is narrowing; otherwise it is judged as widening.

* The default value `Nothing` can be reclassified as a value. In a context where the target type is known, the result is the default value of the target type. In a context where the target type is not known, the result is a null value of type `Object`.

A namespace expression, type expression, event access expression, or void expression cannot be reclassified. Multiple reclassifications can be done at the same time. For example:

```vb
Module Test
    Sub F(i As Integer)
    End Sub

    ReadOnly Property P() As Integer
        Get
        End Get
    End Sub

    Sub Main()
        F(P)
    End Property
End Module
```

In this case, the property group expression `P` is first reclassified from a property group to a property access and then reclassified from a property access to a value. The fewest number of reclassifications are performed to reach a valid classification in the context.

## Constant Expressions

A *constant expression* is an expression whose value can be fully evaluated at compile time.

```antlr
ConstantExpression
    : Expression
    ;
```

The type of a constant expression can be `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Char`, `Single`, `Double`, `Decimal`, `Date`, `Boolean`, `String`, `Object`, or any enumeration type. The following constructs are permitted in constant expressions:

* Literals (including `Nothing`).

* References to constant type members or constant locals.

* References to members of enumeration types.

* Parenthesized subexpressions.

* Coercion expressions, provided the target type is one of the types listed above. Coercions to and from `String` are an exception to this rule and are only allowed on null values because `String` conversions are always done in the current culture of the execution environment at run time. Note that constant coercion expressions can only ever use intrinsic conversions.

* The `+`, `-` and `Not` unary operators, provided the operand and result is of a type listed above.

* The `+`, `-`, `*`, `^`, `Mod`, `/`, `\`, `<<`, `>>`, `&`, `And`, `Or`, `Xor`, `AndAlso`, `OrElse`, `=`, `<`, `>`, `<>`, `<=`, and `=>` binary operators, provided each operand and result is of a type listed above.

* The conditional operator If, provided each operand and result is of a type listed above.

* The following run-time functions: `Microsoft.VisualBasic.Strings.ChrW`; `Microsoft.VisualBasic.Strings.Chr` if the constant value is between 0 and 128; `Microsoft.VisualBasic.Strings.AscW` if the constant string is not empty; `Microsoft.VisualBasic.Strings.Asc` if the constant string is not empty.

The following constructs are *not* permitted in constant expressions:

* Implicit binding through a `With` context.

Constant expressions of an integral type (`ULong`, `Long`, `UInteger`, `Integer`, `UShort`, `Short`, `SByte`, or `Byte`) can be implicitly converted to a narrower integral type, and constant expressions of type `Double` can be implicitly converted to `Single`, provided the value of the constant expression is within the range of the destination type. These narrowing conversions are allowed regardless of whether permissive or strict semantics are being used.


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

## Simple Expressions

Simple expressions are literals, parenthesized expressions, instance expressions, or simple name expressions.

```antlr
SimpleExpression
    : LiteralExpression
    | ParenthesizedExpression
    | InstanceExpression
    | SimpleNameExpression
    | AddressOfExpression
    ;
```

### Literal Expressions

Literal expressions evaluate to the value represented by the literal. A literal expression is classified as a value, except for the literal `Nothing`, which is classified as a default value.

```antlr
LiteralExpression
    : Literal
    ;
```

### Parenthesized Expressions

A parenthesized expression consists of an expression enclosed in parentheses. A parenthesized expression is classified as a value, and the enclosed expression must be classified as a value. A parenthesized expression evaluates to the value of the expression within the parentheses.

```antlr
ParenthesizedExpression
    : OpenParenthesis Expression CloseParenthesis
    ;
```

### Instance Expressions

An *instance expression* is the keyword `Me`. It may only be used within the body of a non-shared method, constructor, or property accessor. It is classified as a value. The keyword `Me` represents the instance of the type containing the method or property accessor being executed. If a constructor explicitly invokes another constructor (Section [Constructors](type-members.md#constructors)), `Me` cannot be used until after that constructor call, because the instance has not yet been constructed.

```antlr
InstanceExpression
    : 'Me'
    ;
```

### Simple Name Expressions

A *simple name expression* consists of a single identifier followed by an optional type argument list.

```antlr
SimpleNameExpression
    : Identifier ( OpenParenthesis 'Of' TypeArgumentList CloseParenthesis )?
    ;
```

The name is resolved and classified by the following "simple name resolution rules":

1.  Starting with the immediately enclosing block and continuing with each enclosing outer block (if any), if the identifier matches the name of a local variable, static variable, constant local, method type parameter, or parameter, then the identifier refers to the matching entity.

    If the identifier matches a local variable, static variable, or constant local and a type argument list was provided, a compile-time error occurs. If the identifier matches a method type parameter and a type argument list was provided, no match occurs and resolution continues. If the identifier matches a local variable, the local variable matched is the implicit function or `Get` accessor return local variable, and the expression is part of an invocation expression, invocation statement, or an `AddressOf` expression, then no match occurs and resolution continues.

    The expression is classified as a variable if it is a local variable, static variable, or parameter. The expression is classified as a type if it is a method type parameter. The expression is classified as a value if it is a constant local.

2.  For each nested type containing the expression, starting from the innermost and going to the outermost, if a lookup of the identifier in the type produces a match with an accessible member:

    21. If the matching type member is a type parameter, then the result is classified as a type and is the matching type parameter. If a type argument list was provided, no match occurs and resolution continues.
    22. Otherwise, if the type is the immediately enclosing type and the lookup identifies a non-shared type member, then the result is the same as a member access of the form `Me.E(Of A)`, where `E` is the identifier and `A` is the type argument list, if any.
    23. Otherwise, the result is exactly the same as a member access of the form `T.E(Of A)`, where `T` is the type containing the matching member, `E` is the identifier, and `A` is the type argument list, if any. In this case, it is an error for the identifier to refer to a non-shared member.

3.  For each nested namespace, starting from the innermost and going to the outermost namespace, do the following:

    31. If the namespace contains an accessible type with the given name and has the same number of type parameters as was supplied in the type argument list, if any, then the identifier refers to that type and is classified as a type.
    32. Otherwise, if no type argument list was supplied and the namespace contains a namespace member with the given name, then the identifier refers to that namespace and is classified as a namespace.
    33. Otherwise, if the namespace contains one or more accessible standard modules, and a member name lookup of the identifier produces an accessible match in exactly one standard module, then the result is exactly the same as a member access of the form `M.E(Of A)`, where `M` is the standard module containing the matching member, `E` is the identifier, and `A` is the type argument list, if any. If the identifier matches accessible type members in more than one standard module, a compile-time error occurs.

4.  If the source file has one or more import aliases, and the identifier matches the name of one of them, then the identifier refers to that namespace or type. If a type argument list is supplied, a compile-time error occurs.

5. If the source file containing the name reference has one or more imports:

    51. If the identifier matches in exactly one import the name of an accessible type with the same number of type parameters as was supplied in the type argument list, if any, or a type member, then the identifier refers to that type or type member. If the identifier matches in more than one import the name of an accessible type with the same number of type parameters as was supplied in the type argument list, if any, or an accessible type member, a compile-time error occurs.
    52. Otherwise, if no type argument list was supplied and the identifier matches in exactly one import the name of a namespace with accessible types, then the identifier refers to that namespace. If no type argument list was supplied and the identifier matches in more than one import the name of a namespace with accessible types, a compile-time error occurs.
    53. Otherwise, if the imports contain one or more accessible standard modules, and a member name lookup of the identifier produces an accessible match in exactly one standard module, then the result is exactly the same as a member access of the form `M.E(Of A)`, where `M` is the standard module containing the matching member, `E` is the identifier, and `A` is the type argument list, if any. If the identifier matches accessible type members in more than one standard module, a compile-time error occurs.

6.  If the compilation environment defines one or more import aliases, and the identifier matches the name of one of them, then the identifier refers to that namespace or type. If a type argument list is supplied, a compile-time error occurs.

7. If the compilation environment defines one or more imports:

    71. If the identifier matches in exactly one import the name of an accessible type with the same number of type parameters as was supplied in the type argument list, if any, or a type member, then the identifier refers to that type or type member. If the identifier matches in more than one import the name of an accessible type with the same number of type parameters as was supplied in the type argument list, if any, or a type member, a compile-time error occurs.
    72. Otherwise, if no type argument list was supplied and the identifier matches in exactly one import the name of a namespace with accessible types, then the identifier refers to that namespace. If no type argument list was supplied and the identifier matches in more than one import the name of a namespace with accessible types, a compile-time error occurs.
    73. Otherwise, if the imports contain one or more accessible standard modules, and a member name lookup of the identifier produces an accessible match in exactly one standard module, then the result is exactly the same as a member access of the form `M.E(Of A)`, where `M` is the standard module containing the matching member,  `E` is the identifier, and `A` is the type argument list, if any. If the identifier matches accessible type members in more than one standard module, a compile-time error occurs.

8. Otherwise, the name given by the identifier is undefined.

A simple name expression that is undefined is a compile-time error.

Normally, a name can only occur once in a particular namespace. However, because namespaces can be declared across multiple .NET assemblies, it is possible to have a situation where two assemblies define a type with the same fully qualified name. In that case, a type declared in the current set of source files is preferred over a type declared in an external .NET assembly. Otherwise, the name is ambiguous and there is no way to disambiguate the name.


### AddressOf Expressions

An `AddressOf` expression is used to produce a method pointer. The expression consists of the `AddressOf` keyword and an expression that must be classified as a method group or a late-bound access. The method group cannot refer to constructors.

The result is classified as a method pointer, with the same associated target expression and type argument list (if any) as the method group.

```antlr
AddressOfExpression
    : 'AddressOf' Expression
    ;
```

## Type Expressions

A *type expression* is a `GetType` expression, a `TypeOf...Is` expression, an `Is` expression, or a `GetXmlNamespace` expression.

```antlr
TypeExpression
    : GetTypeExpression
    | TypeOfIsExpression
    | IsExpression
    | GetXmlNamespaceExpression
    ;
```

### GetType Expressions

A `GetType` expression consists of the keyword `GetType` and the name of a type.

```antlr
GetTypeExpression
    : 'GetType' OpenParenthesis GetTypeTypeName CloseParenthesis
    ;

GetTypeTypeName
    : TypeName
    | QualifiedOpenTypeName
    ;

QualifiedOpenTypeName
    : Identifier TypeArityList? (Period IdentifierOrKeyword TypeArityList?)*
    | 'Global' Period IdentifierOrKeyword TypeArityList?
      (Period IdentifierOrKeyword TypeArityList?)*
    ;

TypeArityList
    : OpenParenthesis 'Of' CommaList? CloseParenthesis
    ;

CommaList
    : Comma Comma*
    ;
```

A `GetType` expression is classified as a value, and its value is the reflection (`System.Type`) class that represents its *GetTypeTypeName*. If the *GetTypeTypeName* is a type parameter, the expression will return the `System.Type` object that corresponds to the type argument supplied for the type parameter at run-time.

The *GetTypeTypeName* is special in two ways:

* It is allowed to be `System.Void`, the only place in the language where this type name may be referenced.

* It may be a constructed generic type with the type arguments omitted. This allows the `GetType` expression to return the `System.Type` object that corresponds to the generic type itself.

The following example demonstrates the `GetType` expression:

```vb
Module Test
    Sub Main()
        Dim t As Type() = { GetType(Integer), GetType(System.Int32), _
            GetType(String), GetType(Double()) }
        Dim i As Integer

        For i = 0 To t.Length - 1
            Console.WriteLine(t(i).Name)
        Next i
    End Sub
End Module
```

The resulting output is:

```
Int32
Int32
String
Double[]
```


### TypeOf...Is Expressions

A `TypeOf...Is` expression is used to check whether the run-time type of a value is compatible with a given type. The first operand must be classified as a value, cannot be a reclassified lambda method, and must be of a reference type or an unconstrained type parameter type. The second operand must be a type name. The result of the expression is classified as a value and is a `Boolean` value. The expression evaluates to `True` if the run-time type of the operand has an identity, default, reference, array, value type, or type parameter conversion to the type, `False` otherwise. A compile-time error occurs if no conversion exists between the type of the expression and the specific type.

```antlr
TypeOfIsExpression
    : 'TypeOf' Expression 'Is' LineTerminator? TypeName
    ;
```

### Is Expressions

An `Is` or `IsNot` expression is used to do a reference equality comparison.

```antlr
IsExpression
    : Expression 'Is' LineTerminator? Expression
    | Expression 'IsNot' LineTerminator? Expression
    ;
```

Each expression must be classified as a value and the type of each expression must be a reference type, an unconstrained type parameter type, or a nullable value type. If the type of one expression is an unconstrained type parameter type or nullable value type, however, the other expression must be the literal `Nothing`.

The result is classified as a value and is typed as `Boolean`. An `Is` operation evaluates to `True` if both values refer to the same instance or both values are `Nothing`, or `False` otherwise. An `IsNot` operation evaluates to `False` if both values refer to the same instance or both values are `Nothing`, or `True` otherwise.


### GetXmlNamespace Expressions

A `GetXmlNamespace` expression consists of the keyword `GetXmlNamespace` and the name of an XML namespace declared by the source file or compilation environment.

```antlr
GetXmlNamespaceExpression
    : 'GetXmlNamespace' OpenParenthesis XMLNamespaceName? CloseParenthesis
    ;
```

An `GetXmlNamespace` expression is classified as a value, and its value is an instance of `System.Xml.Linq.XNamespace` that represents the *XMLNamespaceName*. If that type is not available, then a compile-time error will occur.

For example:

```vb
Imports <xmlns:db="http://example.org/database">

Module Test
    Sub Main()
        Dim db = GetXmlNamespace(db)

        ' The following are equivalent
        Dim customer1 = _
            New System.Xml.Linq.XElement(db + "customer", "Bob")
        Dim customer2 = <db:customer>Bob</>
    End Sub
End Module
```

Everything between the parentheses is considered part of the namespace name, so XML rules around things such as whitespace apply. For example:

```vb
Imports <xmlns:db-ns="http://example.org/database">

Module Test
    Sub Main()

        ' Error, XML name expected
        Dim db1 = GetXmlNamespace( db-ns )

        ' Error, ')' expected
        Dim db2 = GetXmlNamespace(db _
            )

        ' OK.
        Dim db3 = GetXmlNamespace(db-ns)
    End Sub
End Module
```

The XML namespace expression can also be omitted, in which case the expression returns the object that represents the default XML namespace.


## Member Access Expressions

A member access expression is used to access a member of an entity.

```antlr
MemberAccessExpression
    : MemberAccessBase? Period IdentifierOrKeyword
      ( OpenParenthesis 'Of' TypeArgumentList CloseParenthesis )?
    ;

MemberAccessBase
    : Expression
    | NonArrayTypeName
    | 'Global'
    | 'MyClass'
    | 'MyBase'
    ;
```

A member access of the form `E.I(Of A)`, where `E` is an expression, a non-array type name, the keyword `Global`, or omitted and `I` is an identifier with an optional type argument list `A`, is evaluated and classified as follows:

1. If `E` is omitted, then the expression from the immediately containing `With` statement is substituted for `E` and the member access is performed. If there is no containing `With` statement, a compile-time error occurs.

2. If `E` is classified as a namespace or `E` is the keyword `Global`, then the member lookup is done in the context of the specified namespace. If `I` is the name of an accessible member of that namespace with the same number of type parameters as was supplied in the type argument list, if any, then the result is that member. The result is classified as a namespace or a type depending on the member. Otherwise, a compile-time error occurs.

3. If `E` is a type or an expression classified as a type, then the member lookup is done in the context of the specified type. If `I` is the name of an accessible member of `E`, then `E.I` is evaluated and classified as follows:

    31. If `I` is the keyword `New` and `E` is not an enumeration then a compile-time error occurs.
    32. If `I` identifies a type with the same number of type parameters as was supplied in the type argument list, if any, then the result is that type.
    33. If `I` identifies one or more methods, then the result is a method group with the associated type argument list and no associated target expression.
    34. If `I` identifies one or more properties and no type argument list was supplied, then the result is a property group with no associated target expression.
    35. If `I` identifies a shared variable and no type argument list was supplied, then the result is either a variable or a value. If the variable is read-only, and the reference occurs outside the shared constructor of the type in which the variable is declared, then the result is the value of the shared variable `I` in `E`. Otherwise, the result is the shared variable `I` in `E`.
    36. If `I` identifies a shared event and no type argument list was supplied, the result is an event access with no associated target expression.
    37. If `I` identifies a constant and no type argument list was supplied, then the result is the value of that constant.
    38. If `I` identifies an enumeration member and no type argument list was supplied, then the result is the value of that enumeration member.
    39. Otherwise, `E.I` is an invalid member reference, and a compile-time error occurs.

4. If `E` is classified as a variable or value, the type of which is `T`, then the member lookup is done in the context of `T`. If `I` is the name of an accessible member of `T`, then `E.I` is evaluated and classified as follows:

    41. If `I` is the keyword `New`, `E` is  `Me`, `MyBase`, or `MyClass`, and no type arguments were supplied, then the result is a method group representing the instance constructors of the type of `E` with an associated target expression of `E` and no type argument list. Otherwise, a compile-time error occurs.
    42. If `I` identifies one or more methods, including extension methods if `T` is not `Object`, then the result is a method group with the associated type argument list and an associated target expression of `E`.
    43. If `I` identifies one or more properties and no type arguments were supplied, then the result is a property group with an associated target expression of `E`.
    44. If `I` identifies a shared variable or an instance variable and no type arguments were supplied, then the result is either a variable or a value. If the variable is read-only, and the reference occurs outside a constructor of the class in which the variable is declared appropriate for the kind of variable (shared or instance), then the result is the value of the variable `I` in the object referenced by `E`. If `T` is a reference type, then the result is the variable `I` in the object referenced by `E`. Otherwise, if `T` is a value type and the expression `E` is classified as a variable, the result is a variable; otherwise the result is a value.
    45. If `I` identifies an event and no type arguments were supplied, the result is an event access with an associated target expression of `E`.
    46. If `I` identifies a constant and no type arguments were supplied, then the result is the value of that constant.
    47. If `I` identifies an enumeration member and no type arguments were supplied, then the result is the value of that enumeration member.
    48. If `T` is `Object`, then the result is a late-bound member lookup classified as a late-bound access with the associated type argument list and an associated target expression of `E`.

5. Otherwise, `E.I` is an invalid member reference, and a compile-time error occurs.

A member access of the form `MyClass.I(Of A)` is equivalent to `Me.I(Of A)`, but all members accessed on it are treated as if the members are non-overridable. Thus, the member accessed will not be affected by the run-time type of the value on which the member is being accessed.

A member access of the form `MyBase.I(Of A)` is equivalent to `CType(Me, T).I(Of A)` where `T` is the direct base type of the type containing the member access expression. All method invocations on it are treated as if the method being invoked is non-overridable. This form of member access is also called a *base access*.

The following example demonstrates how `Me`, `MyBase` and `MyClass` relate:

```vb
Class Base
    Public Overridable Sub F()
        Console.WriteLine("Base.F")
    End Sub
End Class

Class Derived
    Inherits Base

    Public Overrides Sub F()
        Console.WriteLine("Derived.F")
    End Sub

    Public Sub G()
        MyClass.F()
    End Sub
End Class

Class MoreDerived
    Inherits Derived

    Public Overrides Sub F()
        Console.WriteLine("MoreDerived.F")
    End Sub

    Public Sub H()
        MyBase.F()
    End Sub
End Class

Module Test
    Sub Main()
        Dim x As MoreDerived = new MoreDerived()

        x.F()
        x.G()
        x.H()
    End Sub

End Module
```

This code prints out:

```
MoreDerived.F
Derived.F
Derived.F
```

When a member access expression begins with the keyword `Global`, the keyword represents the outermost unnamed namespace, which is useful in situations where a declaration shadows an enclosing namespace. The `Global` keyword allows "escaping" out to the outermost namespace in that situation. For example:

```vb
Class System
End Class

Module Test
    Sub Main()
        ' Error: Class System does not contain Console
        System.Console.WriteLine("Hello, world!") 


        ' Legal, binds to System in outermost namespace
        Global.System.Console.WriteLine("Hello, world!") 
    End Sub
End Module
```

In the above example, the first method call is invalid because the identifier `System` binds to the class `System`, not the namespace `System`. The only way to access the `System` namespace is to use `Global` to escape out to the outermost namespace.

If the member being accessed is shared, any expression on the left side of the period is superfluous and is not evaluated unless the member access is done late-bound. For example, consider the following code:

```vb
Class C
    Public Shared F As Integer = 10
End Class

Module Test
    Public Function ReturnC() As C
        Console.WriteLine("Returning a new instance of C.")
        Return New C()
    End Function

    Public Sub Main()
        Console.WriteLine("The value of F is: " & ReturnC().F)
    End Sub
End Module
```

It prints `The value of F is: 10` because the function `ReturnC` does not need to be called to provide an instance of `C` to access the shared member `F`.


### Identical Type and Member Names

It is not uncommon to name members using the same name as their type. In that situation, however, inconvenient name hiding can occur:

```vb
Enum Color
    Red
    Green
    Yellow
End Enum

Class Test
    ReadOnly Property Color() As Color
        Get
            Return Color.Red
        End Get
    End Property

    Shared Function DefaultColor() As Color
        Return Color.Green    ' Binds to the instance property!
    End Function
End Class
```

In the previous example, the simple name `Color` in `DefaultColor` binds to the instance property instead of the type. Because an instance member cannot be referenced in a shared member, this would normally be an error.

However, a special rule allows access to the type in this case. If the base expression of a member access expression is a simple name and binds to a constant, field, property, local variable or parameter whose type has the same name, then the base expression can refer either to the member or the type. This can never result in ambiguity because the members that can be accessed off of either one are the same.

### Default Instances

In some situations, classes derived from a common base class usually or always have only a single instance. For example, most windows shown in a user interface only ever have one instance showing on the screen at any time. To simplify working with these types of classes, Visual Basic can automatically generate *default instances* of the classes that provide a single, easily referenced instance for each class.

Default instances are always created for a *family* of types rather than for one particular type. So instead of creating a default instance for a class Form1 that derives from Form, default instances are created for all classes derived from Form. This means that each individual class that derives from the base class does not have to be specially marked to have a default instance.

The default instance of a class is represented by a compiler-generated property that returns the default instance of that class. The property generated as a member of a class called the *group class* that manages allocating and destroying default instances for all classes derived from the particular base class. For example, all of the default instance properties of classes derived from `Form` may be collected in the `MyForms` class. If an instance of the group class is returned by the expression `My.Forms`, then the following code accesses the default instances of derived classes `Form1` and `Form2`:

```vb
Class Form1
    Inherits Form
    Public x As Integer
End Class

Class Form2
    Inherits Form
    Public y As Integer
End Class

Module Main
    Sub Main()
        My.Forms.Form1.x = 10
        Console.WriteLine(My.Forms.Form2.y)
    End Sub
End Module
```

Default instances will not be created until the first reference to them; fetching the property representing the default instance causes the default instance to be created if it has not already been created or has been set to `Nothing`. To allow testing for the existence of a default instance, when a default instance is the target of an `Is` or `IsNot` operator, the default instance will not be created. Thus, it is possible to test whether a default instance is `Nothing` or some other reference without causing the default instance to be created.

Default instances are intended to make it easy to refer to the default instance from outside of the class that has the default instance. Using a default instance from within a class that defines it might cause confusion as to which instance is being referred to, i.e. the default instance or the current instance. For example, the following code modifies only the value `x` in the default instance, even though it is being called from another instance. Thus the code would print the value `5` instead of `10`:

```vb
Class Form1
    Inherits Form

    Public x As Integer = 5

    Public Sub ChangeX()
        Form1.x = 10
    End Sub
End Class

Module Main
    Sub Main()
        Dim f As Form1 = New Form1()
        f.ChangeX()
        Console.WriteLine(f.x)
    End Sub
End Module
```

To prevent this kind of confusion, it is not valid to refer to a default instance from within an instance method of the default instance's type.

#### Default Instances and Type Names

A default instance may also be accessible directly through its type's name. In this case, in any expression context where the type name is not allowed the expression `E`, where `E` represents the fully qualified name of the class with a default instance, is changed to `E'`, where `E'` represents an expression that fetches the default instance property. For example, if default instances for classes derived from `Form` allow accessing the default instance through the type name, then the following code is equivalent to the code in the previous example:

```vb
Module Main
    Sub Main()
        Form1.x = 10
        Console.WriteLine(Form2.y)
    End Sub
End Module
```

This also means that a default instance that is accessible through its type's name is also assignable through the type name. For example, the following code sets the default instance of `Form1` to `Nothing`:

```vb
Module Main
    Sub Main()
        Form1 = Nothing
    End Sub
End Module
```

Note that the meaning of `E.I` were `E` represents a class and `I` represents a shared member does not change. Such an expression still accesses the shared member directly off of the class instance and does not reference the default instance.

#### Group Classes

The `Microsoft.VisualBasic.MyGroupCollectionAttribute` attribute indicates the group class for a family of default instances. The attribute has four parameters:

* The parameter `TypeToCollect` specifies the base class for the group. All instantiable classes without open type parameters that derive from a type with this name (regardless of type parameters) will automatically have a default instance.

* The parameter `CreateInstanceMethodName` specifies the method to call in the group class to create a new instance in a default instance property.

* The parameter `DisposeInstanceMethodName` specifies the method to call in the group class to dispose of a default instance property if the default instance property is assigned the value `Nothing`.

* The parameter `DefaultInstanceAlias` specifics the expression `E'` to substitute for the class name if the default instances are accessible directly through their type name. If this parameter is `Nothing` or an empty string, default instances on this group type are not accessible directly through their type's name. (__Note.__ In all current implementations of the Visual Basic language, the `DefaultInstanceAlias` parameter is ignored, except in compiler-provided code.)

Multiple types can be collected into the same group by separating the names of the types and methods in the first three parameters using commas. There must be the same number of items in each parameter, and the list elements are matched in order. For example, the following attribute declaration collects types that derive from `C1`, `C2` or `C3` into a single group:

```vb
<Microsoft.VisualBasic.MyGroupCollection("C1, C2, C3", _
    "CreateC1, CreateC2, CreateC3", _
    "DisposeC1, DisposeC2, DisposeC3", "My.Cs")>
Public NotInheritable Class MyCs
    ...
End Class
```

The signature of the create method must be of the form `Shared Function <Name>(Of T As {New, <Type>})(Instance Of T) As T`. The dispose method must be of the form `Shared Sub <Name>(Of T As <Type>)(ByRef Instance Of T)`. Thus, the group class for the example in the preceding section could be declared as follows:

```vb
<Microsoft.VisualBasic.MyGroupCollection("Form", "Create", _
    "Dispose", "My.Forms")> _
Public NotInheritable Class MyForms
    Private Shared Function Create(Of T As {New, Form}) _
        (Instance As T) As T
        If Instance Is Nothing Then
            Return New T()
        Else
            Return Instance
        End If
    End Function

    Private Shared Sub Dispose(Of T As Form)(ByRef Instance As T)
        Instance.Close()
        Instance = Nothing
    End Sub
End Class
```

If a source file declared a derived class `Form1`, the generated group class would be equivalent to:

```vb
<Microsoft.VisualBasic.MyGroupCollection("Form", "Create", _
    "Dispose", "My.Forms")> _
Public NotInheritable Class MyForms
    Private Shared Function Create(Of T As {New, Form}) _
        (Instance As T) As T
        If Instance Is Nothing Then
            Return New T()
        Else
            Return Instance
        End If
    End Function

    Private Shared Sub Dispose(Of T As Form)(ByRef Instance As T)
        Instance.Close()
        Instance = Nothing
    End Sub

    Private m_Form1 As Form1

    Public Property Form1() As Form1
        Get
            Return Create(m_Form1)
        End Get
        Set (Value As Form1)
            If Value IsNot Nothing AndAlso Value IsNot m_Form1 Then
                Throw New ArgumentException( _
                    "Property can only be set to Nothing.")
            End If
            Dispose(m_Form1)
        End Set
    End Property
End Class
```

### Extension Method Collection

Extension methods for the member access expression `E.I` are collected by gathering all of the extension methods with the name `I` that are available in the current context:

1. First, each nested type containing the expression is checked, starting from the innermost and going to the outermost.
2. Then, each nested namespace is checked, starting from the innermost and going to the outermost namespace.
3. Then, the imports in the source file are checked.
4. Then, the imports defined by the compilation environment are checked.

An extension method is collected only if there is a widening native conversion from the target expression type to the type of the first parameter of the extension method. And unlike regular simple name expression binding, the search collects *all* extension methods; the collection does not stop when an extension method is found. For example:

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
        Sub M1(c As C1, y As Double)
        End Sub
    End Module
End Namespace

Namespace N1.N2.N3
    Module Test
        Sub Main()
            Dim x As New C1()

            ' Calls N1C1Extensions.M1
            x.M1(10)
        End Sub
    End Module
End Namespace
```

In this example, even though `N2C1Extensions.M1` is found before `N1C1Extensions.M1`, they both are considered as extension methods. Once all of the extension methods have been collected, they are then *curried*. Currying takes the target of the extension method call and applies it to the extension method call, resulting in a new method signature with the first parameter removed (because it has been specified). For example:

```vb
Imports System.Runtime.CompilerServices

Module Ext1
    <Extension> _
    Sub M(x As Integer, y As Integer)
    End Sub
End Module

Module Ext2
    <Extension> _
    Sub M(x As Integer, y As Double)
    End Sub
End Module

Module Main
    Sub Test()
        Dim v As Integer = 10

        ' The curried method signatures considered are:
        '        Ext1.M(y As Integer)
        '        Ext2.M(y As Double)
        v.M(10)
    End Sub
End Module
```

In the above example, the curried result of applying `v` to `Ext1.M` is the method signature `Sub M(y As Integer)`.

In addition to removing the first parameter of the extension method, currying also removes any method type parameters that are a part of the type of the first parameter. When currying an extension method with method type parameter, type inference is applied to the first parameter and the result is fixed for any type parameters that are inferred. If type inference fails, the method is ignored. For example:

```vb
Imports System.Runtime.CompilerServices

Module Ext1
    <Extension> _
    Sub M(Of T, U)(x As T, y As U)
    End Sub
End Module

Module Ext2
    <Extension> _
    Sub M(Of T)(x As T, y As T)
    End Sub
End Module

Module Main
    Sub Test()
        Dim v As Integer = 10

        ' The curried method signatures considered are:
        '        Ext1.M(Of U)(y As U)
        '        Ext2.M(y As Integer)
        v.M(10)
    End Sub
End Module
```

In the above example, the curried result of applying `v` to `Ext1.M` is the method signature `Sub M(Of U)(y As U)`, because the type parameter `T` is inferred as a result of the currying and is now fixed. Because the type parameter `U` was not inferred as a part of the currying, it remains an open parameter. Similarly, because the type parameter `T` is inferred as a result of applying `v` to `Ext2.M`, the type of parameter `y` becomes fixed as `Integer`. It will not be inferred to be any other type. When currying the signature, all constraints except for `New` constraints are also applied. If the constraints are not satisfied, or depend on a type that was not inferred as a part of currying, the extension method is ignored. For example:

```vb
Imports System.Runtime.CompilerServices

Module Ext1
    <Extension> _
    Sub M1(Of T As Structure)(x As T, y As Integer)
    End Sub

    <Extension> _
    Sub M2(Of T As U, U)(x As T, y As U)
    End Sub
End Module

Module Main
    Sub Test()
        Dim s As String = "abc"

        ' Error: String does not satisfy the Structure constraint
        s.M1(10)

        ' Error: T depends on U, which cannot be inferred
        s.M2(10)
    End Sub
End Module
```

__Note.__ One of the main reasons for doing currying of extension methods is that it allows query expressions to infer the type of the iteration before evaluating the arguments to a query pattern method. Since most query pattern methods take lambda expressions, which require type inference themselves, this greatly simplifies the process of evaluating a query expression.

Unlike normal interface inheritance, extension methods that extend two interfaces that do not relate to one another are available, as long as they do not have the same curried signature:

```vb
Imports System.Runtime.CompilerServices

Interface I1
End Interface

Interface I2
End Interface

Class C1
    Implements I1, I2
End Class

Module I1Ext
    <Extension> _
    Sub M1(i As I1, x As Integer)
    End Sub

    <Extension> _
    Sub M2(i As I1, x As Integer)
    End Sub
End Module

Module I2Ext
    <Extension> _
    Sub M1(i As I2, x As Integer)
    End Sub

    <Extension> _
    Sub M2(I As I2, x As Double)
    End Sub
End Module

Module Main
    Sub Test()
        Dim c As New C1()

        ' Error: M is ambiguous between I1Ext.M1 and I2Ext.M1.
        c.M1(10)

        ' Calls I1Ext.M2
        c.M2(10)
    End Sub
End Module
```

Finally, it is important to remember that extension methods are not considered when doing late binding:

```vb
Module Test
    Sub Main()
        Dim o As Object = ...

        ' Ignores extension methods
        o.M1()
    End Sub
End Module
```

## Dictionary Member Access Expressions

A *dictionary member access expression* is used to look up a member of a collection. A dictionary member access takes the form of `E!I`, where `E` is an expression that is classified as a value and `I` is an identifier.

```antlr
DictionaryAccessExpression
    : Expression? '!' IdentifierOrKeyword
    ;
```

The type of the expression must have a default property indexed by a single `String` parameter. The dictionary member access expression `E!I` is transformed into the expression `E.D("I")`, where `D` is the default property of `E`. For example:

```vb
Class Keys
    Public ReadOnly Default Property Item(s As String) As Integer
        Get
            Return 10
        End Get
    End Property 
End Class

Module Test
    Sub Main()
        Dim x As Keys = new Keys()
        Dim y As Integer
        ' The two statements are equivalent.
        y = x!abc
        y = x("abc")
    End Sub
End Module
```

If an exclamation point is specified with no expression, the expression from the immediately containing `With` statement is assumed. If there is no containing `With` statement, a compile-time error occurs.


## Invocation Expressions

An invocation expression consists of an invocation target and an optional argument list.

```antlr
InvocationExpression
    : Expression ( OpenParenthesis ArgumentList? CloseParenthesis )?
    ;

ArgumentList
    : PositionalArgumentList
    | PositionalArgumentList Comma NamedArgumentList
    | NamedArgumentList
    ;

PositionalArgumentList
    : Expression? ( Comma Expression? )*
    ;

NamedArgumentList
    : IdentifierOrKeyword ColonEquals Expression
      ( Comma IdentifierOrKeyword ColonEquals Expression )*
    ;
```

The target expression must be classified as a method group or a value whose type is a delegate type. If the target expression is a value whose type is a delegate type, then the target of the invocation expression becomes the method group for the `Invoke` member of the delegate type and the target expression becomes the associated target expression of the method group.

An argument list has two sections: positional arguments and named arguments. *Positional arguments* are expressions and must precede any named arguments. *Named arguments* start with an identifier that can match keywords, followed by `:=` and an expression.

If the method group only contains one accessible method, including both instance and extension methods, and that method takes no arguments and is a function, then the method group is interpreted as an invocation expression with an empty argument list and the result is used as the target of an invocation expression with the provided argument list(s). For example:

```vb
Class C1
    Function M1() As Integer()
        Return New Integer() { 1, 2, 3 }
    End Sub
End Class

Module Test
    Sub Main()
        Dim c As New C1()

        ' Prints 3
        Console.WriteLine(c.M1(2))
    End Sub
End Module
```

Otherwise, overload resolution is applied to the methods to pick the most applicable method for the given argument list(s). If the most applicable method is a function, then the result of the invocation expression is classified as a value typed as the return type of the function. If the most applicable method is a subroutine, then the result is classified as void. If the most applicable method is a partial method that has no body, then the invocation expression is ignored and the result is classified as void.

For an early-bound invocation expression, the arguments are evaluated in the order in which the corresponding parameters are declared in the target method. For a late-bound member access expression, they are evaluated in the order in which they appear in the member access expression: see Section [Late-Bound Expressions](expressions.md#late-bound-expressions).


### Overloaded Method Resolution

In practice, the rules for determining overload resolution are intended to find the overload that is "closest" to the actual arguments supplied. If there is a method whose parameter types match the argument types, then that method is obviously the closest. Barring that, one method is closer than another if all of its parameter types are narrower than (or the same as) the parameter types of the other method. If neither method's parameters are narrower than the other, then there is no way for to determine which method is closer to the arguments.

__Note.__ Overload resolution does not take into account the expected return type of the method. 

Also note that because of the named parameter syntax, the ordering of the actual and formal parameters may not be the same.

Given a method group, the most applicable method in the group for an argument list is determined using the following steps. If, after applying a particular step, no members remain in the set, then a compile-time error occurs. If only one member remains in the set, then that member is the most applicable member. The steps are:

1.  First, if no type arguments have been supplied, apply type inference to any methods which have type parameters. If type inference succeeds for a method, then the inferred type arguments are used for that particular method. If type inference fails for a method, then that method is eliminated from the set.

2.  Next, eliminate all members from the set that are inaccessible or not applicable (Section [Applicability To Argument List](expressions.md#applicability-to-argument-list)) to the argument list

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

4.  Next, elimination is done based on narrowing as follows. (Note that, if Option Strict is On, then all members that require narrowing have already been judged inapplicable (Section [Applicability To Argument List](expressions.md#applicability-to-argument-list)) and removed by Step 2.)

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

6.  Next, if, given any two members of the set `M` and `N`, `M` is more *specific* (Section [Specificity of members/types given an argument list](expressions.md#specificity-of-memberstypes-given-an-argument-list)) than `N` given the argument list, eliminate `N` from the set. If more than one member remains in the set and the remaining members are not equally specific given the argument list, a compile-time error results.

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

    75. Before type arguments have been substituted, if `M` is *less generic* (Section [Genericity](expressions.md#genericity)) than `N`, eliminate `N` from the set.

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

    711. Before type arguments have been substituted, if `M` has *greater depth of genericity* (Section [Genericity](expressions.md#genericity)) than `N`, then eliminate `N` from the set.

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

## Index Expressions

An *index expression* results in an array element or reclassifies a property group into a property access. An index expression consists of, in order, an expression, an opening parenthesis, an index argument list, and a closing parenthesis.

```antlr
IndexExpression
    : Expression OpenParenthesis ArgumentList? CloseParenthesis
    ;
```

The target of the index expression must be classified as either a property group or a value. An index expression is processed as follows:

* If the target expression is classified as a value and if its type is not an array type, `Object`, or `System.Array`, the type must have a default property. The index is performed on a property group that represents all of the default properties of the type. Although it is not valid to declare a parameterless default property in Visual Basic, other languages may allow declaring such a property. Consequently, indexing a property with no arguments is allowed.

* If the expression results in a value of an array type, the number of arguments in the argument list must be the same as the rank of the array type and may not include named arguments. If any of the indexes are invalid at run time, a `System.IndexOutOfRangeException` exception is thrown. Each expression must be implicitly convertible to type `Integer`. The result of the index expression is the variable at the specified index and is classified as a variable.

* If the expression is classified as a property group, overload resolution is used to determine whether one of the properties is applicable to the index argument list. If the property group only contains one property that has a `Get` accessor and if that accessor takes no arguments, then the property group is interpreted as an index expression with an empty argument list. The result is used as the target of the current index expression. If no properties are applicable, then a compile-time error occurs. Otherwise, the expression results in a property access with the associated target expression (if any) of the property group.

* If the expression is classified as a late-bound property group or as a value whose type is `Object` or `System.Array`, the processing of the index expression is deferred until run time and the indexing is late-bound. The expression results in a late-bound property access typed as `Object`. The associated target expression is either the target expression, if it is a value, or the associated target expression of the property group. At run time the expression is processed as follows:

* If the expression is classified as a late-bound property group, the expression may result in a method group, a property group, or a value (if the member is an instance or shared variable). If the result is a method group or property group, overload resolution is applied to the group to determine the correct method for the argument list. If overload resolution fails, a `System.Reflection.AmbiguousMatchException` exception is thrown. Then the result is processed either as a property access or as an invocation and the result is returned. If the invocation is of a subroutine, the result is `Nothing`.

* If the run-time type of the target expression is an array type or `System.Array`, the result of the index expression is the value of the variable at the specified index.

* Otherwise, the run-time type of the expression must have a default property and the index is performed on the property group that represents all of the default properties on the type. If the type has no default property, then a `System.MissingMemberException` exception is thrown.


## New Expressions

The `New` operator is used to create new instances of types. There are four forms of `New` expressions:

* Object-creation expressions are used to create new instances of class types and value types.

* Array-creation expressions are used to create new instances of array types.

* Delegate-creation expressions (which do not have a distinct syntax from object-creation expressions) are used to create new instances of delegate types.

* Anonymous object-creation expressions are used to create new instances of anonymous class types.

```antlr
NewExpression
    : ObjectCreationExpression
    | ArrayExpression
    | AnonymousObjectCreationExpression
    ;
```

A `New` expression is classified as a value and the result is the new instance of the type.


### Object-Creation Expressions

An object-creation expression is used to create a new instance of a class type or a structure type.

```antlr
ObjectCreationExpression
    : 'New' NonArrayTypeName ( OpenParenthesis ArgumentList? CloseParenthesis )?
      ObjectCreationExpressionInitializer?
    ;

ObjectCreationExpressionInitializer
    : ObjectMemberInitializer
    | ObjectCollectionInitializer
    ;

ObjectMemberInitializer
    : 'With' OpenCurlyBrace FieldInitializerList CloseCurlyBrace
    ;

FieldInitializerList
    : FieldInitializer ( Comma FieldInitializer )*
    ;

FieldInitializer
    : 'Key'? ('.' IdentifierOrKeyword Equals )? Expression
    ;

ObjectCollectionInitializer
    : 'From' CollectionInitializer
    ;

CollectionInitializer
    : OpenCurlyBrace CollectionElementList? CloseCurlyBrace
    ;

CollectionElementList
    : CollectionElement ( Comma CollectionElement )*
    ;

CollectionElement
    : Expression
    | CollectionInitializer
    ;
```

The type of an object creation expression must be a class type, a structure type, or a type parameter with a `New` constraint and cannot be a `MustInherit` class. Given an object creation expression of the form `New T(A)`, where `T` is a class type or structure type and `A` is an optional argument list, overload resolution determines the correct constructor of `T` to call. A type parameter with a `New` constraint is considered to have a single, parameterless constructor. If no constructor is callable, a compile-time error occurs; otherwise the expression results in the creation of a new instance of `T` using the chosen constructor. If there are no arguments, the parentheses may be omitted.

Where an instance is allocated depends on whether the instance is a class type or a value type. `New` instances of class types are created on the system heap, while new instances of value types are created directly on the stack.

An object-creation expression can optionally specify a list of member initializers after the constructor arguments. These member initializers are prefixed with the keyword `With`, and the initializer list is interpreted as if it was in the context of a `With` statement. For example, given the class:

```vb
Class Customer
    Dim Name As String
    Dim Address As String
End Class
```

The code:

```vb
Module Test
    Sub Main()
        Dim x As New Customer() With { .Name = "Bob Smith", _
            .Address = "123 Main St." }
    End Sub
End Module
```

Is roughly equivalent to:

```vb
Module Test
    Sub Main()
        Dim x, _t1 As Customer

        _t1 = New Customer()
        With _t1
            .Name = "Bob Smith"
            .Address = "123 Main St."
        End With

        x = _t1
    End Sub
End Module
```

Each initializer must specify a name to assign, and the name must be a non-`ReadOnly` instance variable or property of the type being constructed; the member access will not be late bound if the type being constructed is `Object`. Initializers may not use the `Key` keyword. Each member in a type can only be initialized once. The initializer expressions, however, may refer to each other. For example:

```vb
Module Test
    Sub Main()
        Dim x As New Customer() With { .Name = "Bob Smith", _
            .Address = .Name & " St." }
    End Sub
End Module
```

The initializers are assigned left-to-right, so if an initializer refers to a member that has not been initialized yet, it will see whatever value the instance variable after the constructor ran:

```vb
Module Test
    Sub Main()
        ' The value of Address will be " St." since Name has not been
        ' assigned yet.
        Dim x As New Customer() With { .Address = .Name & " St." }
    End Sub
End Module
```

Initializers can be nested:

```vb
Class Customer
    Dim Name As String
    Dim Address As Address
    Dim Age As Integer
End Class

Class Address
    Dim Street As String
    Dim City As String
    Dim State As String
    Dim ZIP As String
End Class

Module Test
    Sub Main()
        Dim c As New Customer() With { _
            .Name = "John Smith", _
            .Address = New Address() With { _
                .Street = "23 Main St.", _
                .City = "Peoria", _
                .State = "IL", _
                .ZIP = "13934" }, _
            .Age = 34 }
    End Sub
End Module
```

If the type being created is a collection type and has an instance method named `Add` (including extension methods and shared methods), then the object-creation expression can specify a collection initializer prefixed by the keyword `From`. An object-creation expression cannot specify both a member initializer and a collection initializer. Each element in the collection initializer is passed as an argument to an invocation of the `Add` function. For example:

```vb
Dim list = New List(Of Integer)() From { 1, 2, 3, 4 }
```

is equivalent to:

```vb
Dim list = New List(Of Integer)()
list.Add(1)
list.Add(2)
list.Add(3)
```

If an element is a collection initializer itself, each element of the sub-collection initializer will be passed as an individual argument to the `Add` function. For example, the following:

```vb
Dim dict = Dictionary(Of Integer, String) From { { 1, "One" },{ 2, "Two" } }
```

is equivalent to:

```vb
Dim dict = New Dictionary(Of Integer, String)
dict.Add(1, "One")
dict.Add(2, "Two")
```

This expansion is always done and is only ever done one level deep; after that, sub-initializers are considered array literals. For example:

```vb
' Error: List(Of T) does not have an Add method that takes two parameters.
Dim list = New List(Of Integer())() From { { 1, 2 }, { 3, 4 } }

' OK, this initializes the dictionary with (Integer, Integer()) pairs.
Dim dict = New Dictionary(Of Integer, Integer())() From _
        { {  1, { 2, 3 } }, { 3, { 4, 5 } } }
```


### Array Expressions

An array expression is used to create a new instance of an array type. There are two types of array expressions: array creation expressions, and array literals.

#### Array creation expressions

If an array size initialization modifier is supplied, the resulting array type is derived by deleting each of the individual arguments from the array size initialization argument list. The value of each argument determines the upper bound of the corresponding dimension in the newly allocated array instance. If the expression has a non-empty collection initializer, each argument in the argument list must be a constant, and the rank and dimension lengths specified by the expression list must match those of the collection initializer.

```vb
Dim a() As Integer = New Integer(2) {}
Dim b() As Integer = New Integer(2) { 1, 2, 3 }
Dim c(,) As Integer = New Integer(1, 2) { { 1, 2, 3 } , { 4, 5, 6 } }

' Error, length/initializer mismatch.
Dim d() As Integer = New Integer(2) { 0, 1, 2, 3 }
```

If an array size initialization modifier is not supplied, then the type name must be an array type and the collection initializer must be empty or have the same number of levels of nesting as the rank of the specified array type. All of the elements in the innermost nesting level must be implicitly convertible to the element type of the array and must be classified as a value. The number of elements in each nested collection initializer must always be consistent with the size of the other collections at the same level. The individual dimension lengths are inferred from the number of elements in each of the corresponding nesting levels of the collection initializer. If the collection initializer is empty, the length of each dimension is zero.

```vb
Dim e() As Integer = New Integer() { 1, 2, 3 }
Dim f(,) As Integer = New Integer(,) { { 1, 2, 3 } , { 4, 5, 6 } }

' Error: Inconsistent numbers of elements!
Dim g(,) As Integer = New Integer(,) { { 1, 2 }, { 4, 5, 6 } }

' Error: Inconsistent levels of nesting!
Dim h(,) As Integer = New Integer(,) { 1, 2, { 3, 4 } }
```

The outermost nesting level of a collection initializer corresponds to the leftmost dimension of an array, and the innermost nesting level corresponds to the rightmost dimension. The example:

```vb
Dim array As Integer(,) = _
    { { 0, 1 }, { 2, 3 }, { 4, 5 }, { 6, 7 }, { 8, 9 } }
```

Is equivalent to the following:

```vb
Dim array(4, 1) As Integer

array(0, 0) = 0: array(0, 1) = 1
array(1, 0) = 2: array(1, 1) = 3
array(2, 0) = 4: array(2, 1) = 5
array(3, 0) = 6: array(3, 1) = 7
array(4, 0) = 8: array(4, 1) = 9
```

If the collection initializer is empty (that is, one that contains curly braces but no initializer list) and the bounds of the dimensions of the array being initialized are known, the empty collection initializer represents an array instance of the specified size where all the elements have been initialized to the element type's default value. If the bounds of the dimensions of the array being initialized are not known, the empty collection initializer represents an array instance in which all dimensions are size zero.

An array instance's rank and length of each dimension are constant for the entire lifetime of the instance. In other words, it is not possible to change the rank of an existing array instance, nor is it possible to resize its dimensions.

#### Array Literals

An array literal denotes an array whose element type, rank, and bounds are inferred from a combination of the expression context and a collection initializer. This is explained in Section [Expression Reclassification](expressions.md#expression-reclassification).

```antlr
ArrayExpression
    : ArrayCreationExpression
    | ArrayLiteralExpression
    ;

ArrayCreationExpression
    : 'New' NonArrayTypeName ArrayNameModifier CollectionInitializer
    ;

ArrayLiteralExpression
    : CollectionInitializer
    ;
```

For example:

```vb
' array of integers
Dim a = {1, 2, 3}

' array of shorts
Dim b = {1S, 2S, 3S}

' array of shorts whose type is taken from the context
Dim c As Short() = {1, 2, 3}

' array of type Integer(,)
Dim d = {{1, 0}, {0, 1}}

' jagged array of rank ()()
Dim e = {({1, 0}), ({0, 1})}

' error: inconsistent rank
Dim f = {{1}, {2, 3}}

' error: inconsistent rank
Dim g = {1, {2}}
```

The format and requirements for the collection initializer in an array literal is exactly the same as that for the collection initializer in an array creation expression.

__Note.__ An array literal does not create the array in and of itself; instead, it is the reclassification of the expression into a value that causes the array to be created. For instance, the conversion `CType(new Integer() {1,2,3}, Short())` is not possible because there is no conversion from `Integer()` to `Short()`; but the expression `CType({1,2,3},Short())` is possible because it first reclassifies the array literal into the array creation expression `New Short() {1,2,3}`.


### Delegate-Creation Expressions

A delegate-creation expression is used to create a new instance of a delegate type. The argument of a delegate-creation expression must be an expression classified as a method pointer or a lambda method.

If the argument is a method pointer, one of the methods referenced by the method pointer must be applicable to the signature of the delegate type. A method `M` is applicable to a delegate type `D` if:

* `M` is not `Partial` or has a body.

* Both `M` and `D` are functions, or `D` is a subroutine.

* `M` and `D` have the same number of parameters.

* The parameter types of `M` each have a conversion from the type of the corresponding parameter type of `D`, and their modifiers (i.e. `ByRef`, `ByVal`) match.

* The return type of `M`, if any, has a conversion to the return type of `D`.

If the method pointer references a late-bound access, then the late-bound access is assumed to be to a function that has the same number of parameters as the delegate type.

If strict semantics are not being used and there is only one method referenced by the method pointer, but it is not applicable due to the fact that it has no parameters and the delegate type does, then the method is considered applicable and the parameters or return value are simply ignored. For example:

```vb
Delegate Sub F(x As Integer)

Module Test
    Sub M()
    End Sub

    Sub Main()
        ' Valid
        Dim x As F = AddressOf M
    End Sub
End Module
```

__Note.__ This relaxation is only allowed when strict semantics are not being used because of extension methods. Because extension methods are only considered if a regular method was not applicable, it is possible for an instance method with no parameters to hide an extension method with parameters for the purpose of delegate construction.

If more than one method referenced by the method pointer is applicable to the delegate type, then overload resolution is used to pick between the candidate methods. The types of the parameters to the delegate are used as the types of the arguments for the purposes of overload resolution. If no one method candidate is most applicable, a compile-time error occurs. In the following example, the local variable is initialized with a delegate that refers to the second `Square` method because that method is more applicable to the signature and return type of `DoubleFunc`.

```vb
Delegate Function DoubleFunc(x As Double) As Double

Module Test
    Function Square(x As Single) As Single
        Return x * x
    End Function 

    Function Square(x As Double) As Double
        Return x * x
    End Function

    Sub Main()
        Dim a As New DoubleFunc(AddressOf Square)
    End Sub
End Module
```

Had the second `Square` method not been present, the first `Square` method would have been chosen. If strict semantics are specified by the compilation environment or by `Option Strict`, then a compile-time error occurs if the most specific method referenced by the method pointer is narrower than the delegate signature. A method `M` is considered narrower than a delegate type `D` if:

* A parameter type of `M` has a widening conversion to the corresponding parameter type of `D`.

* Or, the return type, if any, of `M` has a narrowing conversion to the return type of `D`.

If type arguments are associated with the method pointer, only methods with the same number of type arguments are considered. If no type arguments are associated with the method pointer, type inference is used when matching signatures against a generic method. Unlike other normal type inference, the return type of the delegate is used when inferring type arguments, but return types are still not considered when determining the least generic overload. The following example shows both ways of supplying a type argument to a delegate-creation expression:

```vb
Delegate Function D(s As String, i As Integer) As Integer
Delegate Function E() As Integer

Module Test
    Public Function F(Of T)(s As String, t1 As T) As T
    End Function

    Public Function G(Of T)() As T
    End Function

    Sub Main()
        Dim d1 As D = AddressOf f(Of Integer)    ' OK, type arg explicit
        Dim d2 As D = AddressOf f                ' OK, type arg inferred

        Dim e1 As E = AddressOf g(Of Integer)    ' OK, type arg explicit
        Dim e2 As E = AddressOf g                ' OK, infer from return
  End Sub
End Module
```

In the above example, a non-generic delegate type was instantiated using a generic method. It is also possible to create an instance of a constructed delegate type using a generic method. For example:

```vb
Delegate Function Predicate(Of U)(u1 As U, u2 As U) As Boolean

Module Test
    Function Compare(Of T)(t1 As List(of T), t2 As List(of T)) As Boolean
        ...
    End Function

    Sub Main()
        Dim p As Predicate(Of List(Of Integer))
        p = AddressOf Compare(Of Integer)
    End Sub
End Module
```

If the argument to the delegate-creation expression is a lambda method, the lambda method must be applicable to the signature of the delegate type. A lambda method `L` is applicable to a delegate type `D` if:

* If `L` has parameters, `D` has the same number of parameters. (If `L` has no parameters, the parameters of `D` are ignored.)

* The parameter types of `L` each have a conversion to the type of the corresponding parameter type of `D`, and their modifiers (i.e. `ByRef`, `ByVal`) match.

* If `D` is a function, the return type of `L` has a conversion to the return type of `D`. (If `D` is a subroutine, the return value of `L` is ignored.)

If the parameter type of a parameter of `L` is omitted, then the type of the corresponding parameter in `D` is inferred; if the parameter of `L` has array or nullable name modifiers, a compile-time error results. Once all of the parameter types of `L` are available, then the type of the expression in the lambda method is inferred. For example:

```vb
Delegate Function F(x As Integer, y As Long) As Long

Module Test
    Sub Main()
        ' b inferred to Integer, c and return type inferred to Long
        Dim a As F = Function(b, c) b + c

        ' e and return type inferred to Integer, f inferred to Long
        Dim d As F = Function(e, f) e + CInt(f)
    End Sub
End Module
```

In some situations where delegate signature does not exactly match the lambda method or method signature, the .NET Framework may not support the delegate creation natively. In that situation, a lambda method expression is used to match the two methods. For example:

```vb
Delegate Function IntFunc(x As Integer) As Integer

Module Test
    Function SquareString(x As String) As String
        Return CInt(x) * CInt(x)
    End Function 

    Sub Main()
        ' The following two lines are equivalent
        Dim a As New IntFunc(AddressOf SquareString)
        Dim b As New IntFunc( _
            Function(x As Integer) CInt(SquareString(CStr(x))))
    End Sub
End Module
```

The result of a delegate-creation expression is a delegate instance that refers to the matching method with the associated target expression (if any) from the method pointer expression. If the target expression is typed as a value type, then the value type is copied onto the system heap because a delegate can only point to a method of an object on the heap. The method and object to which a delegate refers remain constant for the entire lifetime of the delegate. In other words, it is not possible to change the target or object of a delegate after it has been created.

### Anonymous Object-Creation Expressions

An object-creation expression with member initializers can also omit the type name entirely.

```antlr
AnonymousObjectCreationExpression
    : 'New' ObjectMemberInitializer
    ;
```

In that case, an anonymous type is constructed based on the types and names of the members initialized as a part of the expression. For example:

```vb
Module Test
    Sub Main()
        Dim Customer = New With { .Name = "John Smith", .Age = 34 }

        Console.WriteLine(Customer.Name)
    End Sub
End Module
```

The type created by an anonymous object-creation expression is a class that has no name, inherits directly from `Object`, and has a set of properties with the same name as the members assigned to in the member initializer list. The type of each property is inferred using the same rules as local variable type inference. Generated anonymous types also override `ToString`, returning a string representation of all members and their values. (The exact format of this string is beyond the scope of this specification).

By default, the properties generated by the anonymous type are read-write. It is possible to mark an anonymous type property as read-only by using the `Key` modifier. The `Key` modifier specifies that the field can be used to uniquely identify the value the anonymous type represents. In addition to making the property read-only, it also causes the anonymous type to override `Equals` and  `GetHashCode` and to implement the interface `System.IEquatable(Of T)` (filling in the anonymous type for `T`). The members are defined as follows:

`Function Equals(obj As Object) As Boolean` and `Function Equals(val As T) As Boolean` are implemented by validating that the two instances are of the same type and then comparing each `Key` member using `Object.Equals`. If all `Key` members are equal, then `Equals` returns `True`, otherwise `Equals` returns `False`.

`Function GetHashCode() As Integer` is implemented such that that if `Equals` is true for two instances of the anonymous type, then `GetHashCode` will return the same value. The hash starts with a seed value and then, for each `Key` member, in order multiplies the hash by 31 and adds the `Key` member's hash value (provided by `GetHashCode`) if the member is not a reference type or nullable value type with the value of `Nothing`.

For example, the type created in the statement:

```vb
Dim zipState = New With { Key .ZipCode = 98112, .State = "WA" }
```

creates a class that looks approximately like this (although exact implementation may vary):

```vb
Friend NotInheritable Class $Anonymous1
    Implements IEquatable(Of $Anonymous1)

    Private ReadOnly _zipCode As Integer
    Private _state As String

    Public Sub New(zipCode As Integer, state As String)
        _zipCode = zipcode
        _state = state
    End Sub

    Public ReadOnly Property ZipCode As Integer
        Get
            Return _zipCode
        End Get
    End Property

    Public Property State As String
        Get
            Return _state
        End Get
        Set (value As Integer)
            _state = value
        End Set
    End Property

    Public Overrides Function Equals(obj As Object) As Boolean
        Dim val As $Anonymous1 = TryCast(obj, $Anonymous1)
        Return Equals(val)
    End Function

    Public Overloads Function Equals(val As $Anonymous1) As Boolean _
        Implements IEquatable(Of $Anonymous1).Equals

        If val Is Nothing Then 
            Return False
        End If

        If Not Object.Equals(_zipCode, val._zipCode) Then 
            Return False
        End If

        Return True
    End Function

    Public Overrides Function GetHashCode() As Integer
        Dim hash As Integer = 0

        hash = hash Xor _zipCode.GetHashCode()

        Return hash
    End Function

    Public Overrides Function ToString() As String
        Return "{ Key .ZipCode = " & _zipCode & ", .State = " & _state & " }"
    End Function
End Class
```

To simplify the situation where an anonymous type is created from the fields of another type, field names can be inferred directly from expressions in the following cases:

* A simple name expression `x` infers the name `x`.

* A member access expression `x.y` infers the name `y`.

* A dictionary lookup expression `x!y` infers the name `y`.

* An invocation or index expression with no arguments `x()` infers the name `x`.

* An XML member access expression `x.<y>`, `x...<y>`, `x.@y` infers the name `y`.

* An XML member access expression that is the target of a member access expression `x.<y>.z` infers the name `z`.

* An XML member access expression that is the target of an invocation or index expression with no arguments `x.<y>.z()` infers the name `z`.

* An XML member access expression that is the target of an invocation or index expression `x.<y>(0)` infers the name `y`.

The initializer is interpreted as an assignment of the expression to the inferred name. For example, the following initializers are equivalent:

```vb
Class Address
    Public Street As String
    Public City As String
    Public State As String
    Public ZIP As String
End Class

Class C1
    Sub Test(a As Address)
        Dim cityState1 = New With { .City = a.City, .State = a.State }
        Dim cityState2 = New With { a.City, a.State }
    End Sub
End Class
```

If a member name is inferred that conflicts with an existing member of the type, such as `GetHashCode`, then a compile time error occurs. Unlike regular member initializers, anonymous object-creation expressions do not allow member initializers to have circular references, or to refer to a member before it has been initialized. For example:

```vb
Module Test
    Sub Main()
        ' Error: Circular references
        Dim x = New With { .a = .b, .b = .a }

        ' Error: Referring to .b before it has been assigned to
        Dim y = New With { .a = .b, .b = 10 }

        ' Error: Referring to .a before it has been assigned to
        Dim z = New With { .a = .a }
    End Sub
End Module
```

If two anonymous class creation expressions occur within the same method and yield the same resulting shape -- if the property order, property names, and property types all match -- they will both refer to the same anonymous class. The method scope of an instance or shared member variable with an initializer is the constructor in which the variable is initialized.

__Note.__ It is possible that a compiler may choose to unify anonymous types further, such as at the assembly level, but this cannot be relied upon at this time.


## Cast Expressions

A cast expression coerces an expression to a given type. Specific cast keywords coerce expressions into the primitive types. Three general cast keywords, `CType`, `TryCast` and `DirectCast`, coerce an expression into a type.

```antlr
CastExpression
    : 'DirectCast' OpenParenthesis Expression Comma TypeName CloseParenthesis
    | 'TryCast' OpenParenthesis Expression Comma TypeName CloseParenthesis
    | 'CType' OpenParenthesis Expression Comma TypeName CloseParenthesis
    | CastTarget OpenParenthesis Expression CloseParenthesis
    ;

CastTarget
    : 'CBool' | 'CByte' | 'CChar'  | 'CDate'  | 'CDec' | 'CDbl' | 'CInt'
    | 'CLng'  | 'CObj'  | 'CSByte' | 'CShort' | 'CSng' | 'CStr' | 'CUInt'
    | 'CULng' | 'CUShort'
    ;
```

`DirectCast` and `TryCast` have special behaviors. Because of this, they only support native conversions. Additionally, the target type in a `TryCast` expression cannot be a value type. User-defined conversion operators are not considered when `DirectCast` or `TryCast` is used. (__Note.__ The conversion set that `DirectCast` and `TryCast` support are restricted because they implement "native CLR" conversions. The purpose of `DirectCast` is to provide the functionality of the "unbox" instruction, while the purpose of `TryCast` is to provide the functionality of the "isinst" instruction. Since they map onto CLR instructions, supporting conversions not directly supported by the CLR would defeat the intended purpose.)

`DirectCast` converts expressions that are typed as `Object` differently than `CType`. When converting an expression of type `Object` whose run-time type is a primitive value type, `DirectCast` throws a `System.InvalidCastException` exception if the specified type is not the same as the run-time type of the expression or a `System.NullReferenceException` if the expression evaluates to `Nothing`. (__Note.__ As noted above, `DirectCast` maps directly onto the CLR instruction "unbox" when the type of the expression is `Object`. In contrast, `CType` turns into a call to a runtime helper to do the conversion so that conversions between primitive types can be supported. In the case when an `Object` expression is being converted to a primitive value type and the type of the actual instance match the target type, `DirectCast` will be significantly faster than `CType`.)

`TryCast` converts expressions but does not throw an exception if the expression cannot be converted to the target type. Instead, `TryCast` will result in `Nothing` if the expression cannot be converted at runtime. (__Note.__ As noted above, `TryCast` maps directly onto the CLR instruction "isinst". By combining the type check and the conversion into a single operation, `TryCast` can be cheaper than doing a `TypeOf ... Is` and then a `CType`.)

For example:

```vb
Interface ITest
    Sub Test()
End Interface

Module Test
    Sub Convert(o As Object)
        Dim i As ITest = TryCast(o, ITest)

        If i IsNot Nothing Then
            i.Test()
        End If
    End Sub
End Module
```

If no conversion exists from the type of the expression to the specified type, a compile-time error occurs. Otherwise, the expression is classified as a value and the result is the value produced by the conversion.


## Operator Expressions

There are two kinds of operators. *Unary operators* take one operand and use prefix notation (for example, `-x`). *Binary operators* take two operands and use infix notation (for example, `x + y`). With the exception of the relational operators, which always result in `Boolean`, an operator defined for a particular type results in that type. The operands to an operator must always be classified as a value; the result of an operator expression is classified as a value.

```antlr
OperatorExpression
    : ArithmeticOperatorExpression
    | RelationalOperatorExpression
    | LikeOperatorExpression
    | ConcatenationOperatorExpression
    | ShortCircuitLogicalOperatorExpression
    | LogicalOperatorExpression
    | ShiftOperatorExpression
    | AwaitOperatorExpression
    ;
```

### Operator Precedence and Associativity

When an expression contains multiple binary operators, the *precedence* of the operators controls the order in which the individual binary operators are evaluated. For example, the expression `x + y * z` is evaluated as `x + (y * z)` because the `*` operator has higher precedence than the `+` operator. The following table lists the binary operators in descending order of precedence:


| __Category__     | __Operators__                                          | 
|------------------|--------------------------------------------------------|
| Primary          | All non-operator expressions                           |
| Await            | `Await`                                                |
| Exponentiation   | `^`                                                    |
| Unary negation   | `+`, `-`                                               |
| Multiplicative   | `*`, `/`                                               |
| Integer division | `\`                                                    |
| Modulus          | `Mod`                                                  |
| Additive         | `+`, `-`                                               |
| Concatenation    | `&`                                                    |
| Shift            | `<<`, `>>`                                             |
| Relational       | `=`, `<>`, `<`, `>`, `<=`, `>=`, `Like`, `Is`, `IsNot` |
| Logical NOT      | `Not`                                                  |
| Logical AND      | `And`, `AndAlso`                                       |
| Logical OR       | `Or`, `OrElse`                                         |
| Logical XOR      | `Xor`                                                  |

When an expression contains two operators with the same precedence, the *associativity* of the operators controls the order in which the operations are performed. All binary operators are left-associative, meaning that operations are performed from left to right. Precedence and associativity can be controlled using parenthetical expressions.

### Object Operands

In addition to the regular types supported by each operator, all operators support operands of type `Object`. Operators applied to `Object` operands are handled similarly to method calls made on `Object` values: a late-bound method call might be chosen, in which case the run-time type of the operands, rather than the compile-time type, determines the validity and type of the operation. If strict semantics are specified by the compilation environment or by `Option Strict`, any operators with operands of type `Object` cause a compile-time error, except for the `TypeOf...Is`, `Is` and `IsNot` operators.

When operator resolution determines that an operation should be performed late-bound, the outcome of the operation is the result of applying the operator to the operand types if the run-time types of the operands are types that are supported by the operator. The value `Nothing` is treated as the default value of the type of the other operand in a binary operator expression. In a unary operator expression, or if both operands are `Nothing` in a binary operator expression, the type of the operation is `Integer` or the only result type of the operator, if the operator does not result in `Integer`. The result of the operation is always then cast back to `Object`. If the operand types have no valid operator, a `System.InvalidCastException` exception is thrown. Conversions at run time are done without regard to whether they are implicit or explicit.

If the result of a numeric binary operation would produce an overflow exception (regardless of whether integer overflow checking is on or off), then the result type is promoted to the next wider numeric type, if possible. For example, consider the following code:

```vb
Module Test
    Sub Main()
        Dim o As Object = CObj(CByte(2)) * CObj(CByte(255))

        Console.WriteLine(o.GetType().ToString() & " = " & o)
    End Sub
End Module
```

It prints the following result:

```
System.Int16 = 512
```

If no wider numeric type is available to hold the number, a `System.OverflowException` exception is thrown.

### Operator Resolution

Given an operator type and a set of operands, operator resolution determines which operator to use for the operands. When resolving operators, user-defined operators will be considered first, using the following steps:

1. First, all of the candidate operators are collected. The candidate operators are all of the user-defined operators of the particular operator type in the source type and all of the user-defined operators of the particular type in the target type. If the source type and destination type are related, common operators are only considered once.

2. Then, overload resolution is applied to the operators and operands to select the most specific operator. In the case of binary operators, this may result in a late-bound call.

When collecting the candidate operators for a type `T?`, the operators of type `T` are used instead. Any of `T`'s user-defined operators that involve only non-nullable value types are also lifted. A lifted operator uses the nullable version of any value types, with the exception the return types of `IsTrue` and `IsFalse` (which must be `Boolean`). Lifted operators are evaluated by converting the operands to their non-nullable version, then evaluating the user-defined operator and then converting the result type to its nullable version. If ether operand is `Nothing`, the result of the expression is a value of `Nothing` typed as the nullable version of the result type. For example:

```vb
Structure T
    ...
End Structure

Structure S
    Public Shared Operator +(ByVal op1 As S, ByVal op2 As T) As T
        ...
    End Operator
End Structure

Module Test
    Sub Main()
        Dim x As S?
        Dim y, z As T?

        ' Valid, as S + T = T is lifted to S? + T? = T?
        z = x + y 
    End Sub
End Module
```

If the operator is a binary operator and one of the operands is reference type, the operator is also lifted, but any binding to the operator produces an error. For example:

```vb
Structure S1
    Public F1 As Integer

    Public Shared Operator +(left As S1, right As String) As S1
       ...
    End Operator
End Structure

Module Test
    Sub Main()
        Dim a? As S1
        Dim s As String
        
        ' Error: '+' is not defined for S1? and String
        a = a + s
    End Sub
End Module
```

__Note.__ This rule exists because there has been consideration whether we wish to add null-propagating reference types in a future version, in which case the behavior in the case of binary operators between the two types would change.

As with conversions, user-defined operators are always preferred over lifted operators.

When resolving overloaded operators, there may be differences between classes defined in Visual Basic and those defined in other languages:

* In other languages, `Not`, `And`, and `Or` may be overloaded both as logical operators and bitwise operators. Upon import from an external assembly, either form is accepted as a valid overload for these operators. However, for a type which defines both logical and bitwise operators, only the bitwise implementation will be considered.

* In other languages, `>>` and `<<` may be overloaded both as signed operators and unsigned operators. Upon import from an external assembly, either form is accepted as a valid overload. However, for a type which defines both signed and unsigned operators, only the signed implementation will be considered.

* If no user-defined operator is most specific to the operands, then intrinsic operators will be considered. If no intrinsic operator is defined for the operands and either operand has type Object then the operator will be resolved late-bound; otherwise,  a compile-time error results.

In prior versions of Visual Basic, if there was exactly one operand of type Object, and no applicable user-defined operators, and no applicable intrinsic operators, then it was an error. As of Visual Basic 11, it is now resolved late-bound. For example:

```vb
Module Module1
  Sub Main()
      Dim p As Object = Nothing
      Dim U As New Uri("http://www.microsoft.com")
      Dim j = U * p  ' is now resolved late-bound
   End Sub
End Module
```

A type `T` that has an intrinsic operator also defines that same operator for `T?`. The result of the operator on `T?` will be the same as for `T`, except that if either operand is `Nothing`, the result of the operator will be `Nothing` (i.e. the null value is propagated). For the purposes of resolving the type of an operation, the `?` is removed from any operands that have them, the type of the operation is determined, and a `?` is added to the type of the operation if any of the operands were nullable value types. For example:

```vb
Dim v1? As Integer = 10
Dim v2 As Long = 20

' Type of operation will be Long?
Console.WriteLine(v1 + v2)
```

Each operator lists the intrinsic types it is defined for and the type of the operation performed given the operand types. The result of type of a intrinsic operation follows these general rules:

* If all operands are of the same type, and the operator is defined for the type, then no conversion occurs and the operator for that type is used.

* Any operand whose type is not defined for the operator is converted using the following steps and the operator is resolved against the new types:

  * The operand is converted to the next widest type that is defined for both the operator and the operand and to which it is implicitly convertible.

  * If there is no such type, then the operand is converted to the next narrowest type that is defined for both the operator and the operand and to which it is implicitly convertible.

  * If there is no such type or the conversion cannot occur, a compile-time error occurs.

* Otherwise, the operands are converted to the wider of the operand types and the operator for that type is used. If the narrower operand type cannot be implicitly converted to the wider operator type, a compile-time error occurs.

Despite these general rules, however, there are a number of special cases called out in the operator results tables.

__Note.__ For formatting reasons, the operator type tables abbreviate the predefined names to their first two characters. So "By" is `Byte`, "UI" is `UInteger`, "St" is `String`, etc. "Err" means that there is no operation defined for the given operand types.

## Arithmetic Operators

The `*`, `/`, `\`, `^`, `Mod`, `+`, and `-` operators are the *arithmetic operators*.

```antlr
ArithmeticOperatorExpression
    : UnaryPlusExpression
    | UnaryMinusExpression
    | AdditionOperatorExpression
    | SubtractionOperatorExpression
    | MultiplicationOperatorExpression
    | DivisionOperatorExpression
    | ModuloOperatorExpression
    | ExponentOperatorExpression
    ;
```

Floating-point arithmetic operations may be performed with higher precision than the result type of the operation. For example, some hardware architectures support an "extended" or "long double" floating-point type with greater range and precision than the `Double` type, and implicitly perform all floating-point operations using this higher-precision type. Hardware architectures can be made to perform floating-point operations with less precision only at excessive cost in performance; rather than require an implementation to forfeit both performance and precision, Visual Basic allows the higher-precision type to be used for all floating-point operations. Other than delivering more precise results, this rarely has any measurable effects. However, in expressions of the form `x * y / z`, where the multiplication produces a result that is outside the `Double` range, but the subsequent division brings the temporary result back into the `Double` range, the fact that the expression is evaluated in a higher-range format may cause a finite result to be produced instead of infinity.


### Unary Plus Operator

```antlr
UnaryPlusExpression
    : '+' Expression
    ;
```

The unary plus operator is defined for the `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Single`, `Double`, and `Decimal` types.

__Operation Type:__


| __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| Sh | SB | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 


### Unary Minus Operator

```antlr
UnaryMinusExpression
    : '-' Expression
    ;
```

The unary minus operator is defined for the following types:

`SByte`, `Short`, `Integer`, and `Long`. The result is computed by subtracting the operand from zero. If integer overflow checking is on and the value of the operand is the maximum negative `SByte`, `Short`, `Integer`, or `Long`, a `System.OverflowException` exception is thrown. Otherwise, if the value of the operand is the maximum negative `SByte`, `Short`, `Integer`, or `Long`, the result is that same value, and the overflow is not reported.

`Single` and `Double`. The result is the value of the operand with its sign inverted, including the values 0 and Infinity. If the operand is NaN, the result is also NaN.

`Decimal`. The result is computed by subtracting the operand from zero.

__Operation Type:__

| __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| Sh | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 


### Addition Operator

The addition operator computes the sum of the two operands.

```antlr
AdditionOperatorExpression
    : Expression '+' LineTerminator? Expression
    ;
```

The addition operator is defined for the following types:

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, and `Long`. If integer overflow checking is on and the sum is outside the range of the result type, a `System.OverflowException` exception is thrown. Otherwise, overflows are not reported, and any significant high-order bits of the result are discarded.

* `Single` and `Double`. The sum is computed according to the rules of IEEE 754 arithmetic.

* `Decimal`. If the resulting value is too large to represent in the decimal format, a `System.OverflowException` exception is thrown. If the result value is too small to represent in the decimal format, the result is 0.

* `String`. The two `String` operands are concatenated together.

* `Date`. The `System.DateTime` type defines overloaded addition operators. Because `System.DateTime` is equivalent to the intrinsic `Date` type, these operators is also available on the `Date` type.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| __Bo__ | Sh | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __In__ |    |    |    |    |    | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __Lo__ |    |    |    |    |    |    |    | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __UL__ |    |    |    |    |    |    |    |    | UL | De | Si | Do | Err | Err | Do | Ob | 
| __De__ |    |    |    |    |    |    |    |    |    | De | Si | Do | Err | Err | Do | Ob | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Si | Do | Err | Err | Do | Ob | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do | Ob | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | St  | Err | St | Ob | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | St  | St | Ob | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | St | Ob | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |    | Ob | 


### Subtraction Operator

The subtraction operator subtracts the second operand from the first operand.

```antlr
SubtractionOperatorExpression
    : Expression '-' LineTerminator? Expression
    ;
```

The subtraction operator is defined for the following types:

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, and `Long`. If integer overflow checking is on and the difference is outside the range of the result type, a `System.OverflowException` exception is thrown. Otherwise, overflows are not reported, and any significant high-order bits of the result are discarded.

* `Single` and `Double`. The difference is computed according to the rules of IEEE 754 arithmetic.

* `Decimal`. If the resulting value is too large to represent in the decimal format, a `System.OverflowException` exception is thrown. If the result value is too small to represent in the decimal format, the result is 0.

* `Date`. The `System.DateTime` type defines overloaded subtraction operators. Because `System.DateTime` is equivalent to the intrinsic `Date` type, these operators is also available on the `Date` type.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Sh | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __In__ |    |    |    |    |    | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | De | Si | Do | Err | Err | Do  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Si | Do | Err | Err | Do  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Do  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


### Multiplication Operator

The multiplication operator computes the product of two operands.

```antlr
MultiplicationOperatorExpression
    : Expression '*' LineTerminator? Expression
    ;
```

The multiplication operator is defined for the following types:

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, and `Long`. If integer overflow checking is on and the product is outside the range of the result type, a `System.OverflowException` exception is thrown. Otherwise, overflows are not reported, and any significant high-order bits of the result are discarded.

* `Single` and `Double`. The product is computed according to the rules of IEEE 754 arithmetic.

* `Decimal`. If the resulting value is too large to represent in the decimal format, a `System.OverflowException` exception is thrown. If the result value is too small to represent in the decimal format, the result is 0.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Sh | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __In__ |    |    |    |    |    | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | De | Si | Do | Err | Err | Do  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Si | Do | Err | Err | Do  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Do  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


### Division Operators

Division operators compute the quotient of two operands. There are two division operators: the regular (floating-point) division operator and the integer division operator.

```antlr
DivisionOperatorExpression
    : FPDivisionOperatorExpression
    | IntegerDivisionOperatorExpression
    ;

FPDivisionOperatorExpression
    : Expression '/' LineTerminator? Expression
    ;

IntegerDivisionOperatorExpression
    : Expression '\\' LineTerminator? Expression
    ;
```

The regular division operator is defined for the following types:

* `Single` and `Double`. The quotient is computed according to the rules of IEEE 754 arithmetic.

* `Decimal`. If the value of the right operand is zero, a `System.DivideByZeroException` exception is thrown. If the resulting value is too large to represent in the decimal format, a `System.OverflowException` exception is thrown. If the result value is too small to represent in the decimal format, the result is zero. The scale of the result, before any rounding, is the closest scale to the preferred scale which will preserve a result equal to the exact result.  The preferred scale is the scale of the first operand less the scale of the second operand.

According to normal operator resolution rules, regular division purely between operands of types such as `Byte`, `Short`, `Integer`, and `Long` would cause both operands to be converted to type `Decimal`. However, when doing operator resolution on the division operator when neither type is `Decimal`, `Double` is considered narrower than `Decimal`. This convention is followed because `Double` division is more efficient than `Decimal` division.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Do | Do | Do | Do | Do | Do | Do | Do | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __SB__ |    | Do | Do | Do | Do | Do | Do | Do | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __By__ |    |    | Do | Do | Do | Do | Do | Do | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __Sh__ |    |    |    | Do | Do | Do | Do | Do | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __US__ |    |    |    |    | Do | Do | Do | Do | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __In__ |    |    |    |    |    | Do | Do | Do | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __UI__ |    |    |    |    |    |    | Do | Do | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Do | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | Do | De | Si | Do | Err | Err | Do  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | De | Si | Do | Err | Err | Do  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Si | Do | Err | Err | Do  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Do  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 

The integer division operator is defined for `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, and `Long`. If the value of the right operand is zero, a `System.DivideByZeroException` exception is thrown. The division rounds the result towards zero, and the absolute value of the result is the largest possible integer that is less than the absolute value of the quotient of the two operands. The result is zero or positive when the two operands have the same sign, and zero or negative when the two operands have opposite signs. If the left operand is the maximum negative `SByte`, `Short`, `Integer`, or `Long`, and the right operand is `-1`, an overflow occurs; if integer overflow checking is on, a `System.OverflowException` exception is thrown. Otherwise, the overflow is not reported and the result is instead the value of the left operand.

__Note.__ As the two operands for unsigned types will always be zero or positive, the result is always zero or positive.  As the result of the expression will always be less than or equal to the largest of the two operands, it is not possible for an overflow to occur.  As such integer overflow checking is not performed for integer divide with two unsigned integers. The result is the type as that of the left operand.


__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Sh | SB | Sh | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __In__ |    |    |    |    |    | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Lo | Err | Err | Lo  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Lo  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


### Mod Operator

The `Mod` (modulo) operator computes the remainder of the division between two operands.

```antlr
ModuloOperatorExpression
    : Expression 'Mod' LineTerminator? Expression
    ;
```

The `Mod` operator is defined for the following types:

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong` and `Long`. The result of `x Mod y` is the value produced by `x - (x \ y) * y`. If `y` is zero, a `System.DivideByZeroException` exception is thrown. The modulo operator never causes an overflow.

* `Single` and `Double`. The remainder is computed according to the rules of IEEE 754 arithmetic.

* `Decimal`. If the value of the right operand is zero, a `System.DivideByZeroException` exception is thrown. If the resulting value is too large to represent in the decimal format, a `System.OverflowException` exception is thrown. If the result value is too small to represent in the decimal format, the result is zero.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Sh | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __In__ |    |    |    |    |    | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | De | Si | Do | Err | Err | Do  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Si | Do | Err | Err | Do  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Do  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


### Exponentiation Operator

The exponentiation operator computes the first operand raised to the power of the second operand.

```antlr
ExponentOperatorExpression
    : Expression '^' LineTerminator? Expression
    ;
```

The exponentiation operator is defined for type `Double`. The value is computed according to the rules of IEEE 754 arithmetic.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __SB__ |    | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __By__ |    |    | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __Sh__ |    |    |    | Do | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __US__ |    |    |    |    | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __In__ |    |    |    |    |    | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __UI__ |    |    |    |    |    |    | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | Do | Do | Do | Err | Err | Do  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Do | Do | Err | Err | Do  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Do  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


## Relational Operators

The *relational operators* compare values to one other. The comparison operators are `=`, `<>`, `<`, `>`, `<=`, and `>=`.

```antlr
RelationalOperatorExpression
    : Expression '=' LineTerminator? Expression
    | Expression '<' '>' LineTerminator? Expression
    | Expression '<' LineTerminator? Expression
    | Expression '>' LineTerminator? Expression
    | Expression '<' '=' LineTerminator? Expression
    | Expression '>' '=' LineTerminator? Expression
    ;
```

All of the relational operators result in a `Boolean` value.

The relational operators have the following general meaning:

* The `=` operator tests whether the two operands are equal.

* The `<>` operator tests whether the two operands are not equal.

* The `<` operator tests whether the first operand is less than the second operand.

* The `>` operator tests whether the first operand is greater than the second operand.

* The `<=` operator tests whether the first operand is less than or equal to the second operand.

* The `>=` operator tests whether the first operand is greater than or equal to the second operand.

The relational operators are defined for the following types:

* `Boolean`. The operators compare the truth values of the two operands. `True` is considered to be less than `False`, which matches with their numeric values.

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, and `Long`. The operators compare the numeric values of the two integral operands.

* `Single` and `Double`. The operators compare the operands according to the rules of the IEEE 754 standard.

* `Decimal`. The operators compare the numeric values of the two decimal operands.

* `Date`. The operators return the result of comparing the two date/time values.

* `Char`. The operators return the result of comparing the two Unicode values.

* `String`. The operators return the result of comparing the two values using either a binary comparison or a text comparison. The comparison used is determined by the compilation environment and the `Option Compare` statement. A binary comparison determines whether the numeric Unicode value of each character in each string is the same. A text comparison does a Unicode text comparison based on the current culture in use on the .NET Framework. When doing a string comparison, a null value is equivalent to the string literal `""`.

__Operation Type:__
        
|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| __Bo__ | Bo | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Bo | Ob | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __In__ |    |    |    |    |    | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __Lo__ |    |    |    |    |    |    |    | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __UL__ |    |    |    |    |    |    |    |    | UL | De | Si | Do | Err | Err | Do | Ob | 
| __De__ |    |    |    |    |    |    |    |    |    | De | Si | Do | Err | Err | Do | Ob | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Si | Do | Err | Err | Do | Ob | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do | Ob | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Da  | Err | Da | Ob | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Ch  | St | Ob | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | St | Ob | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |    | Ob | 


## Like Operator

The `Like` operator determines whether a string matches a given pattern.

```antlr
LikeOperatorExpression
    : Expression 'Like' LineTerminator? Expression
    ;
```

The `Like` operator is defined for the `String` type. The first operand is the string being matched, and the second operand is the pattern to match against. The pattern is made up of Unicode characters. The following character sequences have special meanings:

* The character `?` matches any single character.

* The character `*` matches zero or more characters.

* The character `#` matches any single digit (0-9).

* A list of characters surrounded by brackets (`[ab...]`) matches any single character in the list.

* A list of characters surrounded by brackets and prefixed by an exclamation point (`[!ab...]`) matches any single character not in the character list.

* Two characters in a character list separated by a hyphen (`-`) specify a range of Unicode characters starting with the first character and ending with the second character. If the second character is not later in the sort order than the first character, a run-time exception occurs. A hyphen that appears at the beginning or end of a character list specifies itself.

To match the special characters left bracket (`[`), question mark (`?`), number sign (`#`), and asterisk (`*`), brackets must enclose them. The right bracket (`]`) cannot be used within a group to match itself, but it can be used outside a group as an individual character. The character sequence `[]` is considered to be the string literal `""`. 

Note that character comparisons and ordering for character lists are dependent on the type of comparisons being used. If binary comparisons are being used, character comparisons and ordering are based on the numeric Unicode values. If text comparisons are being used, character comparisons and ordering are based on the current locale being used on the .NET Framework.

In some languages, special characters in the alphabet represent two separate characters and vice versa. For example, several languages use the character `æ` to represent the characters `a` and `e` when they appear together, while the characters `^` and `O` can be used to represent the character `Ô`. When using text comparisons, the `Like` operator recognizes such cultural equivalences. In that case, an occurrence of the single special character in either pattern or string matches the equivalent two-character sequence in the other string. Similarly, a single special character in pattern enclosed in brackets (by itself, in a list, or in a range) matches the equivalent two-character sequence in the string and vice versa.

In a `Like` expression where both operands are `Nothing` or one operand has an intrinsic conversion to `String` and the other operand is `Nothing`, `Nothing` is treated as if it were the empty string literal `""`.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|--------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| __Bo__ | St | St | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __SB__ |    | St | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __By__ |    |    | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __Sh__ |    |    |    | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __US__ |    |    |    |    | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __In__ |    |    |    |    |    | St | St | St | St | St | St | St | St | St | St | Ob | 
| __UI__ |    |    |    |    |    |    | St | St | St | St | St | St | St | St | St | Ob | 
| __Lo__ |    |    |    |    |    |    |    | St | St | St | St | St | St | St | St | Ob | 
| __UL__ |    |    |    |    |    |    |    |    | St | St | St | St | St | St | St | Ob | 
| __De__ |    |    |    |    |    |    |    |    |    | St | St | St | St | St | St | Ob | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | St | St | St | St | St | Ob | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | St | St | St | St | Ob | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | St | St | St | Ob | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |    | St | St | Ob | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |    |    | St | Ob | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    | Ob | 


## Concatenation Operator

```antlr
ConcatenationOperatorExpression
    : Expression '&' LineTerminator? Expression
    ;
```

The *concatenation operator* is defined for all of the intrinsic types, including the nullable versions of the intrinsic value types. It is also defined for concatenation between the types mentioned above and `System.DBNull`, which is treated as a `Nothing` string. The concatenation operator converts all of its operands to `String`; in the expression, all conversions to `String` are considered to be widening, regardless of whether strict semantics are used. A `System.DBNull` value is converted to the literal `Nothing` typed as `String`. A nullable value type whose value is `Nothing` is also converted to the literal `Nothing` typed as `String`, rather than throwing a run-time error.

A concatenation operation results in a string that is the concatenation of the two operands in order from left to right. The value `Nothing` is treated as if it were the empty string literal `""`.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|--------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| __Bo__ | St | St | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __SB__ |    | St | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __By__ |    |    | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __Sh__ |    |    |    | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __US__ |    |    |    |    | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __In__ |    |    |    |    |    | St | St | St | St | St | St | St | St | St | St | Ob | 
| __UI__ |    |    |    |    |    |    | St | St | St | St | St | St | St | St | St | Ob | 
| __Lo__ |    |    |    |    |    |    |    | St | St | St | St | St | St | St | St | Ob | 
| __UL__ |    |    |    |    |    |    |    |    | St | St | St | St | St | St | St | Ob | 
| __De__ |    |    |    |    |    |    |    |    |    | St | St | St | St | St | St | Ob | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | St | St | St | St | St | Ob | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | St | St | St | St | Ob | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | St | St | St | Ob | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |    | St | St | Ob | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |    |    | St | Ob | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    | Ob | 


## Logical Operators

The `And`, `Not`, `Or`, and `Xor` operators are called the logical operators.

```antlr
LogicalOperatorExpression
    : 'Not' Expression
    | Expression 'And' LineTerminator? Expression
    | Expression 'Or' LineTerminator? Expression
    | Expression 'Xor' LineTerminator? Expression
    ;
```

The logical operators are evaluated as follows:

* For the `Boolean` type:

  * A logical `And` operation is performed on its two operands.

  * A logical `Not` operation is performed on its operand.

  * A logical `Or` operation is performed on its two operands.

  * A logical exclusive-`Or` operation is performed on its two operands.

* For `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, and all enumerated types, the specified operation is performed on each bit of the binary representation of the two operand(s):

  * `And`: The result bit is 1 if both bits are 1; otherwise the result bit is 0.

  * `Not`: The result bit is 1 if the bit is 0; otherwise the result bit is 1.

  * `Or`: The result bit is 1 if either bit is 1; otherwise the result bit is 0.

  * `Xor`: The result bit is 1 if either bit is 1 but not both bits; otherwise the result bit is 0 (that is, 1 `Xor` 0 = 1, 1 `Xor` 1 = 0).

* When the logical operators `And` and `Or` are lifted for the type `Boolean?`, they are extended to encompass three-valued Boolean logic as such:

  * `And` evaluates to true if both operands are true; false if one of the operands is false; `Nothing` otherwise.

  * `Or` evaluates to true if either operand is true; false is both operands are false; `Nothing` otherwise.

For example:

```vb
Module Test
    Sub Main()
        Dim x?, y? As Boolean

        x = Nothing
        y = True 

        If x Or y Then
            ' Will execute
        End If
    End Sub
End Module
```

__Note.__ Ideally, the logical operators `And` and `Or` would be lifted using three-valued logic for any type that can be used in a Boolean expression (i.e. a type that implements `IsTrue` and `IsFalse`), in the same way that `AndAlso` and `OrElse` short circuit across any type that can be used in a Boolean expression. Unfortunately, three-valued lifting is only applied to `Boolean?`, so user-defined types that desire three-valued logic must do so manually by defining `And` and `Or` operators for their nullable version.

No overflows are possible from these operations. The enumerated type operators do the bitwise operation on the underlying type of the enumerated type, but the return value is the enumerated type.

__Not Operation Type:__

| __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| Bo | SB | By | Sh | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo | Ob | 

__And, Or, Xor Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Bo | SB | Sh | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Bo  | Ob  | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __In__ |    |    |    |    |    | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Lo | Err | Err | Lo  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Lo  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


### Short-circuiting Logical Operators

The `AndAlso` and `OrElse` operators are the short-circuiting versions of the `And` and `Or` logical operators.

```antlr
ShortCircuitLogicalOperatorExpression
    : Expression 'AndAlso' LineTerminator? Expression
    | Expression 'OrElse' LineTerminator? Expression
    ;
```

Because of their short circuiting behavior, the second operand is not evaluated at run time if the operator result is known after evaluating the first operand.

The short-circuiting logical operators are evaluated as follows:

* If the first operand in an `AndAlso` operation evaluates to `False` or returns True from its `IsFalse` operator, the expression returns its first operand. Otherwise, the second operand is evaluated and a logical `And` operation is performed on the two results.

* If the first operand in an `OrElse` operation evaluates to `True` or returns True from its `IsTrue` operator, the expression returns its first operand. Otherwise, the second operand is evaluated and a logical `Or` operation is performed on its two results.

The `AndAlso` and `OrElse` operators are defined for the type `Boolean`, or for any type `T` that overloads the following operators:

```vb
Public Shared Operator IsTrue(op As T) As Boolean
Public Shared Operator IsFalse(op As T) As Boolean
```

as well as overloading the corresponding `And` or `Or` operator:

```vb
Public Shared Operator And(op1 As T, op2 As T) As T
Public Shared Operator Or(op1 As T, op2 As T) As T
```

When evaluating the `AndAlso` or `OrElse` operators, the first operand is evaluated only once, and the second operand is either not evaluated or evaluated exactly once. For example, consider the following code:

```vb
Module Test
    Function TrueValue() As Boolean
        Console.Write(" True")
        Return True
    End Function

    Function FalseValue() As Boolean
        Console.Write(" False")
        Return False
    End Function

    Sub Main()
        Console.Write("And:")
        If FalseValue() And TrueValue() Then
        End If
        Console.WriteLine()

        Console.Write("Or:")
        If TrueValue() Or FalseValue() Then
        End If
        Console.WriteLine()

        Console.Write("AndAlso:")
        If FalseValue() AndAlso TrueValue() Then
        End If
        Console.WriteLine()

        Console.Write("OrElse:")
        If TrueValue() OrElse FalseValue() Then
        End If
        Console.WriteLine()
    End Sub
End Module
```

It prints the following result:

```
And: False True
Or: True False
AndAlso: False
OrElse: True
```

In the lifted form of the `AndAlso` and `OrElse` operators, if the first operand was a null `Boolean?`, then the second operand is evaluated but the result is always a null `Boolean?`.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __SB__ |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __By__ |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __Sh__ |    |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __US__ |    |    |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __In__ |    |    |    |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __UI__ |    |    |    |    |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Bo | Bo | Err | Err | Bo  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Bo | Err | Err | Bo  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Bo  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


## Shift Operators

The binary operators `<<` and `>>` perform bit shifting operations.

```antlr
ShiftOperatorExpression
    : Expression '<' '<' LineTerminator? Expression
    | Expression '>' '>' LineTerminator? Expression
    ;
```

The operators are defined for the `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong` and `Long` types. Unlike the other binary operators, the result type of a shift operation is determined as if the operator was a unary operator with just the left operand. The type of the right operand must be implicitly convertible to `Integer` and is not used in determining the result type of the operation.

The `<<` operator causes the bits in the first operand to be shifted left the number of places specified by the shift amount. The high-order bits outside the range of the result type are discarded and the low-order vacated bit positions are zero-filled.

The `>>` operator causes the bits in the first operand to be shifted right the number of places specified by the shift amount. The low-order bits are discarded and the high-order vacated bit positions are set to zero if the left operand is positive or to one if negative. If the left operand is of type `Byte`, `UShort`, `UInteger`, or `ULong` the vacant high-order bits are zero-filled.

The shift operators shift the bits of the underlying representation of the first operand by the amount of the second operand. If the value of the second operand is greater than the number of bits in the first operand, or is negative, then the shift amount is computed as `RightOperand And SizeMask` where `SizeMask` is:

| __LeftOperand Type__  | __SizeMask__ | 
|-----------------------|--------------|
| `Byte`, `SByte`       | 7 (`&H7`)    | 
| `UShort`, `Short`     | 15 (`&HF`)   | 
| `UInteger`, `Integer` | 31 (`&H1F`)  | 
| `ULong`, `Long`       | 63 (`&H3F`)  | 

If the shift amount is zero, the result of the operation is identical to the value of the first operand. No overflows are possible from these operations.

__Operation Type:__


| __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| Sh | SB | By | Sh | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo | Ob | 


## Boolean Expressions

A Boolean expression is an expression that can be tested to see if it is true or if it is false.

```antlr
BooleanExpression
    : Expression
    ;
```

A type `T` can be used in a Boolean expression if, in order of preference:

* `T` is `Boolean` or `Boolean?`

* `T` has a widening conversion to `Boolean`

* `T` has a widening conversion to `Boolean?`

* `T` defines two pseudo operators, `IsTrue` and `IsFalse`.

* `T` has a narrowing conversion to `Boolean?` that does not involve a conversion from `Boolean` to `Boolean?`.

* `T` has a narrowing conversion to `Boolean`.

__Note.__ It is interesting to note that if `Option Strict` is off, an expression that has a narrowing conversion to `Boolean` will be accepted without a compile-time error but the language will still prefer an `IsTrue` operator if it exists. This is because `Option Strict` only changes what is and isn't accepted by the language, and never changes the actual meaning of an expression. Thus, `IsTrue` has to always be preferred over a narrowing conversion, regardless of `Option Strict`.

For example, the following class does not define a widening conversion to `Boolean`. As a result, its use in the `If` statement causes a call to the `IsTrue` operator.

```vb
Class MyBool
    Public Shared Widening Operator CType(b As Boolean) As MyBool
        ...
    End Operator

    Public Shared Narrowing Operator CType(b As MyBool) As Boolean
        ...
    End Operator

    Public Shared Operator IsTrue(b As MyBool) As Boolean
        ...
    End Operator

    Public Shared Operator IsFalse(b As MyBool) As Boolean
        ...
    End Operator
End Class

Module Test
    Sub Main()
        Dim b As New MyBool

        If b Then Console.WriteLine("True")
    End Sub
End Module
```

If a Boolean expression is typed as or converted to `Boolean` or `Boolean?`, then it is true if the value is `True` and false otherwise.

Otherwise, a Boolean expression calls the `IsTrue` operator and returns `True` if the operator returned `True`; otherwise it is false (but never calls the `IsFalse` operator).

In the following example, `Integer` has a narrowing conversion to `Boolean`, so a null `Integer?` has a narrowing conversion to both `Boolean?` (yielding a null `Boolean`) and to `Boolean` (which throws an exception). The narrowing conversion to `Boolean?` is preferred, and so the value of "`i`" as a Boolean expression is therefore `False`.

```vb
Dim i As Integer? = Nothing
If i Then Console.WriteLine()
```


## Lambda Expressions

A *lambda expression* defines an anonymous method called a *lambda method*. Lambda methods make it easy to pass "in-line" methods to other methods that take delegate types.

```antlr
LambdaExpression
    : SingleLineLambda
    | MultiLineLambda
    ;

SingleLineLambda
    : LambdaModifier* 'Function' ( OpenParenthesis ParameterList? CloseParenthesis )? Expression
    | 'Sub' ( OpenParenthesis ParameterList? CloseParenthesis )? Statement
    ;

MultiLineLambda
    : MultiLineFunctionLambda
    | MultiLineSubLambda
    ;

MultiLineFunctionLambda
    : LambdaModifier* 'Function' ( OpenParenthesis ParameterList? CloseParenthesis )? ( 'As' TypeName )? LineTerminator
      Block
      'End' 'Function'
    ;

MultiLineSubLambda
    : LambdaModifier* 'Sub' ( OpenParenthesis ParameterList? CloseParenthesis )? LineTerminator
      Block
      'End' 'Sub'
    ;

LambdaModifier
    : 'Async' | 'Iterator'
    ;
```

The example:

```vb
Module Test
    Delegate Function IntFunc(x As Integer) As Integer

    Sub Apply(a() As Integer, func As IntFunc)
        For index As Integer = 0 To a.Length - 1
            a(index) = func(a(index))
        Next index
    End Sub

    Sub Main()
        Dim a() As Integer = { 1, 2, 3, 4 }

        Apply(a, Function(x As Integer) x * 2)

        For Each value In a
            Console.Write(value & " ")
        Next value
    End Sub
End Module
```

will print out:

```
2 4 6 8
```

A lambda expression begins with the optional modifiers `Async` or `Iterator`, followed by the keyword `Function` or `Sub` and a parameter list. Parameters in a lambda expression cannot be declared `Optional` or `ParamArray` and cannot have attributes. Unlike regular methods, omitting a parameter type for a lambda method does not automatically infer `Object`. Instead, when a lambda method is reclassified, the omitted parameter types and `ByRef` modifiers are inferred from the target type. In the previous example, the lambda expression could have been written as `Function(x) x * 2`, and it would have inferred the type of `x` to be `Integer` when the lambda method was used to create an instance of the `IntFunc` delegate type. Unlike local variable inference, if a lambda method parameter omits a type but has an array or nullable name modifier, a compile-time error occurs.

A __regular lambda expression__ is one with neither `Async` nor `Iterator` modifiers.

An __iterator lambda expression__ is one with the `Iterator` modifier and no `Async` modifier. It must be a function. When it is reclassified to a value, it can only be reclassified to a value of delegate type whose return type is `IEnumerator`, or `IEnumerable`, or `IEnumerator(Of T)` or `IEnumerable(Of T)` for some `T`, and which has no ByRef parameters.

An __async lambda expression__ is one with the `Async` modifier and no `Iterator` modifier. An async sub lambda may only be reclassified to a value of sub delegate type with no ByRef parameters. An async function lambda may only be reclassified to a value of function delegate type whose return type is `Task` or `Task(Of T)` for some `T`, and which has no ByRef parameters.

Lambda expressions can either be single-line or multi-line. Single-line `Function` lambda expressions contain a single expression that represents the value returned from the lambda method. Single-line `Sub` lambda expressions contain a single statement without its closing `StatementTerminator`. For example:

```vb
Module Test
    Sub Do(a() As Integer, action As Action(Of Integer))
        For index As Integer = 0 To a.Length - 1
            action(a(index))
        Next index
    End Sub

    Sub Main()
        Dim a() As Integer = { 1, 2, 3, 4 }

        Do(a, Sub(x As Integer) Console.WriteLine(x))
    End Sub
End Module
```

Single-line lambda constructs bind less tightly than all other expressions and statements. Thus, for example, "`Function() x + 5`" is equivalent to "`Function() (x+5)"` rather than "`(Function() x) + 5`". To avoid ambiguity, a single-line `Sub` lambda expression may not contain a Dim statement or a label declaration statement. Also, unless it is enclosed in parentheses, a single-line `Sub` lambda expression may not be immediately followed by a colon ":", a member access operator ".", a dictionary member access operator "!" or an open parenthesis "(". It may not contain any block statement (`With`, `SyncLock, If...EndIf`, `While`, `For`, `Do`, `Using`) nor `OnError` nor `Resume`.

__Note.__ In the lambda expression `Function(i) x=i`, the body is interpreted as an *expression* (which tests whether `x` and `i` are equal). But in the lambda expression `Sub(i) x=i`, the body is interpreted as a statement (which assigns `i` to `x`).

A multi-line lambda expression contains a statement block and must end with an appropriate `End` statement (i.e. `End Function` or `End Sub`). As with regular methods, a multi-line lambda method's `Function` or `Sub` statement and `End` statements must be on their own lines. For example:

```vb
' Error: Function statement must be on its own line!
Dim x = Sub(x As Integer) : Console.WriteLine(x) : End Sub

' OK
Dim y = Sub(x As Integer)
               Console.WriteLine(x)
          End Sub
```

Multi-line `Function` lambda expressions can declare a return type but cannot put attributes on it. If a multi-line `Function` lambda expression does not declare a return type but the return type can be inferred from the context in which the lambda expression is used , then that return type is used. Otherwise the return type of the function is calculated as follows:

* In a regular lambda expression, the return type is the dominant type of the expressions in all the `Return` statements in the statement block.

* In an async lambda expression, the return type is `Task(Of T)` where `T` is the dominant type of the expressions in all the `Return` statements in the statement block.

* In an iterator lambda expression, the return type is `IEnumerable(Of T)` where `T` is the dominant type of the expressions in all the `Yield` statements in the statement block.

For example:

```vb
Function f(min As Integer, max As Integer) As IEnumerable(Of Integer)
    If min > max Then Throw New ArgumentException()
    Dim x = Iterator Function()
                  For i = min To max
                    Yield i
                Next
               End Function

    ' infers x to be a delegate with return type IEnumerable(Of Integer)
    Return x()
End Function
```

In all cases, if there are no `Return` (respectively `Yield`) statements, or if there is no dominant type among them, and strict semantics are being used, a compile-time error occurs; otherwise the dominant type is implicitly `Object`.

Note that the return type is calculated from all `Return` statements, even if they are not reachable. For example:

```vb
' Return type is Double
Dim x = Function()
              Return 10
               Return 10.50
          End Function
```

There is no implicit return variable, as there is no name for the variable.

The statement blocks inside multi-line lambda expressions have the following restrictions:

* `On Error` and `Resume` statements are not allowed, although `Try` statements are allowed.

* Static locals cannot be declared in multi-line lambda expressions.

* It is not possible to branch into or out of the statement block of a multi-line lambda expression, although the normal branching rules apply within it. For example:

  ```vb
  Label1:
  Dim x = Sub()
                 ' Error: Cannot branch out
                 GoTo Label1

                 ' OK: Wholly within the lamba.
                 GoTo Label2:
            Label2:
            End Sub

  ' Error: Cannot branch in
  GoTo Label2
  ```

A lambda expression is roughly equivalent to an anonymous method declared on the containing type. The initial example is roughly equivalent to:

```vb
Module Test
    Delegate Function IntFunc(x As Integer) As Integer

    Sub Apply(a() As Integer, func As IntFunc)
        For index As Integer = 0 To a.Length - 1
            a(index) = func(a(index))
        Next index
    End Sub

    Function $Lambda1(x As Integer) As Integer
        Return x * 2
    End Function

    Sub Main()
        Dim a() As Integer = { 1, 2, 3, 4 }

        Apply(a, AddressOf $Lambda1)

        For Each value In a
            Console.Write(value & " ")
        Next value
    End Sub
End Module
```


### Closures

Lambda expressions have access to all of the variables in scope, including local variables or parameters defined in the containing method and lambda expressions. When a lambda expression refers to a local variable or parameter, the lambda expression captures the variable being referred to into a closure. A closure is an object that lives on the heap instead of on the stack, and when a variable is captured, all references to the variable are redirected to the closure. This enables lambda expressions to continue to refer to local variables and parameters even after the containing method is complete. For example:

```vb
Module Test
    Delegate Function D() As Integer

    Function M() As D
        Dim x As Integer = 10
        Return Function() x
    End Function

    Sub Main()
        Dim y As D = M()

        ' Prints 10
        Console.WriteLine(y())
    End Sub
End Module
```

is roughly equivalent to:

```vb
Module Test
    Delegate Function D() As Integer

    Class $Closure1
        Public x As Integer

        Function $Lambda1() As Integer
            Return x
        End Function
    End Class

    Function M() As D
        Dim c As New $Closure1()
        c.x = 10
        Return AddressOf c.$Lambda1
    End Function

    Sub Main()
        Dim y As D = M()

        ' Prints 10
        Console.WriteLine(y())
    End Sub
End Module
```

A closure captures a new copy of a local variable each time it enters the block in which the local variable is declared, but the new copy is initialized with the value of the previous copy, if any. For example:

```vb
Module Test
    Delegate Function D() As Integer

    Function M() As D()
        Dim a(9) As D

        For i As Integer = 0 To 9
            Dim x
            a(i) = Function() x
            x += 1
        Next i

        Return a
    End Function

    Sub Main()
        Dim y() As D = M()

        For i As Integer = 0 To 9
            Console.Write(y(i)() & " ")
        Next i
    End Sub
End Module
```

prints

```
1 2 3 4 5 6 7 8 9 10
```

instead of

```
9 9 9 9 9 9 9 9 9 9
```

Because closures have to be initialized when entering a block, it is not allowed to `GoTo` into a block with a closure from outside of that block, although it is allowed to `Resume` into a block with a closure. For example:

```vb
Module Test
    Sub Main()
        Dim a = 10

        If a = 10 Then
L1:
            Dim x = Function() a

            ' Valid, source is within block
            GoTo L2
L2:
        End If

        ' ERROR: target is inside block with closure
        GoTo L1
    End Sub
End Module
```

Because they cannot be captured into a closure, the following cannot appear inside of a lambda expression:

* Reference parameters.

* Instance expressions (`Me`, `MyClass`, `MyBase`), if the type of `Me` is not a class.

The members of an anonymous type-creation expression, if the lambda expression is part of the expression. For example:

```vb
' Error: Lambda cannot refer to anonymous type field
Dim x = New With { .a = 12, .b = Function() .a }
```

`ReadOnly` instance variables in instance constructors or `ReadOnly` shared variables in shared constructors where the variables are used in a non-value context. For example:

```vb
Class C1
    ReadOnly F1 As Integer

    Sub New()
        ' Valid, doesn't modify F1
        Dim x = Function() F1

        ' Error, tries to modify F1
        Dim f = Function() ModifyValue(F1)
    End Sub

    Sub ModifyValue(ByRef x As Integer)
    End Sub
End Class
```

## Query Expressions

A *query expression* is an expression that applies a series of *query operators* to the elements of a *queryable* collection. For example, the following expression takes a collection of `Customer` objects and returns the names of all the customers in the state of Washington:

```vb
Dim names = _
    From cust In Customers _
    Where cust.State = "WA" _
    Select cust.Name
```

A query expression must start with a `From` or an `Aggregate` operator and can end with any query operator. The result of a query expression is classified as a value; the result type of the expression depends on the result type of the last query operator in the expression.

```antlr
QueryExpression
    : FromOrAggregateQueryOperator QueryOperator*
    ;

FromOrAggregateQueryOperator
    : FromQueryOperator
    | AggregateQueryOperator
    ;

QueryOperator
    : FromQueryOperator
    | AggregateQueryOperator
    | SelectQueryOperator
    | DistinctQueryOperator
    | WhereQueryOperator
    | OrderByQueryOperator
    | PartitionQueryOperator
    | LetQueryOperator
    | GroupByQueryOperator
    | JoinOrGroupJoinQueryOperator
    ;

JoinOrGroupJoinQueryOperator
    : JoinQueryOperator
    | GroupJoinQueryOperator
    ;
```

### Range Variables

Some query operators introduce a special kind of variable called a *range variable*. Range variables are not real variables; instead, they represent the individual values during the evaluation of the query over the input collections.

```antlr
CollectionRangeVariableDeclarationList
    : CollectionRangeVariableDeclaration ( Comma CollectionRangeVariableDeclaration )*
    ;

CollectionRangeVariableDeclaration
    : Identifier ( 'As' TypeName )? 'In' LineTerminator? Expression
    ;

ExpressionRangeVariableDeclarationList
    : ExpressionRangeVariableDeclaration ( Comma ExpressionRangeVariableDeclaration )*
    ;

ExpressionRangeVariableDeclaration
    : Identifier ( 'As' TypeName )? Equals Expression
    ;
```

Range variables are scoped from the introducing query operator to the end of a query expression, or to a query operator such as `Select` that hides them. For example, in the following query

```vb
Dim waCusts = _
    From cust As Customer In Customers _
    Where cust.State = "WA"
```

the `From` query operator introduces a range variable `cust` typed as `Customer` that represents each customer in the `Customers` collection. The following `Where` query operator then refers to the range variable `cust` in the filter expression to determine whether to filter an individual customer out of the resulting collection.

There are two types of range variables: *collection range variables* and *expression range variables*. Collection range variables take their values from the elements of the collections being queried. The collection expression in a collection range variable declaration must be classified as a value whose type is queryable. If the type of a collection range variable is omitted, it is inferred to be the element type of the collection expression, or `Object` if the collection expression does not have an element type (i.e. only defines a `Cast` method). If the collection expression is not queryable (i.e. the element type of the collection cannot be inferred), a compile-time error results.

An expression range variable is a range variable whose value is calculated by an expression rather than a collection. In the following example, the `Select` query operator introduces an expression range variable named `cityState` calculated from two fields:

```vb
Dim cityStates = _
    From cust As Customer In Customers _
    Select cityState = cust.City & "," & cust.State _
    Where cityState.Length() < 10
```

An expression range variable is not required to reference another range variable, although such a variable may be of dubious value. The expression assigned to an expression range variable must be classified as a value and must be implicitly convertible to the type of the range variable, if given.

Only in a Let operator may an expression range variable have its type specified. In other operators, or if its type is not specified, then local variable type inference is used to determine the type of the range variable.

A range variable must follow the rules for declaring local variables in respect to shadowing. Thus, a range variable cannot hide the name of a local variable or parameter in the enclosing method or another range variable (unless the query operator specifically hides all current range variables in scope).


### Queryable Types

Query expressions are implemented by translating the expression into calls to well-known methods on a collection type. These well-defined methods define the element type of the queryable collection as well as the result types of query operators executed on the collection. Each query operator specifies the method or methods that the query operator is generally translated into, although the specific translation is implementation dependent. The methods are given in the specification using a general format that looks like:

```vb
Function Select(selector As Func(Of T, R)) As CR
```

The following applies to the methods:

* The method must be an instance or extension member of the collection type and must be accessible.

* The method may be generic, provided that is possible to infer all type arguments.

* The method may be overloaded, in which case overload resolution is used to determine the exactly method to use.

* Another delegate type may be used in place of the delegate `Func` type, provided that it has the same signature, including return type, as the matching `Func` type.

* The type `System.Linq.Expressions.Expression(Of D)` may be used in place of the delegate `Func` type, provided that `D` is a delegate type that has the same signature, including return type, as the matching `Func` type.

* The type `T` represents the element type of the input collection. All of the methods defined by a collection type must have the same input element type for the collection type to be queryable.

* The type `S` represents the element type of the second input collection in the case of query operators that perform joins.

* The type `K` represents a key type in the case of query operators that have a set of range variables that act as keys.

* The type `N` represents a type that is used as a numeric type (although it could still be a user-defined type and not an intrinsic numeric type).

* The type `B` represents a type that can be used in a Boolean expression.

* The type `R` represents the element type of the result collection, if the query operator produces a result collection. `R` depends on the number of range variables in scope at the conclusion of the query operator. If a single range variable is in scope, then `R` is the type of that range variable. In the example

  ```vb
  Dim custNames = From c In Customers
                  Select c.Name
  ```

  the result of the query will be a collection type with an element type of `String`. If multiple range variables are in scope, then `R` is an anonymous type that contains all of the range variables in scope as `Key` fields. In the example:

  ```vb
  Dim custNames = From c In Customers, o In c.Orders 
                  Select Name = c.Name, ProductName = o.ProductName
  ```

  the result of the query will be a collection type with an element type of an anonymous type with a read-only property named `Name` of type `String` and a read-only property named `ProductName` of type `String`.

  Within a query expression, anonymous types generated to contain range variables are *transparent*, which means that range variables are always available without qualification. For example, in the previous example the range variables `c` and `o` could be accessed without qualification in the `Select` query operator, even though the input collection's element type was an anonymous type.

* The type `CX` represents a collection type, not necessarily the input collection type, whose element type is some type `X`.

A queryable collection type must satisfy one of the following conditions, in order of preference:

* It must define a conforming `Select` method.

* It must have have one of the following methods

  ```vb
  Function AsEnumerable() As CT
  Function AsQueryable() As CT
  ```

  which can be called to obtain a queryable collection. If both methods are provided, `AsQueryable` is preferred over `AsEnumerable`.

* It must have a method

  ```vb
  Function Cast(Of T)() As CT
  ```

  which can be called with the type of the range variable to produce a queryable collection.

Because determining the element type of a collection occurs independently of an actual method invocation, the applicability of specific methods cannot be determined. Thus, when determining the element type of a collection if there are instance methods that match well-known methods, then any extension methods that match well-known methods are ignored.

Query operator translation occurs in the order in which the query operators occur in the expression. It is not necessary for a collection object to implement all of the methods needed by all the query operators, although every collection object must at least support the `Select` query operator. If a needed method is not present, a compile-time error occurs. When binding well-known method names, non-methods are ignored for the purpose of multiple inheritance in interfaces and extension method binding, although shadowing semantics still apply. For example:

```vb
Class Q1
    Public Function [Select](selector As Func(Of Integer, Integer)) As Q1
    End Function
End Class

Class Q2
    Inherits Q1

    Public [Select] As Integer
End Class

Module Test
    Sub Main()
        Dim qs As New Q2()

        ' Error: Q2.Select still hides Q1.Select
        Dim zs = From q In qs Select q
    End Sub
End Module
```

### Default Query Indexer

Every queryable collection type whose element type is `T` and does not already have a default property is considered to have a default property of the following general form:

```vb
Public ReadOnly Default Property Item(index As Integer) As T
    Get
        Return Me.ElementAtOrDefault(index)
    End Get
End Property
```

The default property can only be referred to using the default property access syntax; the default property cannot be referred to by name. For example:

```vb
Dim customers As IEnumerable(Of Customer) = ...
Dim customerThree = customers(2)

' Error, no such property
Dim customerFour = customers.Item(4)
```

If the collection type does not have an `ElementAtOrDefault` member, a compile-time error will occur.

### From Query Operator

The `From` query operator introduces a collection range variable that represents the individual members of a collection to be queried.

```antlr
FromQueryOperator
    : LineTerminator? 'From' LineTerminator? CollectionRangeVariableDeclarationList
    ;
```

For example, the query expression:

```vb
From c As Customer In Customers ...
```

can be thought of as equivalent to

```vb
For Each c As Customer In Customers
        ...
Next c
```

When a `From` query operator declares multiple collection range variables or is not the first `From` query operator in the query expression, each new collection range variable is *cross joined* to the existing set of range variables. The result is that the query is evaluated over the cross-product of all the elements in the joined collections. For example, the expression:

```vb
From c In Customers _
From e In Employees _
...
```

can be thought of as equivalent to:

```vb
For Each c In Customers
    For Each e In Employees
            ...
    Next e
Next c
```

and is exactly equivalent to:

```vb
From c In Customers, e In Employees ...
```

The range variables introduced in previous query operators can be used in a later `From` query operator. For example, in the following query expression the second `From` query operator refers to the value of the first range variable:

```vb
From c As Customer In Customers _
From o As Order In c.Orders _
Select c.Name, o
```

Multiple range variables in a `From` query operator or multiple `From` query operators are only supported if the collection type contains one or both of the following methods:

```vb
Function SelectMany(selector As Func(Of T, CR)) As CR
Function SelectMany(selector As Func(Of T, CS), _
                          resultsSelector As Func(Of T, S, R)) As CR
```

The code

```vb
Dim xs() As Integer = ...
Dim ys() As Integer = ...
Dim zs = From x In xs, y In ys ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim ys() As Integer = ...
Dim zs = _
    xs.SelectMany( _
        Function(x As Integer) ys, _
        Function(x As Integer, y As Integer) New With {x, y})...
```

__Note.__ `From` is not a reserved word.


### Join Query Operator

The `Join` query operator joins existing range variables with a new collection range variable, producing a single collection whose elements have been joined together based on an equality expression.

```antlr
JoinQueryOperator
    : LineTerminator? 'Join' LineTerminator? CollectionRangeVariableDeclaration
      JoinOrGroupJoinQueryOperator? LineTerminator? 'On' LineTerminator? JoinConditionList
    ;

JoinConditionList
    : JoinCondition ( 'And' LineTerminator? JoinCondition )*
    ;

JoinCondition
    : Expression 'Equals' LineTerminator? Expression
    ;
```

For example:

```vb
Dim customersAndOrders = _
    From cust In Customers _
    Join ord In Orders On cust.ID Equals ord.CustomerID
```

The equality expression is more restricted than a regular equality expression:

* Both expressions must be classified as a value.

* Both expressions must reference at least one range variable.

* The range variable declared in the join query operator must be referenced by one of the expressions, and that expression must not reference any other range variables.

If the types of the two expressions are not the exact same type, then

* If the equality operator is defined for the two types, both expressions are implicitly convertible to it, and it is not `Object`, then convert both expressions to that type.

* Otherwise, if there is a dominant type that both expressions can be implicitly converted to, then convert both expressions to that type.

* Otherwise, a compile-time error occurs.

The expressions are compared using hash values (i.e. by calling `GetHashCode()`) rather than by using equality operators for efficiency. A `Join` query operator may do multiple joins or equality conditions in the same operator. A `Join` query operator is only supported if the collection type contains a method:

```vb
Function Join(inner As CS, _
                  outerSelector As Func(Of T, K), _
                  innerSelector As Func(Of S, K), _
                  resultSelector As Func(Of T, S, R)) As CR
```

The code

```vb
Dim xs() As Integer = ...
Dim ys() As Integer = ...
Dim zs = From x In xs _
            Join y In ys On x Equals y _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim ys() As Integer = ...
Dim zs = _
    xs.Join( _
        ys, _
        Function(x As Integer) x, _
        Function(y As Integer) y, _
        Function(x As Integer, y As Integer) New With {x, y})...
```

__Note.__ `Join`, `On` and `Equals` are not reserved words.


### Let Query Operator

The `Let` query operator introduces an expression range variable. This allows calculating an intermediate value once that will be used multiple times in later query operators.

```antlr
LetQueryOperator
    : LineTerminator? 'Let' LineTerminator? ExpressionRangeVariableDeclarationList
    ;
```

For example:

```vb
Dim taxedPrices = _
    From o In Orders _
    Let tax = o.Price * 0.088 _
    Where tax > 3.50 _
    Select o.Price, tax, total = o.Price + tax
```

can be thought of as equivalent to:

```vb
For Each o In Orders
    Dim tax = o.Price * 0.088
    ...
Next o
```

A `Let` query operator is only supported if the collection type contains a method:

```vb
Function Select(selector As Func(Of T, R)) As CR
```

The code

```vb
Dim xs() As Integer = ...
Dim zs = From x In xs _
            Let y = x * 10 _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim zs = _
    xs.Select(Function(x As Integer) New With {x, .y = x * 10})...
```


### Select Query Operator

The `Select` query operator is like the `Let` query operator in that it introduces expression range variables; however, a `Select` query operator hides the currently available range variables instead of adding to them. Also, the type of an expression range variable introduced by a `Select` query operator is always inferred using local variable type inference rules; an explicit type cannot be specified, and if no type can be inferred, a compile-time error occurs.

```antlr
SelectQueryOperator
    : LineTerminator? 'Select' LineTerminator? ExpressionRangeVariableDeclarationList
    ;
```

For example, in the query:

```vb
Dim smiths = _
    From cust In Customers _
    Select name = cust.name _
    Where name.EndsWith("Smith")
```

the `Where` query operator only has access to the `name` range variable introduced by the `Select` operator; if the `Where` operator had tried to reference `cust`, a compile-time error would have occurred.

Instead of explicitly specifying the names of the range variables, a `Select` query operator can infer the names of the range variables, using the same rules as anonymous type object creation expressions. For example:

```vb
Dim custAndOrderNames = _
      From cust In Customers, ord In cust.Orders _
      Select cust.name, ord.ProductName _
        Where name.EndsWith("Smith")
```

If the name of the range variable is not supplied and a name cannot be inferred, a compile-time error occurs. If the `Select` query operator contains only a single expression, no error occurs if a name for that range variable cannot be inferred but the range variable is nameless.  For example:

```vb
Dim custAndOrderNames = _
      From cust In Customers, ord In cust.Orders _
      Select cust.Name & " bought " & ord.ProductName _
        Take 10
```

If there is an ambiguity in a `Select` query operator between assigning a name to a range variable and an equality expression, the name assignment is preferred. For example:

```vb
Dim badCustNames = _
      From c In Customers _
        Let name = "John Smith" _
      Select name = c.Name ' Creates a range variable named "name"


Dim goodCustNames = _
      From c In Customers _
        Let name = "John Smith" _
      Select match = (name = c.Name)
```

Each expression in the `Select` query operator must be classified as a value. A `Select` query operator is supported only if the collection type contains a method:

```vb
Function Select(selector As Func(Of T, R)) As CR
```

The code

```vb
Dim xs() As Integer = ...
Dim zs = From x In xs _
            Select x, y = x * 10 _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim zs = _
    xs.Select(Function(x As Integer) New With {x, .y = x * 10})...
```


### Distinct Query Operator

The `Distinct` query operator restricts the values in a collection only to those with distinct values, as determined by comparing the element type for equality.

```antlr
DistinctQueryOperator
    : LineTerminator? 'Distinct' LineTerminator?
    ;
```

For example, the query:

```vb
Dim distinctCustomerPrice = _
    From cust In Customers, ord In cust.Orders _
    Select cust.Name, ord.Price _
    Distinct
```

will only return one row for each distinct pairing of customer name and order price, even if the customer has multiple orders with the same price. A `Distinct` query operator is supported only if the collection type contains a method:

```vb
Function Distinct() As CT
```

The code

```vb
Dim xs() As Integer = ...
Dim zs = From x In xs _
            Distinct _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim zs = xs.Distinct()...
```

__Note.__ `Distinct` is not a reserved word.


### Where Query Operator

The `Where` query operator restricts the values in a collection to those that satisfy a given condition.

```antlr
WhereQueryOperator
    : LineTerminator? 'Where' LineTerminator? BooleanExpression
    ;
```

A `Where` query operator takes a Boolean expression that is evaluated for each set of range variable values; if the value of the expression is true, then the values appear in the output collection, otherwise the values are skipped. For example, the query expression:

```vb
From cust In Customers, ord In Orders _
Where cust.ID = ord.CustomerID _
...
```

can be thought of as equivalent to the nested loop

```vb
For Each cust In Customers
    For Each ord In Orders
            If cust.ID = ord.CustomerID Then
                ...
            End If
    Next ord
Next cust
```

A `Where` query operator is supported only if the collection type contains a method:

```vb
Function Where(predicate As Func(Of T, B)) As CT
```

The code

```vb
Dim xs() As Integer = ...
Dim zs = From x In xs _
            Where x < 10 _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim zs = _
    xs.Where(Function(x As Integer) x < 10)...
```

__Note.__ `Where` is not a reserved word.


### Partition Query Operators

```antlr
PartitionQueryOperator
    : LineTerminator? 'Take' LineTerminator? Expression
    | LineTerminator? 'Take' 'While' LineTerminator? BooleanExpression
    | LineTerminator? 'Skip' LineTerminator? Expression
    | LineTerminator? 'Skip' 'While' LineTerminator? BooleanExpression
    ;
```

The `Take` query operator results in the first `n` elements of a collection. When used with the `While` modifier, the `Take` operator results in the first `n` elements of a collection that satisfy a Boolean expression. The `Skip` operator skips the first `n` elements of a collection and then returns the remainder of the collection.  When used in conjunction with the `While` modifier, the `Skip` operator skips the first `n` elements of a collection that satisfy a Boolean expression and then returns the rest of the collection. The expressions in a `Take` or `Skip` query operator must be classified as a value.

A `Take` query operator is supported only if the collection type contains a method:

```vb
Function Take(count As N) As CT
```

A `Skip` query operator is supported only if the collection type contains a method:

```vb
Function Skip(count As N) As CT
```

A `Take While` query operator is supported only if the collection type contains a method:

```vb
Function TakeWhile(predicate As Func(Of T, B)) As CT
```

A `Skip While` query operator is supported only if the collection type contains a method:

```vb
Function SkipWhile(predicate As Func(Of T, B)) As CT
```

The code

```vb
Dim xs() As Integer = ...
Dim zs = From x In xs _
            Skip 10 _
            Take 5 _
            Skip While x < 10 _
            Take While x > 5 _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim zs = _
    xs.Skip(10). _
        Take(5). _
        SkipWhile(Function(x) x < 10). _
        TakeWhile(Function(x) x > 5)...
```

__Note.__ `Take` and `Skip` are not reserved words.


### Order By Query Operator

The `Order By` query operator orders the values that appear in the range variables. 

```antlr
OrderByQueryOperator
    : LineTerminator? 'Order' 'By' LineTerminator? OrderExpressionList
    ;

OrderExpressionList
    : OrderExpression ( Comma OrderExpression )*
    ;

OrderExpression
    : Expression Ordering?
    ;

Ordering
    : 'Ascending' | 'Descending'
    ;
```

An `Order By` query operator takes expressions that specify the key values that should be used to order the iteration variables. For example, the following query returns products sorted by price:

```vb
Dim productsByPrice = _
    From p In Products _
    Order By p.Price _
    Select p.Name
```

An ordering can be marked as `Ascending`, in which case smaller values come before larger values, or `Descending`, in which case larger values come before smaller values. The default for an ordering if none is specified is `Ascending`. For example, the following query returns products sorted by price with the most expensive product first:

```vb
Dim productsByPriceDesc = _
    From p In Products _
    Order By p.Price Descending _
    Select p.Name
```

The `Order By` query operator may specify multiple expressions for ordering, in which case the collection is ordered in a nested manner. For example, the following query orders customers by state, then by city within each state and then by ZIP code within each city:

```vb
Dim customersByLocation = _
    From c In Customers _
    Order By c.State, c.City, c.ZIP _
    Select c.Name, c.State, c.City, c.ZIP
```

The expressions in an `Order By` query operator must be classified as a value. An `Order By` query operator is supported only if the collection type contains one or both of the following methods:

```vb
Function OrderBy(keySelector As Func(Of T, K)) As CT
Function OrderByDescending(keySelector As Func(Of T, K)) As CT
```

The return type `CT` must be an *ordered collection*. An ordered collection is a collection type that contains one or both of the methods:

```vb
Function ThenBy(keySelector As Func(Of T, K)) As CT
Function ThenByDescending(keySelector As Func(Of T, K)) As CT
```

The code

```vb
Dim xs() As Integer = ...
Dim zs = From x In xs _
            Order By x Ascending, x Mod 2 Descending _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim zs = _
    xs.OrderBy(Function(x) x).ThenByDescending(Function(x) x Mod 2)...
```

__Note.__ Because query operators simply map syntax to methods that implement a particular query operation, order preservation is not dictated by the language and is determined by the implementation of the operator itself. This is very similar to user-defined operators in that the implementation to overload the addition operator for a user-defined numeric type may not perform anything resembling an addition. Of course, to preserve predictability, implementing something that does not match user expectations is not recommended.

__Note.__ `Order` and `By` are not reserved words.


### Group By Query Operator

The `Group By` query operator groups the range variables in scope based on one or more expressions, and then produces new range variables based on those groupings.

```antlr
GroupByQueryOperator
    : LineTerminator? 'Group' ( LineTerminator? ExpressionRangeVariableDeclarationList )?
      LineTerminator? 'By' LineTerminator? ExpressionRangeVariableDeclarationList
      LineTerminator? 'Into' LineTerminator? ExpressionRangeVariableDeclarationList
    ;
```

For example, the following query groups all customers by `State`, and then computes the count and average age of each group:

```vb
Dim averageAges = _
    From cust In Customers _
    Group By cust.State _
    Into Count(), Average(cust.Age)
```

The `Group By` query operator has three clauses: the optional `Group` clause, the `By` clause, and the `Into` clause. The `Group` clause has the same syntax and effect as a `Select` query operator, except that it only affects the range variables available in the `Into` clause and not the `By` clause. For example:

```vb
Dim averageAges = _
    From cust In Customers _
    Group cust.Age By cust.State _
    Into Count(), Average(Age)
```

The `By` clause declares expression range variables that are used as key values in the grouping operation. The `Into` clause allows the declaration of expression range variables that calculate aggregations over each of the groups formed by the `By` clause. Within the `Into` clause, the expression range variable can only be assigned an expression which is a method invocation of an *aggregate function*. An aggregate function is a function on the collection type of the group (which may not necessarily be the same collection type of the original collection) which looks like either of the following methods:

```vb
Function _name_() As _type_
Function _name_(selector As Func(Of T, R)) As R
```

If an aggregate function takes a delegate argument, then the invocation expression can have an argument expression that must be classified as a value.  The argument expression can use the range variables that are in scope; within the call to an aggregate function, those range variables represent the values in the group being formed, not all of the values in the collection. For example, in the original example in this section the `Average` function calculates the average of the customers' ages per state rather than for all of the customers together.

All collection types are considered to have the aggregate function `Group` defined on it, which takes no parameters and simply returns the group. Other standard aggregate functions that a collection type may provide are:

`Count` and `LongCount`, which return the count of the elements in the group or the count of the elements in the group that satisfy a Boolean expression. `Count` and `LongCount` are supported only if the collection type contains one of the methods:

```vb
Function Count() As N
Function Count(selector As Func(Of T, B)) As N
Function LongCount() As N
Function LongCount(selector As Func(Of T, B)) As N
```

`Sum`, which returns the sum of an expression across all the elements in the group. `Sum` is supported only if the collection type contains one of the methods:

```vb
Function Sum() As N
Function Sum(selector As Func(Of T, N)) As N
```

`Min` which returns the minimum value of an expression across all the elements in the group. `Min` is supported only if the collection type contains one of the methods:

```vb
Function Min() As N
Function Min(selector As Func(Of T, N)) As N
```

`Max`, which returns the maximum value of an expression across all the elements in the group. `Max` is supported only if the collection type contains one of the methods:

```vb
Function Max() As N
Function Max(selector As Func(Of T, N)) As N
```

`Average`, which returns the average of an expression across all the elements in the group. `Average` is supported only if the collection type contains one of the methods:

```vb
Function Average() As N
Function Average(selector As Func(Of T, N)) As N
```

`Any`, which determines whether a group contains members or if a Boolean expression is true for any element in the group. `Any` returns a value that can be used in a Boolean expression and is supported only if the collection type contains one of the methods:

```vb
Function Any() As B
Function Any(predicate As Func(Of T, B)) As B
```

`All`, which determines whether a Boolean expression is true for all elements in the group. `All` returns a value that can be used in a Boolean expression and is supported only if the collection type contains a method:

```vb
Function All(predicate As Func(Of T, B)) As B
```

After a `Group By` query operator, the range variables previously in scope are hidden, and the range variables introduced by the `By` and `Into` clauses are available. A `Group By` query operator is supported only if the collection type contains the method:

```vb
Function GroupBy(keySelector As Func(Of T, K), _
                      resultSelector As Func(Of K, CT, R)) As CR
```

Range variable declarations in the `Group` clause are supported only if the collection type contains the method:

```vb
Function GroupBy(keySelector As Func(Of T, K), _
                      elementSelector As Func(Of T, S), _
                      resultSelector As Func(Of K, CS, R)) As CR
```

The code

```vb
Dim xs() As Integer = ...
Dim zs = From x In xs _
            Group y = x * 10, z = x / 10 By evenOdd = x Mod 2 _
            Into Sum(y), Average(z) _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim zs = _
    xs.GroupBy( _
        Function(x As Integer) x Mod 2, _
        Function(x As Integer) New With {.y = x * 10, .z = x / 10}, _
        Function(evenOdd, group) New With { _
            evenOdd, _
            .Sum = group.Sum(Function(e) e.y), _
            .Average = group.Average(Function(e) e.z)})...
```

__Note.__ `Group`, `By`, and `Into` are not reserved words.


### Aggregate Query Operator

The `Aggregate` query operator performs a similar function as the `Group By` operator, except it allows aggregating over groups that have already been formed. Because the group has already been formed, the `Into` clause of an `Aggregate` query operator does not hide the range variables in scope (in this way, `Aggregate` is more like a `Let`, and `Group By` is more like a `Select`).

```antlr
AggregateQueryOperator
    : LineTerminator? 'Aggregate' LineTerminator? CollectionRangeVariableDeclaration QueryOperator*
      LineTerminator? 'Into' LineTerminator? ExpressionRangeVariableDeclarationList
    ;
```

For example, the following query aggregates the total of all the orders placed by customers in Washington:

```vb
Dim orderTotals = _
    From cust In Customers _
    Where cust.State = "WA" _
    Aggregate order In cust.Orders _
    Into Sum(order.Total)
```

The result of this query is a collection whose element type is an anonymous type with a property named `cust` typed as `Customer` and a property named `Sum` typed as `Integer`.

Unlike `Group By`, additional query operators can be placed between the `Aggregate` and `Into` clauses. Between an `Aggregate` clause and the end of the `Into` clause, all range variables in scope, including those declared by the `Aggregate` clause can be used. For example, the following query aggregates the sum total of all the orders placed by customers in Washington before 2006:

```vb
Dim orderTotals = _
    From cust In Customers _
    Where cust.State = "WA" _
    Aggregate order In cust.Orders _
    Where order.Date <= #01/01/2006# _
    Into Sum = Sum(order.Total)
```

The `Aggregate` operator can also be used to start a query expression. In this case, the result of the query expression will be the single value computed by the `Into` clause. For example, the following query calculates the sum of all the order totals before January 1st, 2006:

```vb
Dim ordersTotal = _
    Aggregate order In Orders _
    Where order.Date <= #01/01/2006# _
    Into Sum(order.Total)
```

The result of the query is a single `Integer` value. An `Aggregate` query operator is always available (although the aggregate function must be also be available for the expression to be valid). The code

```vb
Dim xs() As Integer = ...
Dim zs = _
    Aggregate x In xs _
    Where x < 5 _
    Into Sum()
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim zs = _
    xs.Where(Function(x) x < 5).Sum()
```

__Note.__ `Aggregate` and `Into` are not reserved words.


### Group Join Query Operator

The `Group Join` query operator combines the functions of the `Join` and `Group By` query operators into a single operator. `Group Join` joins two collections based on matching keys extracted from the elements, grouping together all of the elements on the right side of the join that match a particular element on the left side of the join. Thus, the operator produces a set of hierarchical results.

```antlr
GroupJoinQueryOperator
    : LineTerminator? 'Group' 'Join' LineTerminator? CollectionRangeVariableDeclaration
      JoinOrGroupJoinQueryOperator? LineTerminator? 'On' LineTerminator? JoinConditionList
      LineTerminator? 'Into' LineTerminator? ExpressionRangeVariableDeclarationList
    ;
```

For example, the following query produces elements that contain a single customer's name, a group of all of their orders, and the total amount of all of those orders:

```vb
Dim custsWithOrders = _
    From cust In Customers _
    Group Join order In Orders On cust.ID Equals order.CustomerID _ 
    Into Orders = Group, OrdersTotal = Sum(order.Total) _
    Select cust.Name, Orders, OrdersTotal
```

The result of the query is a collection whose element type is an anonymous type with three properties: `Name`, typed as `String`, `Orders` typed as a collection whose element type is `Order`, and `OrdersTotal`, typed as `Integer`. A `Group Join` query operator is supported only if the collection type contains the method:

```vb
Function GroupJoin(inner As CS, _
                         outerSelector As Func(Of T, K), _
                         innerSelector As Func(Of S, K), _
                         resultSelector As Func(Of T, CS, R)) As CR
```

The code

```vb
Dim xs() As Integer = ...
Dim ys() As Integer = ...
Dim zs = From x In xs _
            Group Join y in ys On x Equals y _
            Into g = Group _
            ...
```

is generally translated to

```vb
Dim xs() As Integer = ...
Dim ys() As Integer = ...
Dim zs = _
    xs.GroupJoin( _
        ys, _
        Function(x As Integer) x, _
        Function(y As Integer) y, _
        Function(x, group) New With {x, .g = group})...
```

__Note.__ `Group`, `Join`, and `Into` are not reserved words.


## Conditional Expressions

A conditional `If` expression tests an expression and returns a value.

```antlr
ConditionalExpression
    : 'If' OpenParenthesis BooleanExpression Comma Expression Comma Expression CloseParenthesis
    | 'If' OpenParenthesis Expression Comma Expression CloseParenthesis
    ;
```

Unlike the `IIF` runtime function, however, a conditional expression only evaluates its operands if necessary. Thus, for example, the expression `If(c Is Nothing, c.Name, "Unknown")` will not throw an exception if the value of `c` is `Nothing`. The conditional expression has two forms: one that takes two operands and one that takes three operands.

If three operands are provided, all three expressions must be classified as values, and the first operand must be a Boolean expression. If the result is of the expression is true, then the second expression will be the result of the operator, otherwise the third expression will be the result of the operator. The result type of the expression is the dominant type between the types of the second and third expression. If there is no dominant type, then a compile-time error occurs.

If two operands are provided, both of the operands must be classified as values, and the first operand must be either a reference type or a nullable value type. The expression `If(x, y)` is then evaluated as if was the expression `If(x IsNot Nothing, x, y)`, with two exceptions. First, the first expression is only ever evaluated once, and second, if the second operand's type is a non-nullable value type and the first operand's type is, the `?` is removed from the type of the first operand when determining the dominant type for the result type of the expression. For example:

```vb
Module Test
    Sub Main()
        Dim x?, y As Integer
        Dim a?, b As Long

        a = If(x, a)        ' Result type: Long?
        y = If(x, 0)        ' Result type: Integer
    End Sub
End Module
```

In both forms of the expression, if an operand is `Nothing`, its type is not used to determine the dominant type. In the case of the expression `If(<expression>, Nothing, Nothing)`, the dominant type is considered to be `Object`.


## XML Literal Expressions

An XML literal expression represents an XML (eXtensible Markup Language) 1.0 value.

```antlr
XMLLiteralExpression
    : XMLDocument
    | XMLElement
    | XMLProcessingInstruction
    | XMLComment
    | XMLCDATASection
    ;
```

The result of an XML literal expression is a value typed as one of the types from the `System.Xml.Linq` namespace. If the types in that namespace are not available, then an XML literal expression will cause a compile-time error. The values are generated through constructor calls translated from the XML literal expression. For example, the code:

```vb
Dim book As System.Xml.Linq.XElement = _
    <book title="My book"></book>
```

is roughly equivalent to the code:

```vb
Dim book As System.Xml.Linq.XElement = _
    New System.Xml.Linq.XElement( _
        "book", _
        New System.Xml.Linq.XAttribute("title", "My book"))
```

An XML literal expression can take the form of an XML document, an XML element, an XML processing instruction, an XML comment, or a CDATA section.

__Note.__ This specification contains only enough of a description of XML to describe the behavior of the Visual Basic language. More information on XML can be found at http://www.w3.org/TR/REC-xml/.


### Lexical rules

```antlr
XMLCharacter
    : '<Unicode tab character (0x0009)>'
    | '<Unicode linefeed character (0x000A)>'
    | '<Unicode carriage return character (0x000D)>'
    | '<Unicode characters 0x0020 - 0xD7FF>'
    | '<Unicode characters 0xE000 - 0xFFFD>'
    | '<Unicode characters 0x10000 - 0x10FFFF>'
    ;

XMLString
    : XMLCharacter+
    ;

XMLWhitespace
    : XMLWhitespaceCharacter+
    ;

XMLWhitespaceCharacter
    : '<Unicode carriage return character (0x000D)>'
    | '<Unicode linefeed character (0x000A)>'
    | '<Unicode space character (0x0020)>'
    | '<Unicode tab character (0x0009)>'
    ;

XMLNameCharacter
    : XMLLetter
    | XMLDigit
    | '.'
    | '-'
    | '_'
    | ':'
    | XMLCombiningCharacter
    | XMLExtender
    ;

XMLNameStartCharacter
    : XMLLetter
    | '_'
    | ':'
    ;

XMLName
    : XMLNameStartCharacter XMLNameCharacter*
    ;

XMLLetter
    : '<Unicode character as defined in the Letter production of the XML 1.0 specification>'
    ;

XMLDigit
    : '<Unicode character as defined in the Digit production of the XML 1.0 specification>'
    ;

XMLCombiningCharacter
    : '<Unicode character as defined in the CombiningChar production of the XML 1.0 specification>'
    ;

XMLExtender
    : '<Unicode character as defined in the Extender production of the XML 1.0 specification>'
    ;
```

XML literal expressions are interpreted using the lexical rules of XML instead of the lexical rules of regular Visual Basic code. The two sets of rules generally differ in the following ways:

* White space is significant in XML. As a result, the grammar for XML literal expressions explicitly states where white space is allowed. Whitespace is not preserved, except when it occurs in the context of character data within an element. For example:

  ```vb
  ' The following element preserves no whitespace
  Dim e1 = _
      <customer>
          <name>Bob</>
      </>

  ' The following element preserves all of the whitespace
  Dim e2 = _
      <customer>
          Bob
      </>
  ```

* XML end-of-line whitespace is normalized according to the XML specification.

* XML is case-sensitive. Keywords must match casing exactly, or else a compile-time error will occur.

* Line terminators are considered white space in XML. As a result, no line-continuation characters are needed in XML literal expressions.

* XML does not accept full-width characters. If full-width characters are used, a compile-time error will occur.


### Embedded expressions

XML literal expressions can contain *embedded expressions*. An embedded expression is a Visual Basic expression that is evaluated and used to fill in one or more values at the location of embedded expression.

```antlr
XMLEmbeddedExpression
    : '<' '%' '=' LineTerminator? Expression LineTerminator? '%' '>'
    ;
```

For example, the following code places the string `John Smith` as the value of the XML element:

```vb
Dim name as String = "John Smith"
Dim element As System.Xml.Linq.XElement = <customer><%= name %></customer>
```

Expressions can be embedded in a number of contexts. For example, the following code produces an element named `customer`:

```vb
Dim name As String = "customer"
Dim element As System.Xml.Linq.XElement = <<%= name %>>John Smith</>
```

Each context where an embedded expression can be used specifies the types that will be accepted. When within the context of the expression part of an embedded expression, the normal lexical rules for Visual Basic code still apply so, for example, line continuations must be used:

```vb
' Visual Basic expression uses line continuation, XML does not
Dim element As System.Xml.Linq.XElement = _
    <<%= name & _
          name %>>John 
                     Smith</>
```


### XML Documents

```antlr
XMLDocument
    : XMLDocumentPrologue XMLMisc* XMLDocumentBody XMLMisc*
    ;

XMLDocumentPrologue
    : '<' '?' 'xml' XMLVersion XMLEncoding? XMLStandalone? XMLWhitespace? '?' '>'
    ;

XMLVersion
    : XMLWhitespace 'version' XMLWhitespace? '=' XMLWhitespace? XMLVersionNumberValue
    ;

XMLVersionNumberValue
    : SingleQuoteCharacter '1' '.' '0' SingleQuoteCharacter
    | DoubleQuoteCharacter '1' '.' '0' DoubleQuoteCharacter
    ;

XMLEncoding
    : XMLWhitespace 'encoding' XMLWhitespace? '=' XMLWhitespace? XMLEncodingNameValue
    ;

XMLEncodingNameValue
    : SingleQuoteCharacter XMLEncodingName SingleQuoteCharacter
    | DoubleQuoteCharacter XMLEncodingName DoubleQuoteCharacter
    ;

XMLEncodingName
    : XMLLatinAlphaCharacter XMLEncodingNameCharacter*
    ;

XMLEncodingNameCharacter
    : XMLUnderscoreCharacter
    | XMLLatinAlphaCharacter
    | XMLNumericCharacter
    | XMLPeriodCharacter
    | XMLDashCharacter
    ;

XMLLatinAlphaCharacter
    : '<Unicode Latin alphabetic character (0x0041-0x005a, 0x0061-0x007a)>'
    ;

XMLNumericCharacter
    : '<Unicode digit character (0x0030-0x0039)>'
    ;

XMLHexNumericCharacter
    : XMLNumericCharacter
    | '<Unicode Latin hex alphabetic character (0x0041-0x0046, 0x0061-0x0066)>'
    ;

XMLPeriodCharacter
    : '<Unicode period character (0x002e)>'
    ;

XMLUnderscoreCharacter
    : '<Unicode underscore character (0x005f)>'
    ;

XMLDashCharacter
    : '<Unicode dash character (0x002d)>'
    ;

XMLStandalone
    : XMLWhitespace 'standalone' XMLWhitespace? '=' XMLWhitespace? XMLYesNoValue
    ;

XMLYesNoValue
    : SingleQuoteCharacter XMLYesNo SingleQuoteCharacter
    | DoubleQuoteCharacter XMLYesNo DoubleQuoteCharacter
    ;

XMLYesNo
    : 'yes'
    | 'no'
    ;

XMLMisc
    : XMLComment
    | XMLProcessingInstruction
    | XMLWhitespace
    ;

XMLDocumentBody
    : XMLElement
    | XMLEmbeddedExpression
    ;
```

An XML document results in a value typed as `System.Xml.Linq.XDocument`. Unlike the XML 1.0 specification, XML documents in XML literal expressions are required to specify the XML document prologue; XML literal expressions without the XML document prologue are interpreted as their individual entity. For example:

```vb
Dim doc As System.Xml.Linq.XDocument = _
    <?xml version="1.0"?>
    <?instruction?>
    <customer>Bob</>

Dim pi As System.Xml.Linq.XProcessingInstruction = _
    <?instruction?>
```

An XML document can contain an embedded expression whose type can be any type; at runtime, however, the object must satisfy the requirements of the `XDocument` constructor or a run-time error will occur.

Unlike regular XML, XML document expressions do not support DTDs (Document Type Declarations). Also, the encoding attribute, if supplied, will be ignored since the encoding of the Xml literal expression is always the same as the encoding of the source file itself.

__Note.__ Although the encoding attribute is ignored, it is still valid attribute in order to maintain the ability to include any valid Xml 1.0 documents in source code.


### XML Elements

```antlr
XMLElement
    : XMLEmptyElement
    | XMLElementStart XMLContent XMLElementEnd
    ;

XMLEmptyElement
    : '<' XMLQualifiedNameOrExpression XMLAttribute* XMLWhitespace? '/' '>'
    ;

XMLElementStart
    : '<' XMLQualifiedNameOrExpression XMLAttribute* XMLWhitespace? '>'
    ;

XMLElementEnd
    : '<' '/' '>'
    | '<' '/' XMLQualifiedName XMLWhitespace? '>'
    ;

XMLContent
    : XMLCharacterData? ( XMLNestedContent XMLCharacterData? )+
    ;

XMLCharacterData
    : '<Any XMLCharacterDataString that does not contain the string "]]>">'
    ;

XMLCharacterDataString
    : '<Any Unicode character except < or &>'+
    ;

XMLNestedContent
    : XMLElement
    | XMLReference
    | XMLCDATASection
    | XMLProcessingInstruction
    | XMLComment
    | XMLEmbeddedExpression
    ;

XMLAttribute
    : XMLWhitespace XMLAttributeName XMLWhitespace? '=' XMLWhitespace? XMLAttributeValue
    | XMLWhitespace XMLEmbeddedExpression
    ;

XMLAttributeName
    : XMLQualifiedNameOrExpression
    | XMLNamespaceAttributeName
    ;

XMLAttributeValue
    : DoubleQuoteCharacter XMLAttributeDoubleQuoteValueCharacter* DoubleQuoteCharacter
    | SingleQuoteCharacter XMLAttributeSingleQuoteValueCharacter* SingleQuoteCharacter
    | XMLEmbeddedExpression
    ;

XMLAttributeDoubleQuoteValueCharacter
    : '<Any XMLCharacter except <, &, or DoubleQuoteCharacter>'
    | XMLReference
    ;

XMLAttributeSingleQuoteValueCharacter
    : '<Any XMLCharacter except <, &, or SingleQuoteCharacter>'
    | XMLReference
    ;

XMLReference
    : XMLEntityReference
    | XMLCharacterReference
    ;

XMLEntityReference
    : '&' XMLEntityName ';'
    ;

XMLEntityName
    : 'lt' | 'gt' | 'amp' | 'apos' | 'quot'
    ;

XMLCharacterReference
    : '&' '#' XMLNumericCharacter+ ';'
    | '&' '#' 'x' XMLHexNumericCharacter+ ';'
    ;
```

An XML element results in a value typed as `System.Xml.Linq.XElement`. Unlike regular XML, XML elements can omit the name in the closing tag and the current most-nested element will be closed. For example:

```vb
Dim name = <name>Bob</>
```

Attribute declarations in an XML element result in values typed as `System.Xml.Linq.XAttribute`. Attribute values are normalized according to the XML specification. When the value of an attribute is `Nothing` the attribute will not be created, so the attribute value expression will not have to be checked for `Nothing`. For example:

```vb
Dim expr = Nothing

' Throws null argument exception
Dim direct = New System.Xml.Linq.XElement( _
    "Name", _
    New System.Xml.Linq.XAttribute("Length", expr))

' Doesn't throw exception, the result is <Name/>
Dim literal = <Name Length=<%= expr %>/>
```

XML elements and attributes can contain nested expressions in the following places:

The name of the element, in which case the embedded expression must be a value of a type implicitly convertible to `System.Xml.Linq.XName`. For example:

```vb
Dim name = <<%= "name" %>>Bob</>
```

The name of an attribute of the element, in which case the embedded expression must be a value of a type implicitly convertible to `System.Xml.Linq.XName`. For example:

```vb
Dim name = <name <%= "length" %>="3">Bob</>
```

The value of an attribute of the element, in which case the embedded expression can be a value of any type. For example:

```vb
Dim name = <name length=<%= 3 %>>Bob</>
```

An attribute of the element, in which case the embedded expression can be a value of any type. For example:

```vb
Dim name = <name <%= new XAttribute("length", 3) %>>Bob</>
```

The content of the element, in which case the embedded expression can be a value of any type. For example:

```vb
Dim name = <name><%= "Bob" %></>
```

If the type of the embedded expression is `Object()`, the array will be passed as a paramarray to the `XElement` constructor.


### XML Namespaces

XML elements can contain XML namespace declarations, as defined by the XML namespaces 1.0 specification.

```antlr
XMLNamespaceAttributeName
    : XMLPrefixedNamespaceAttributeName
    | XMLDefaultNamespaceAttributeName
    ;

XMLPrefixedNamespaceAttributeName
    : 'xmlns' ':' XMLNamespaceName
    ;

XMLDefaultNamespaceAttributeName
    : 'xmlns'
    ;

XMLNamespaceName
    : XMLNamespaceNameStartCharacter XMLNamespaceNameCharacter*
    ;

XMLNamespaceNameStartCharacter
    : '<Any XMLNameCharacter except :>'
    ;

XMLNamespaceNameCharacter
    : XMLLetter
    | '_'
    ;

XMLQualifiedNameOrExpression
    : XMLQualifiedName
    | XMLEmbeddedExpression
    ;

XMLQualifiedName
    : XMLPrefixedName
    | XMLUnprefixedName
    ;

XMLPrefixedName
    : XMLNamespaceName ':' XMLNamespaceName
    ;

XMLUnprefixedName
    : XMLNamespaceName
    ;
```

The restrictions on defining the namespaces `xml` and `xmlns` are enforced and will produce compile-time errors. XML namespace declarations cannot have an embedded expression for their value; the value supplied must be a non-empty string literal. For example:

```vb
' Declares a valid namespace
Dim customer = <db:customer xmlns:db="http://example.org/database">Bob</>

' Error: xmlns cannot be re-defined
Dim bad1 = <elem xmlns:xmlns="http://example.org/namespace"/>

' Error: cannot have an embedded expression
Dim bad2 = <elem xmlns:db=<%= "http://example.org/database" %>>Bob</>
```

__Note.__ This specification contains only enough of a description of XML namespace to describe the behavior of the Visual Basic language. More information on XML namespaces can be found at http://www.w3.org/TR/REC-xml-names/.

XML element and attribute names can be qualified using namespace names. Namespaces are bound as in regular XML, with the exception that any namespace imports declared at the file level are considered to be declared in a context enclosing the declaration, which is itself enclosed by any namespace imports declared by the compilation environment. If a namespace name cannot be found, a compile-time error occurs. For example:

```vb
Imports System.Xml.Linq
Imports <xmlns:db="http://example.org/database">

Module Test
    Sub Main()
        ' Binds to the imported namespace above.
        Dim c1 = <db:customer>Bob</>

        ' Binds to the namespace declaration in the element
        Dim c2 = _
            <db:customer xmlns:db="http://example.org/database-other">Mary</>

        ' Binds to the inner namespace declaration
        Dim c3 = _
            <database xmlns:db="http://example.org/database-one">
                <db:customer xmlns:db="http://example.org/database-two">Joe</>
            </>

        ' Error: namespace db2 cannot be found
        Dim c4 = _
            <db2:customer>Jim</>
    End Sub
End Module
```

XML namespaces declared in an element do not apply to XML literals inside embedded expressions. For example:

```vb
' Error: Namespace prefix 'db' is not declared
Dim customer = _
    <db:customer xmlns:db="http://example.org/database">
        <%= <db:customer>Bob</> %>
    </>
```

__Note.__ This is because the embedded expression can be anything, including a function call. If the function call contained an XML literal expression, it is not clear whether programmers would expect the XML namespace to be applied or ignored.


### XML Processing Instructions

An XML processing instruction results in a value typed as `System.Xml.Linq.XProcessingInstruction`. XML processing instructions cannot contain embedded expressions, as they are valid syntax within the processing instruction.

```antlr
XMLProcessingInstruction
    : '<' '?' XMLProcessingTarget ( XMLWhitespace XMLProcessingValue? )? '?' '>'
    ;

XMLProcessingTarget
    : '<Any XMLName except a casing permutation of the string "xml">'
    ;

XMLProcessingValue
    : '<Any XMLString that does not contain a question-mark followed by ">">'
    ;
```

### XML Comments

An XML comment results in a value typed as `System.Xml.Linq.XComment`. XML comments cannot contain embedded expressions, as they are valid syntax within the comment.

```antlr
XMLComment
    : '<' '!' '-' '-' XMLCommentCharacter* '-' '-' '>'
    ;

XMLCommentCharacter
    : '<Any XMLCharacter except dash (0x002D)>'
    | '-' '<Any XMLCharacter except dash (0x002D)>'
    ;
```

### CDATA sections

A CDATA section results in a value typed as `System.Xml.Linq.XCData`. CDATA sections cannot contain embedded expressions, as they are valid syntax within the CDATA section.

```antlr
XMLCDATASection
    : '<' '!' ( 'CDATA' '[' XMLCDATASectionString? ']' )? '>'
    ;

XMLCDATASectionString
    : '<Any XMLString that does not contain the string "]]>">'
    ;
```

## XML Member Access Expressions

An XML member access expression accesses the members of an XML value.

```antlr
XMLMemberAccessExpression
    : Expression '.' LineTerminator? '<' XMLQualifiedName '>'
    | Expression '.' LineTerminator? '@' LineTerminator? '<' XMLQualifiedName '>'
    | Expression '.' LineTerminator? '@' LineTerminator? IdentifierOrKeyword
    | Expression '.' '.' '.' LineTerminator? '<' XMLQualifiedName '>'
    ;
```

There are three types of XML member access expressions:

* *Element access*, in which an XML name follows a single dot. For example:

  ```vb
  Dim customer = _
      <customer>
          <name>Bob</>
      </>
  Dim customerName = customer.<name>.Value
  ```

  Element access maps to the function:

  ```vb
  Function Elements(name As System.Xml.Linq.XName) As _
      System.Collections.Generic.IEnumerable(Of _
          System.Xml.Linq.XNode)
  ```

  So the above example is equivalent to:

  ```vb
  Dim customerName = customer.Elements("name").Value
  ```

* *Attribute access*, in which a Visual Basic identifier follows a dot and an at sign, or an XML name follows a dot and an at sign. For example:

  ```vb
  Dim customer = <customer age="30"/>
  Dim customerAge = customer.@age
  ```

  Attribute access maps to the function:

  ```vb
  Function AttributeValue(name As System.Xml.Linq.XName) as String
  ```

  So the above example is equivalent to:

  ```vb
  Dim customerAge = customer.AttributeValue("age")
  ```

  __Note.__ The `AttributeValue` extension method (as well as the related extension property `Value`) is not currently defined in any assembly. If the extension members are needed, they are automatically defined in the assembly being produced.

* *Descendents access*, in which an XML names follows three dots. For example:

  ```vb
  Dim company = _
      <company>
          <customers>
              <customer>Bob</>
              <customer>Mary</>
              <customer>Joe</>
          </>
      </>
  Dim customers = company...<customer>
  ```

  Descendents access maps to the function:

  ```vb
  Function Descendents(name As System.Xml.Linq.XName) As _
      System.Collections.Generic.IEnumerable(Of _
          System.Xml.Linq.XElement)
  ```

  So the above example is equivalent to:

  ```vb
  Dim customers = company.Descendants("customer")
  ```

The base expression of an XML member access expression must be a value and must be of the type:

* If an element or descendents access,  `System.Xml.Linq.XContainer` or a derived type, or `System.Collections.Generic.IEnumerable(Of T)` or a derived type, where `T` is `System.Xml.Linq.XContainer` or a derived type.

* If an attribute access, `System.Xml.Linq.XElement` or a derived type, or `System.Collections.Generic.IEnumerable(Of T)` or a derived type, where `T` is `System.Xml.Linq.XElement` or a derived type.

Names in XML member access expressions cannot be empty. They can be namespace qualified, using any namespaces defined by imports. For example:

```vb
Imports <xmlns:db="http://example.org/database">

Module Test
    Sub Main()
        Dim customer = _
            <db:customer>
                <db:name>Bob</>
            </>
        Dim name = customer.<db:name>
    End Sub
End Module
```

Whitespace is not allowed after the dot(s) in an XML member access expression, or between the angle brackets and the name. For example:

```vb
Dim customer = _
    <customer age="30">
        <name>Bob</>
    </>
' All the following are error cases
Dim age = customer.@ age
Dim name = customer.< name >
Dim names = customer...< name >
```

If the types in the `System.Xml.Linq` namespace are not available, then an XML member access expression will cause a compile-time error.


## Await Operator

The await operator is related to async methods, which are described in Section [Async Methods](statements.md#async-methods).

```antlr
AwaitOperatorExpression
    : 'Await' Expression
    ;
```

`Await` is a reserved word if the immediately enclosing method or lambda expression in which it appears has an `Async` modifier, and if the `Await` appears after that `Async` modifier; it is unreserved elsewhere. It is also unreserved in preprocessor directives. The await operator is only allowed in the body of a method or lambda expressions where it is a reserved word. Within the immediately enclosing method or lambda, an await expression may not occur inside the body of a `Catch` or `Finally` block, nor inside the body of a `SyncLock` statement, nor inside a query expression.

The await operator takes a single expression which must be classified as a value and whose type must be an *awaitable* type, or `Object`. If its type is `Object` then all processing is deferred until run-time. A type `C` is said to be awaitable if all of the following are true:

* `C` contains an accessible instance or extension method named `GetAwaiter` which has no arguments and which returns some type `E`;

* `E` contains a readable instance or extension property named `IsCompleted` which takes no arguments and has type Boolean;

* `E` contains an accessible instance or extension method named `GetResult` which takes no arguments;

* `E` implements either `System.Runtime.CompilerServices.INotifyCompletion` or `ICriticalNotifyCompletion`.

If `GetResult` was a `Sub`, then the await expression is classified as void. Otherwise, the await expression is classified as a value and its type is the return type of the `GetResult` method.

Here is an example of a class that can be awaited:

```vb
Class MyTask(Of T)
    Function GetAwaiter() As MyTaskAwaiter(Of T)
        Return New MyTaskAwaiter With {.m_Task = Me}
    End Function

    ...
End Class

Structure MyTaskAwaiter(Of T)
    Implements INotifyCompletion

    Friend m_Task As MyTask(Of T)

    ReadOnly Property IsCompleted As Boolean
        Get
            Return m_Task.IsCompleted
        End Get
    End Property

    Sub OnCompleted(r As Action) Implements INotifyCompletion.OnCompleted
        ' r is the "resumptionDelegate"
        Dim sc = SynchronizationContext.Current
        If sc Is Nothing Then
            m_Task.ContinueWith(Sub() r())
        Else
            m_Task.ContinueWith(Sub() sc.Post(Sub() r(), Nothing))
        End If
    End Sub

    Function GetResult() As T
        If m_Task.IsCanceled Then Throw New TaskCanceledException(m_Task)
        If m_Task.IsFaulted Then Throw m_Task.Exception.InnerException
        Return m_Task.Result
    End Function
End Structure
```

__Note.__ Library authors are recommended to follow the pattern that they invoke the continuation delegate on the same `SynchronizationContext` as their `OnCompleted` was itself invoked on. Also, the resumption delegate should not be executed synchronously within the `OnCompleted` method since that can lead to stack overflow: instead, the delegate should be queued for subsequent execution.

When control flow reaches an `Await` operator, behavior is as follows.

1.  The `GetAwaiter` method of the await operand is invoked. The result of this invocation is termed the *awaiter*.

2.  The awaiter's `IsCompleted` property is retrieved. If the result is true then:

    21. The `GetResult` method of the awaiter is invoked. If `GetResult` was a function, then the value of the await expression is the return value of this function.

3.  If the IsCompleted property isn't true then:

    31. Either `ICriticalNotifyCompletion.UnsafeOnCompleted` is invoked on the awaiter (if the awaiter's type `E` implements `ICriticalNotifyCompletion`) or `INotifyCompletion.OnCompleted` (otherwise). In both cases it passes a *resumption delegate* associated with the current instance of the async method.

    32. The control point of the current async method instance is suspended, and control flow resumes in the *current caller* (defined in Section [Async Methods](statements.md#async-methods)).

    33. If later the resumption delegate is invoked,

        331. the resumption delegate first restores `System.Threading.Thread.CurrentThread.ExecutionContext` to what it was at the time `OnCompleted` was called,
        332. then it resumes control flow at the control point of the async method instance (see Section [Async Methods](statements.md#async-methods)),
        333. where it calls the `GetResult` method of the awaiter, as in 2.1 above.

If the await operand has type Object, then this behavior is deferred until runtime:

- Step 1 is accomplished by calling GetAwaiter() with no arguments; it may therefore bind at runtime to instance methods which take optional parameters.
- Step 2 is accomplished by retrieving the IsCompleted() property with no arguments, and by attempting an intrinsic conversion to Boolean.
- Step 3.a is accomplished by attempting `TryCast(awaiter, ICriticalNotifyCompletion)`, and if this fails then `DirectCast(awaiter, INotifyCompletion)`.

The resumption delegate passed in 3.a may only be invoked once. If it is invoked more than once, the behavior is undefined.

