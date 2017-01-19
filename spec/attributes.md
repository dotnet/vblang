# Attributes

The Visual Basic language enables the programmer to specify modifiers on declarations, which represent information about the entities being declared. For example, affixing a class method with the modifiers `Public`, `Protected`, `Friend`, `Protected Friend`, or `Private` specifies its accessibility.

In addition to the modifiers defined by the language, Visual Basic also enables programmers to create new modifiers, called *attributes*, and to use them when declaring new entities. These new modifiers, which are defined through the declaration of attribute classes, are then assigned to entities through *attribute blocks*.

__Note.__ Attributes may be retrieved at run time through the .NET Framework's reflection APIs. These APIs are outside the scope of this specification.

For instance, a framework might define a `Help` attribute that can be placed on program elements such as classes and methods to provide a mapping from program elements to documentation, as the following example demonstrates:

```vb
<AttributeUsage(AttributeTargets.All)> _
Public Class HelpAttribute
    Inherits Attribute

    Public Sub New(urlValue As String)
        Me.UrlValue = urlValue
    End Sub

    Public Topic As String
    Private UrlValue As String

    Public ReadOnly Property Url() As String
        Get
            Return UrlValue
        End Get
    End Property
End Class
```

The example defines an attribute class named `HelpAttribute`, or `Help` for short, that has one positional parameter (`UrlValue`) and one named argument (`Topic`).

The next example shows several uses of the attribute:

```vb
<Help("http://www.example.com/.../Class1.htm")> _
Public Class Class1
    <Help("http://www.example.com/.../Class1.htm", Topic:="F")> _
    Public Sub F()
    End Sub
End Class
```

The next example checks to see if `Class1` has a `Help` attribute, and writes out the associated `Topic` and `Url` values if the attribute is present.

```vb
Module Test
    Sub Main()
        Dim type As Type = GetType(Class1)
        Dim arr() As Object = _
            type.GetCustomAttributes(GetType(HelpAttribute), True)

        If arr.Length = 0 Then
            Console.WriteLine("Class1 has no Help attribute.")
        Else
            Dim ha As HelpAttribute = CType(arr(0), HelpAttribute)
            Console.WriteLine("Url = " & ha.Url & ", Topic = " & ha.Topic)
        End If
    End Sub
End Module
```

## Attribute Classes

An *attribute class* is a non-generic class that derives from `System.Attribute` and is not `MustInherit`. The attribute class may have a `System.AttributeUsage` attribute that declares what the attribute is valid on, whether it may be used multiple times in a declaration, and whether it is inherited. The following example defines an attribute class named `SimpleAttribute` that can be placed on class declarations and interface declarations:

```vb
<AttributeUsage(AttributeTargets.Class Or AttributeTargets.Interface)> _
Public Class SimpleAttribute
    Inherits System.Attribute
End Class
```

The next example shows a few uses of the `Simple` attribute. Although the attribute class is named `SimpleAttribute`, uses of this attribute may omit the `Attribute` suffix, thus shortening the name to `Simple`:

```vb
<Simple> Class Class1
End Class

<Simple> Interface Interface1
End Interface
```

If the attribute lacks a `System.AttributeUsage`, then the attribute can be placed onany target (equivalent to `AttributeTargets.All`). The `System.AttributeUsage` attribute has a variable initializer, `AllowMultiple`, which specifies whether the indicated attribute can be specified more than once for a given declaration. If `AllowMultiple` for an attribute is `True`, it is a *multiple-use attribute class*, and can be specified more than once on a declaration. If `AllowMultiple` for an attribute is `False` or unspecified for an attribute, it is a *single-use attribute class*, and can be specified at most once on a declaration.

The following example defines a multiple-use attribute class named `AuthorAttribute`:

```vb
<AttributeUsage(AttributeTargets.Class, AllowMultiple:=True)> _
Public Class AuthorAttribute
    Inherits System.Attribute

    Private _Value As String

    Public Sub New(value As String)
        Me._Value = value
    End Sub

    Public ReadOnly Property Value() As String
        Get
            Return _Value
        End Get
    End Property
End Class
```

The example shows a class declaration with two uses of the `Author` attribute:

```vb
<Author("Maria Hammond"), Author("Ramesh Meyyappan")> _
Class Class1
End Class
```

The `System.AttributeUsage` attribute has a public instance variable, `Inherited`, that specifies whether the attribute, when specified on a base type, is also inherited by types that derive from this base type. If the `Inherited` public instance variable is not initialized, a default value of `False` is used. Properties and events do not inherit attributes, although the methods defined by properties and events do. Interfaces do not inherit attributes.

If a single-use attribute is both inherited and specified on a derived type, the attribute specified on the derived type overrides the inherited attribute. If a multiple-use attribute is both inherited and specified on a derived type, both attributes are specified on the derived type. For example:

```vb
<AttributeUsage(AttributeTargets.Class, AllowMultiple:=True, _
                Inherited:=True) > _
Class MultiUseAttribute
    Inherits System.Attribute

    Public Sub New(value As Boolean)
    End Sub
End Class

<AttributeUsage(AttributeTargets.Class, Inherited:=True)> _
Class SingleUseAttribute
    Inherits Attribute

    Public Sub New(value As Boolean)
    End Sub
End Class

<SingleUse(True), MultiUse(True)> Class Base
End Class

' Derived has three attributes defined on it: SingleUse(False),
' MultiUse(True) and MultiUse(False)
<SingleUse(False), MultiUse(False)> _
Class Derived
    Inherits Base
End Class
```

The positional parameters of the attribute are defined by the parameters of the public constructors of the attribute class. Positional parameters must be `ByVal` and may not specify `ByRef`. Public instance variables and properties are defined by public read-write properties or instance variables of the attribute class. The types that can be used in positional parameters and public instance variables and properties are restricted to attribute types. A type is an attribute type if it is one of the following:

* Any primitive type except for `Date` and `Decimal`.

* The type `Object`.

* The type `System.Type`.

* An enumerated type, provided that it and the types in which it is nested (if any) have `Public` accessibility.

* A one-dimensional array of one of the previous types in this list.

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