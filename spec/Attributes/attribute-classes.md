Visual Basic Specification / Attributes / **Attribute Classes**  
**Sub-Topics:** **[Attribute Classes](#Attribute-Classes)**, [Attribute Blocks](Attribrute-Blocks), [Attribute Names](Attribute-Names), [Attribute Arguments](Attribute-Arguments)

----

## Attribute Classes

An *attribute class* is a non-generic class that derives from `System.Attribute` and is not `MustInherit`. The attribute class may have a `System.AttributeUsage` attribute that declares what the attribute is valid on, whether it may be used multiple times in a declaration, and whether it is inherited. The following example defines an attribute class named `SimpleAttribute` that can be placed on class declarations and interface declarations:

```vb
<AttributeUsage(AttributeTargets.Class Or AttributeTargets.Interface)>
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

If the attribute lacks a `System.AttributeUsage`, then the attribute can be placed on any target (equivalent to `AttributeTargets.All`). The `System.AttributeUsage` attribute has a variable initializer, `AllowMultiple`, which specifies whether the indicated attribute can be specified more than once for a given declaration. If `AllowMultiple` for an attribute is `True`, it is a *multiple-use attribute class*, and can be specified more than once on a declaration. If `AllowMultiple` for an attribute is `False` or unspecified for an attribute, it is a *single-use attribute class*, and can be specified at most once on a declaration.

The following example defines a multiple-use attribute class named `AuthorAttribute`:

```vb
<AttributeUsage(AttributeTargets.Class, AllowMultiple:=True)>
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
<AttributeUsage(AttributeTargets.Class, AllowMultiple:=True,
                Inherited:=True) >
Class MultiUseAttribute
    Inherits System.Attribute

    Public Sub New(value As Boolean)
    End Sub
End Class

<AttributeUsage(AttributeTargets.Class, Inherited:=True)>
Class SingleUseAttribute
    Inherits Attribute

    Public Sub New(value As Boolean)
    End Sub
End Class

<SingleUse(True), MultiUse(True)> Class Base
End Class

' Derived has three attributes defined on it: SingleUse(False),
' MultiUse(True) and MultiUse(False)
<SingleUse(False), MultiUse(False)>
Class Derived
    Inherits Base
End Class
```

The positional parameters of the attribute are defined by the parameters of the public constructors of the attribute class. Positional parameters must be `ByVal` and may not specify `ByRef`. Public instance variables and properties are defined by public read-write properties or instance variables of the attribute class. The types that can be used in positional parameters and public instance variables and properties are restricted to attribute types. A type is an attribute type if it is one of the following:

* Any primitive type except for `System.Date` and `System.Decimal`.

* The type `System.Object`.

* The type `System.Type`.

* An enumerated type, provided that it and the types in which it is nested (if any) have `Public` accessibility.

* A one-dimensional array of one of the previous types in this list.
