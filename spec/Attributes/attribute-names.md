Visual Basic Specification / Attributes / **Attribute Names**  
**Sub-Topics:** [Attribute Classes](Attribute-Classes), [Attribute Blocks](Attribrute-Blocks), **[Attribute Names](#Attribute-Names)**, [Attribute Arguments](Attribute-Arguments)

----

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
