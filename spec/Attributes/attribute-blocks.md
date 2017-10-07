^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Visual Basic Specification]("spec/VisualBasic-Specification.md") / Attributes / **Attribute Blocks**  
**Sub-Topics:** [Attribute Classes](Attribute-Classes), **[Attribute Blocks](#Attribrute-Blocks)**, [Attribute Names](Attribute-Names), [Attribute Arguments](#Attribute-Arguments)

----

## Attribute Blocks

Attributes are specified in *attribute blocks*. Each attribute block is delimited by angle brackets ("<>"), and multiple attributes can be specified in a comma-separated list within an attribute block or in multiple attribute blocks. The order in which attributes are specified is not significant. For example, the attribute blocks `<A, B>`, `<B, A>`, `<A> <B>` and `<B> <A>` are all equivalent.

```antlr
Attributes
    : AttributeBlock+
    ;

AttributeBlock
    : LineTerminator? '<' AttributeList LineTerminator? '>' LineTerminator?
    ;

AttributeList
    : Attribute ( Comma Attribute )*
    ;

Attribute
    : ( AttributeModifier ':' )? SimpleTypeName
    ( OpenParenthesis AttributeArguments? CloseParenthesis )?
    ;

AttributeModifier
    : 'Assembly' | 'Module'
    ;
```

An attribute may not be specified on a kind of declaration it does not support, and single-use attributes may not be specified more than once in an attribute block. The example below causes errors both because it attempts to use `HelpString` on the interface `Interface1` and more than once on the declaration of `Class1`.

```vb
<AttributeUsage(AttributeTargets.Class)> _
Public Class HelpStringAttribute
    Inherits System.Attribute

    Private InternalValue As String

    Public Sub New(value As String)
        Me.InternalValue = value
    End Sub

    Public ReadOnly Property Value() As String
        Get
            Return InternalValue
        End Get
    End Property
End Class

' Error: HelpString only applies to classes.
<HelpString("Description of Interface1")> _
Interface Interface1
    Sub Sub1()
End Interface

' Error: HelpString is single-use.
<HelpString("Description of Class1"), _
    HelpString("Another description of Class1")> _
Public Class Class1
End Class
```

An attribute consists of an optional attribute modifier, an attribute name, an optional list of positional arguments, and variable/property initializers. If there are no parameters or initializers, the parentheses may be omitted. If an attribute has a modifier, it must be in an attribute block at the top of a source file.

If a source file contains an attribute block at the top of the file that specifies attributes for the assembly or module that will contain the source file, each attribute in the attribute block must be prefixed by both the `Assembly` or `Module` modifier and a colon.


### Attribute Names

The name of an attribute specifies an attribute class. By convention, attribute classes are named with the suffix `Attribute`. Uses of an attribute may either include or omit this suffix. Consequently the name of an attribute class that corresponds to an attribute identifier is either the identifier itself or the concatenation of the qualified identifier and `Attribute`. When the compiler resolves an attribute name, it appends `Attribute` to the name and tries the lookup. If that lookup fails, the compiler tries the lookup without the suffix. For example, uses of an attribute class `SimpleAttribute` may omit the `Attribute` suffix, thus shortening the name to `Simple`:

```vb
<Simple> Class Class1
End Class

<Simple> Interface Interface1
End Interface
```

The example above is semantically equivalent to the following:

```vb
<SimpleAttribute> Class Class1
End Class

<SimpleAttribute> Interface Interface1
End Interface
```

In general, attributes named with the suffix `Attribute` are preferred. The following example shows two attribute classes named `T` and `T``Attribute`.

```vb
<AttributeUsage(AttributeTargets.All)> _
Public Class T
    Inherits System.Attribute
End Class

<AttributeUsage(AttributeTargets.All)> _
Public Class TAttribute
    Inherits System.Attribute
End Class

' Refers to TAttribute.
<T> Class Class1 
End Class

' Refers to TAttribute.
<TAttribute> Class Class2 
End Class
```

Both the attribute block `<T>` and the attribute block `<TAttribute>` refer to the attribute class named `TAttribute`. It is not possible to use `T` as an attribute until you remove the declaration for class `TAttribute`.

### Attribute Arguments

Arguments to an attribute may take two forms: *positional arguments* and *instance variable/property initializers*. Any positional arguments to the attribute must precede the instance variable/property initializers. A positional argument consists of a constant expression, a one-dimensional array-creation expression or a `GetType` expression. An instance variable/property initializer consists of an identifier, which can match keywords, followed by a colon and equal sign, and terminated by a constant expression or a `GetType` expression.

Given an attribute with attribute class `T`, positional argument list `P`, and instance variable/property initializer list `N`, these steps determine whether the arguments are valid:

1. Follow the compile-time processing steps for compiling an expression of the form `New T(P)`. This either results in a compile-time error or determines a constructor on `T` that is most applicable to the argument list.

2. If the constructor determined in step 1 has parameters that are not attribute types or is inaccessible at the declaration site, a compile-time error occurs.

3. For each instance variable/property initializer `Arg` in `N`, let `Name` be the identifier of the instance variable/property initializer `Arg`. `Name` must identify a non-`Shared`, writeable, `Public` instance variable or parameterless property on `T` whose type is an attribute type. If `T` has no such instance variable or property, a compile-time error occurs.

For example:

```vb
<AttributeUsage(AttributeTargets.All)> _
Public Class GeneralAttribute
    Inherits Attribute

    Public Sub New(x As Integer)
    End Sub

    Public Sub New(x As Double)
    End Sub

    Public y As Type

    Public Property z As Integer
        Get
        End Get

        Set
        End Set
    End Property
End Class

' Calls the first constructor.
<General(10, z:=30, y:=GetType(Integer))> _
Class C1
End Class

' Calls the second constructor.
<General(10.5, z:=10)> _
Class C2
End Class
```

Type parameters cannot be used anywhere in attribute arguments. However, constructed types may be used:

```vb
<AttributeUsage(AttributeTargets.All)> _
Class A 
   Inherits System.Attribute 

   Public Sub New(t As Type)
   End Sub 
End Class

Class List(Of T) 
    ' Error: attribute argument cannot use type parameter
    <A(GetType(T))> Dim t1 As T 

    ' OK: closed type
    <A(GetType(List(Of Integer)))> Dim y As Integer
End Class
```


```antlr
AttributeArguments
    : AttributePositionalArgumentList
    | AttributePositionalArgumentList Comma VariablePropertyInitializerList
    | VariablePropertyInitializerList
    ;

AttributePositionalArgumentList
    : AttributeArgumentExpression? ( Comma AttributeArgumentExpression? )*
    ;

VariablePropertyInitializerList
    : VariablePropertyInitializer ( Comma VariablePropertyInitializer )*
    ;

VariablePropertyInitializer
    : IdentifierOrKeyword ColonEquals AttributeArgumentExpression
    ;

AttributeArgumentExpression
    : ConstantExpression
    | GetTypeExpression
    | ArrayExpression
    ;
```