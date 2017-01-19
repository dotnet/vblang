# General Concepts

This chapter covers a number of concepts that are required to understand the semantics of the Microsoft Visual Basic language. Many of the concepts should be familiar to Visual Basic programmers or C/C++ programmers, but their precise definitions may differ.

## Declarations

A Visual Basic program is made up of named entities. These entities are introduced through *declarations* and represent the "meaning" of the program.

At a top level, *namespaces* are entities that organize other entities, such as nested namespaces and types. *Types* are entities that describe values and define executable code. Types may contain nested types and type members. *Type members* are constants, variables, methods, operators, properties, events, enumeration values, and constructors.

An entity that can contain other entities defines a *declaration space*. Entities are introduced into a declaration space either through declarations or inheritance; the containing declaration space is called the entities' *declaration context*. Declaring an entity in a declaration space in turn defines a new declaration space that can contain further nested entity declarations; thus, the declarations in a program form a hierarchy of declaration spaces.

Except in the case of overloaded type members, it is invalid for declarations to introduce identically named entities of the same kind into the same declaration context. Additionally, a declaration space may never contain different kinds of entities with the same name; for example, a declaration space may never contain a variable and a method by the same name.

__Note.__ It may be possible in other languages to create a declaration space that contains different kinds of entities with the same name (for example, if the language is case sensitive and allows different declarations based on casing). In that situation, the most accessible entity is considered bound to that name; if more than one type of entity is most accessible then the name is ambiguous. `Public` is more accessible than `Protected Friend`, `Protected Friend` is more accessible than `Protected` or `Friend`, and `Protected` or `Friend` is more accessible than `Private`.

The declaration space of a namespace is "open ended," so two namespace declarations with the same fully qualified name contribute to the same declaration space. In the example below, the two namespace declarations contribute to the same declaration space, in this case declaring two classes with the fully qualified names `Data.Customer` and `Data.Order:`

```vb
Namespace Data
    Class Customer
    End Class
End Namespace

Namespace Data
    Class Order
    End Class
End Namespace
```

Because the two declarations contribute to the same declaration space, a compile-time error would occur if each contained a declaration of a class with the same name.

### Overloading and Signatures

The only way to declare identically named entities of the same kind in a declaration space is through *overloading*. Only methods, operators, instance constructors, and properties may be overloaded.

Overloaded type members must possess unique signatures. The signature of a type member consists of the number of type parameters, and the number and types of the member's parameters. Conversion operators also include the return type of the operator in the signature.

The following are not part of a member's signature, and hence cannot be overloaded on:

* Modifiers to a type member (for example, `Shared` or `Private`).

* Modifiers to a parameter (for example, `ByVal` or `ByRef`).

* The names of the parameters.

* The return type of a method or operator (except for conversion operators) or the element type of a property.

* Constraints on a type parameter.

The following example shows a set of overloaded method declarations along with their signatures. This declaration would not be valid since several of the method declarations have identical signatures.

```vb
Interface ITest
    Sub F1()                              ' Signature is ().
    Sub F2(x As Integer)                  ' Signature is (Integer).
    Sub F3(ByRef x As Integer)            ' Signature is (Integer).
    Sub F4(x As Integer, y As Integer)    ' Signature is (Integer, Integer).
    Function F5(s As String) As Integer   ' Signature is (String).
    Function F6(x As Integer) As Integer  ' Signature is (Integer).
    Sub F7(a() As String)                 ' Signature is (String()).
    Sub F8(ParamArray a() As String)      ' Signature is (String()).
    Sub F9(Of T)()                        ' Signature is !1().
    Sub F10(Of T, U)(x As T, y As U)      ' Signature is !2(!1, !2)
    Sub F11(Of U, T)(x As T, y As U)      ' Signature is !2(!2, !1)
    Sub F12(Of T)(x As T)                 ' Signature is !1(!1)
    Sub F13(Of T As IDisposable)(x As T)  ' Signature is !1(!1)
End Interface
```

It is valid to define a generic type that may contain members with identical signatures based on the type arguments supplied. Overload resolution rules are used to try and disambiguate between such overloads, although there may be situations in which it is impossible to disambiguate. For example:

```vb
Class C(Of T)
    Sub F(x As Integer)
    End Sub

    Sub F(x As T)
    End Sub

    Sub G(Of U)(x As T, y As U)
    End Sub

    Sub G(Of U)(x As U, y As T)
    End Sub
End Class

Module Test
    Sub Main()
        Dim x As New C(Of Integer)
        x.F(10)                   ' Calls C(Of T).F(Integer)
        x.G(Of Integer)(10, 10)    ' Error: Can't choose between overloads
    End Sub
End Module
```

## Scope

The *scope* of an entity's name is the set of all declaration spaces within which it is possible to refer to that name without qualification. In general, the scope of an entity's name is its entire declaration context; however, an entity's declaration may contain nested declarations of entities with the same name. In that case, the nested entity *shadows*, or hides, the outer entity, and access to the shadowed entity is only possible through qualification.

Shadowing through nesting occurs in namespaces or types nested within namespaces, in types nested within other types, and in the bodies of type members. Shadowing through the nesting of declarations always occurs implicitly; no explicit syntax is required.

In the following example, within the `F` method, the instance variable `i` is shadowed by the local variable `i`, but within the `G` method, `i` still refers to the instance variable.

```vb
Class Test
    Private i As Integer = 0

    Sub F()
        Dim i As Integer = 1
    End Sub

    Sub G()
        i = 1
    End Sub
End Class
```

When a name in an inner scope hides a name in an outer scope, it shadows all overloaded occurrences of that name. In the following example, the call `F(1)` invokes the `F` declared in `Inner` because all outer occurrences of `F` are hidden by the inner declaration. For the same reason, the call `F("Hello")` is in error.

```vb
Class Outer
    Shared Sub F(i As Integer)
    End Sub

    Shared Sub F(s As String)
    End Sub

    Class Inner
        Shared Sub F(l As Long)
        End Sub

        Sub G()
            F(1) ' Invokes Outer.Inner.F.
            F("Hello") ' Error.
        End Sub
    End Class
End Class
```

## Inheritance

An inheritance relationship is one in which one type (the *derived* type) derives from another (the *base* type), such that the derived type's declaration space implicitly contains the accessible non-constructor type members and nested types of its base type. In the following example, class `A` is the base class of `B`, and `B` is derived from `A`.

```vb
Class A
End Class

Class B
    Inherits A
End Class
```

Since `A` does not explicitly specify a base class, its base class is implicitly `Object`.

The following are important aspects of inheritance:

* Inheritance is transitive. If type *C* is derived from type *B*, and type *B* is derived from type *A*, type *C* inherits the type members declared in type *B* as well as the type members declared in type *A*.

* A derived type extends, but cannot narrow, its base type. A derived type can add new type members, and it can shadow inherited type members, but it cannot remove the definition of an inherited type member.

* Because an instance of a type contains all of the type members of its base type, a conversion always exists from a derived type to its base type.

* All types must have a base type, except for the type `Object`. Thus, `Object` is the ultimate base type of all types, and all types can be converted to it.

* Circularity in derivation is not permitted. That is, when a type `B` derives from a type `A`, it is an error for type `A` to derive directly or indirectly from type `B`.

* A type may not directly or indirectly derive from a type nested within it.

The following example produces a compile-time error because the classes circularly depend on each other.

```vb
Class A
    Inherits B
End Class

Class B
    Inherits C
End Class

Class C
    Inherits A
End Class
```

The following example also produces a compile-time error because `B` indirectly derives from its nested class `C` through class `A`.

```vb
Class A
    Inherits B.C
End Class

Class B
    Inherits A

    Public Class C
    End Class 
End Class
```

The next example does not produce an error because class `A` does not derive from class `B`.

```vb
Class A
    Class B
        Inherits A
    End Class 
End Class
```

### MustInherit and NotInheritable Classes

A `MustInherit` class is an incomplete type that can act only as a base type. A `MustInherit` class cannot be instantiated, so it is an error to use the `New` operator on one. It is valid to declare variables of `MustInherit` classes; such variables can only be assigned `Nothing` or a value that is of a class derived from the `MustInherit` class.

When a regular class is derived from a `MustInherit` class, the regular class must override all inherited `MustOverride` members. For example:

```vb
MustInherit Class A
    Public MustOverride Sub F()
End Class

MustInherit Class B
    Inherits A

    Public Sub G()
    End Sub
End Class 

Class C
    Inherits B

    Public Overrides Sub F()
    End Sub 
End Class
```

The `MustInherit` class `A` introduces a `MustOverride` method `F`. Class `B` introduces an additional method `G`, but does not provide an implementation of `F`. Class `B` must therefore also be declared `MustInherit`. Class `C` overrides `F` and provides an actual implementation. Since there are no outstanding `MustOverride` members in class `C`, it is not required to be `MustInherit`.

A `NotInheritable` class is a class from which another class cannot be derived. `NotInheritable` classes are primarily used to prevent unintended derivation.

In this example, class `B` is in error because it attempts to derive from the `NotInheritable` class `A`. A class cannot be marked both `MustInherit` and `NotInheritable`.

```vb
NotInheritable Class A
End Class

Class B
    ' Error, a class cannot derive from a NotInheritable class.
    Inherits A
End Class
```

### Interfaces and Multiple Inheritance

Unlike other types, which only derive from a single base type, an interface may derive from multiple base interfaces. Because of this, an interface can inherit an identically named type member from different base interfaces. In such a case, the multiply-inherited name is not available in the derived interface, and referring to any of those type members through the derived interface causes a compile-time error, regardless of signatures or overloading. Instead, conflicting type members must be referenced through a base interface name.

In the following example, the first two statements cause compile-time errors because the multiply-inherited member `Count` is not available in interface `IListCounter`:

```vb
Interface IList
    Property Count() As Integer
End Interface

Interface ICounter
    Sub Count(i As Integer)
End Interface

Interface IListCounter
    Inherits IList
    Inherits ICounter 
End Interface 

Module Test
    Sub F(x As IListCounter)
        x.Count(1)                  ' Error, Count is not available.
        x.Count = 1                 ' Error, Count is not available.
        CType(x, IList).Count = 1   ' Ok, invokes IList.Count.
        CType(x, ICounter).Count(1) ' Ok, invokes ICounter.Count.
    End Sub 
End Module
```

As illustrated by the example, the ambiguity is resolved by casting `x` to the appropriate base interface type. Such casts have no run-time costs; they merely consist of viewing the instance as a less-derived type at compile time.

When a single type member is inherited from the same base interface through multiple paths, the type member is treated as if it were only inherited once. In other words, the derived interface only contains one instance of each type member inherited from a particular base interface. For example:

```vb
Interface IBase
    Sub F(i As Integer)
End Interface

Interface ILeft
    Inherits IBase
End Interface

Interface IRight
    Inherits IBase
End Interface

Interface IDerived
    Inherits ILeft, IRight
End Interface

Class Derived
    Implements IDerived

    ' Only have to implement F once.
    Sub F(i As Integer) Implements IDerived.F
    End Sub
End Class
```

If a type member name is shadowed in one path through the inheritance hierarchy, then the name is shadowed in all paths. In the following example, the `IBase.F` member is shadowed by the `ILeft.F` member, but is not shadowed in `IRight`:

```vb
Interface IBase
    Sub F(i As Integer)
End Interface 

Interface ILeft
    Inherits IBase

    Shadows Sub F(i As Integer)
End Interface 

Interface IRight
    Inherits IBase

    Sub G()
End Interface 

Interface IDerived
    Inherits ILeft, IRight 
End Interface 

Class Test
    Sub H(d As IDerived)
        d.F(1)                  ' Invokes ILeft.F.
        CType(d, IBase).F(1)    ' Invokes IBase.F.
        CType(d, ILeft).F(1)    ' Invokes ILeft.F.
        CType(d, IRight).F(1)   ' Invokes IBase.F.
    End Sub 
End Class
```

The invocation `d.F(1)` selects `ILeft.F`, even though `IBase.F` appears to not be shadowed in the access path that leads through `IRight`. Because the access path from `IDerived` to `ILeft` to `IBase` shadows `IBase.F`, the member is also shadowed in the access path from `IDerived` to `IRight` to `IBase`.

### Shadowing

A derived type shadows the name of an inherited type member by re-declaring it. Shadowing a name does not remove the inherited type members with that name; it merely makes all of the inherited type members with that name unavailable in the derived class. The shadowing declaration may be any type of entity.

Entities than can be overloaded can choose one of two forms of shadowing. *Shadowing by name* is specified using the `Shadows` keyword. An entity that shadows by name hides everything by that name in the base class, including all overloads. *Shadowing by name and signature* is specified using the `Overloads` keyword. An entity that shadows by name and signature hides everything by that name with the same signature as the entity. For example:

```vb
Class Base
    Sub F()
    End Sub

    Sub F(i As Integer)
    End Sub

    Sub G()
    End Sub

    Sub G(i As Integer)
    End Sub
End Class

Class Derived
    Inherits Base

    ' Only hides F(Integer).
    Overloads Sub F(i As Integer)
    End Sub

    ' Hides G() and G(Integer).
    Shadows Sub G(i As Integer)
    End Sub
End Class

Module Test
    Sub Main()
        Dim x As New Derived()

        x.F() ' Calls Base.F().
        x.G() ' Error: Missing parameter.
    End Sub
End Module
```

Shadowing a method with a `ParamArray` argument by name and signature hides only the individual signature, not all possible expanded signatures. This is true even if the signature of the shadowing method matches the unexpanded signature of the shadowed method. The following example:

```vb
Class Base
    Sub F(ParamArray x() As Integer)
        Console.WriteLine("Base")
    End Sub
End Class

Class Derived 
    Inherits Base

    Overloads Sub F(x() As Integer)
        Console.WriteLine("Derived")
    End Sub
End Class

Module Test
    Sub Main
        Dim d As New Derived()
        d.F(10)
    End Sub
End Module
```

prints `Base`, even though `Derived.F` has the same signature as the unexpanded form of `Base.F`.

Conversely, a method with a `ParamArray` argument only shadows methods with the same signature, not all possible expanded signatures. The following example:

```vb
Class Base
    Sub F(x As Integer)
        Console.WriteLine("Base")
    End Sub
End Class

Class Derived
    Inherits Base

    Overloads Sub F(ParamArray x() As Integer)
        Console.WriteLine("Derived")
    End Sub
End Class

Module Test
    Sub Main()
        Dim d As New Derived()
        d.F(10)
    End Sub
End Module
```

prints `Base`, even though `Derived.F` has an expanded form that has the same signature as `Base.F`.

A shadowing method or property that does not specify `Shadows` or `Overloads` assumes `Overloads` if the method or property is declared `Overrides`, `Shadows` otherwise. If one member of a set of overloaded entities specifies the `Shadows` or `Overloads` keyword, they all must specify it. The `Shadows` and `Overloads` keywords cannot be specified at the same time. Neither `Shadows` nor `Overloads` can be specified in a standard module; members in a standard module implicitly shadow members inherited from `Object`.

It is valid to shadow the name of a type member that has been multiply-inherited through interface inheritance (and which is thereby unavailable), thus making the name available in the derived interface.

For example:

```vb
Interface ILeft
    Sub F()
End Interface

Interface IRight
    Sub F()
End Interface

Interface ILeftRight
    Inherits ILeft, IRight

    Shadows Sub F()
End Interface

Module Test
    Sub G(i As ILeftRight)
        i.F() ' Calls ILeftRight.F.
        CType(i, ILeft).F() ' Calls ILeft.F.
        CType(i, IRight).F() ' Calls IRight.F.
    End Sub
End Module
```

Because methods are allowed to shadow inherited methods, it is possible for a class to contain several `Overridable` methods with the same signature. This does not present an ambiguity problem, since only the most-derived method is visible. In the following example, the `C` and `D` classes contain two `Overridable` methods with the same signature:

```vb
Class A
    Public Overridable Sub F()
        Console.WriteLine("A.F")
    End Sub 
End Class 

Class B
    Inherits A

    Public Overrides Sub F()
        Console.WriteLine("B.F")
    End Sub 
End Class 

Class C
    Inherits B

    Public Shadows Overridable Sub F()
        Console.WriteLine("C.F")
    End Sub 
End Class 

Class D
    Inherits C

    Public Overrides Sub F()
        Console.WriteLine("D.F")
    End Sub 
End Class 

Module Test
    Sub Main()
        Dim d As New D()
        Dim a As A = d
        Dim b As B = d
        Dim c As C = d
        a.F()
        b.F()
        c.F()
        d.F()
    End Sub 
End Module
```

There are two `Overridable` methods here: one introduced by class `A` and the one introduced by class `C`. The method introduced by class `C` hides the method inherited from class `A`. Thus, the `Overrides` declaration in class `D` overrides the method introduced by class `C`, and it is not possible for class `D` to override the method introduced by class `A`. The example produces the output:

```
B.F
B.F
D.F
D.F
```

It is possible to invoke the hidden `Overridable` method by accessing an instance of class `D` through a less-derived type in which the method is not hidden.

It is not valid to shadow a `MustOverride` method, because in most cases this would make the class unusable. For example:

```vb
MustInherit Class Base
    Public MustOverride Sub F()
End Class

MustInherit Class Derived
    Inherits Base

    Public Shadows Sub F()
    End Sub
End Class

Class MoreDerived
    Inherits Derived

    ' Error: MustOverride method Base.F is not overridden.
End Class
```

In this case, the class `MoreDerived` is required to override the `MustOverride` method `Base.F`, but because the class `Derived` shadows `Base.F`, this is not possible. There is no way to declare a valid descendent of `Derived`.

In contrast to shadowing a name from an outer scope, shadowing an accessible name from an inherited scope causes a warning to be reported, as in the following example:

```vb
Class Base
    Public Sub F()
    End Sub

    Private Sub G()
    End Sub 
End Class

Class Derived
    Inherits Base

    Public Sub F() ' Warning: shadowing an inherited name.
    End Sub

    Public Sub G() ' No warning, Base.G is not accessible here.
    End Sub
End Class
```

The declaration of method `F` in class `Derived` causes a warning to be reported. Shadowing an inherited name is specifically not an error, since that would preclude separate evolution of base classes. For example, the above situation might have come about because a later version of class `Base` introduced a method `F` that was not present in an earlier version of the class. Had the above situation been an error, *any* change made to a base class in a separately versioned class library could potentially cause derived classes to become invalid.

The warning caused by shadowing an inherited name can be eliminated through use of the `Shadows` or `Overloads` modifier:

```vb
Class Base
    Public Sub F()
    End Sub 
End Class 

Class Derived
    Inherits Base

    Public Shadows Sub F() 'OK.
    End Sub
End Class
```

The `Shadows` modifier indicates the intention to shadow the inherited member. It is not an error to specify the `Shadows` or `Overloads` modifier if there is no type member name to shadow.

A declaration of a new member shadows an inherited member only within the scope of the new member, as in the following example:

```vb
Class Base
    Public Shared Sub F()
    End Sub 
End Class 

Class Derived
    Inherits Base

    Private Shared Shadows Sub F() ' Shadows Base.F in class Derived only.
    End Sub 
End Class 

Class MoreDerived
    Inherits Derived

    Shared Sub G()
        F() ' Invokes Base.F.
    End Sub 
End Class
```

In the example above, the declaration of method `F` in class `Derived` shadows the method `F` that was inherited from class `Base`, but since the new method `F` in class `Derived` has `Private` access, its scope does not extend to class `MoreDerived`. Thus, the call `F()` in `MoreDerived.G` is valid and will invoke `Base.F`. In the case of overloaded type members, the entire set of overloaded type members is treated as if they all had the most permissive access for the purposes of shadowing.

```vb
Class Base
    Public Sub F()
    End Sub
End Class

Class Derived
    Inherits Base

    Private Shadows Sub F()
    End Sub

    Public Shadows Sub F(i As Integer)
    End Sub
End Class

Class MoreDerived
    Inherits Derived

    Public Sub G()
        F()   ' Error. No accessible member with this signature.
    End Sub
End Class
```

In this example, even though the declaration of `F()` in `Derived` is declared with `Private` access, the overloaded `F(Integer)` is declared with `Public` access. Therefore, for the purpose of shadowing, the name `F` in `Derived` is treated as if it was `Public`, so both methods shadow `F` in `Base`.

## Implementation

An *implementation* relationship exists when a type declares that it implements an interface and the type implements all the type members of the interface. A type that implements a particular interface is convertible to that interface. Interfaces cannot be instantiated, but it is valid to declare variables of interfaces; such variables can only be assigned a value that is of a class that implements the interface. For example:

```vb
Interface ITestable
    Function Test(value As Byte) As Boolean
End Interface

Class TestableClass
    Implements ITestable

    Function Test(value As Byte) As Boolean Implements ITestable.Test
        Return value > 128
    End Function
End Class

Module Test
    Sub F()
        Dim x As ITestable = New TestableClass
        Dim b As Boolean

        b = x.Test(34)
    End Sub
End Module
```

A type implementing an interface with multiply-inherited type members must still implement those methods, even though they cannot be accessed directly from the derived interface being implemented. For example:

```vb
Interface ILeft
    Sub Test()
End Interface

Interface IRight
    Sub Test()
End Interface

Interface ILeftRight
    Inherits ILeft, IRight
End Interface

Class LeftRight
    Implements ILeftRight

    ' Has to reference ILeft explicitly.
    Sub TestLeft() Implements ILeft.Test
    End Sub

    ' Has to reference IRight explicitly.
    Sub TestRight() Implements IRight.Test
    End Sub

    ' Error: Test is not available in ILeftRight.
    Sub TestLeftRight() Implements ILeftRight.Test
    End Sub
End Class
```

Even `MustInherit` classes must provide implementations of all the members of implemented interfaces; however, they can defer implementation of these methods by declaring them as `MustOverride`. For example:

```vb
Interface ITest
    Sub Test1()
    Sub Test2()
End Interface

MustInherit Class TestBase
    Implements ITest

    ' Provides an implementation.
    Sub Test1() Implements ITest.Test1
    End Sub

    ' Defers implementation.
    MustOverride Sub Test2() Implements ITest.Test2
End Class

Class TestDerived
    Inherits TestBase

    ' Have to implement MustOverride method.
    Overrides Sub Test2()
    End Sub
End Class
```

A type may choose to re-implement an interface that its base type implements. To re-implement the interface, the type must explicitly state that it implements the interface. A type re-implementing an interface may choose to re-implement only some, but not all, of the members of the interface -- any members not re-implemented continue to use the base type's implementation. For example:

```vb
Class TestBase
    Implements ITest

    Sub Test1() Implements ITest.Test1
        Console.WriteLine("TestBase.Test1")
    End Sub

    Sub Test2() Implements ITest.Test2
        Console.WriteLine("TestBase.Test2")
    End Sub
End Class

Class TestDerived
    Inherits TestBase
    Implements ITest  ' Required to re-implement

    Sub DerivedTest1() Implements ITest.Test1
        Console.WriteLine("TestDerived.DerivedTest1")
    End Sub
End Class

Module Test
    Sub Main()
        Dim Test As ITest = New TestDerived()
        Test.Test1()
        Test.Test2()
    End Sub
End Module
```

This example prints:

```
TestDerived.DerivedTest1
TestBase.Test2
```

When a derived type implements an interface whose base interfaces are implemented by the derived type's base types, the derived type can choose to only implement the interface's type members that are not already implemented by the base types. For example:

```vb
Interface IBase
    Sub Base()
End Interface

Interface IDerived
    Inherits IBase

    Sub Derived()
End Interface

Class Base
    Implements IBase

    Public Sub Base() Implements IBase.Base
    End Sub
End Class

Class Derived
    Inherits Base
    Implements IDerived

    ' Required: IDerived.Derived not implemented by Base.
    Public Sub Derived() Implements IDerived.Derived
    End Sub
End Class
```

An interface method can also be implemented using an overridable method in a base type. In that case, a derived type may also override the overridable method and alter the implementation of the interface. For example:

```vb
Class Base
    Implements ITest

    Public Sub Test1() Implements ITest.Test1
        Console.WriteLine("TestBase.Test1")
    End Sub

    Public Overridable Sub Test2() Implements ITest.Test2
        Console.WriteLine("TestBase.Test2")
    End Sub
End Class

Class Derived
    Inherits Base

    ' Overrides base implementation.
    Public Overrides Sub Test2()
        Console.WriteLine("TestDerived.Test2")
    End Sub
End Class
```

### Implementing Methods

A type *implements* a type member of an implemented interface by supplying a method with an `Implements` clause. The two type members must have the same number of parameters, all of the types and modifiers of the parameters must match, including the default value of optional parameters, the return type must match, and all of the constraints on method parameters must match. For example:

```vb
Interface ITest
    Sub F(ByRef x As Integer)
    Sub G(Optional y As Integer = 20)
    Sub H(Paramarray z() As Integer)
End Interface

Class Test
    Implements ITest

    ' Error: ByRef/ByVal mismatch.
    Sub F(x As Integer) Implements ITest.F
    End Sub

    ' Error: Defaults do not match.
    Sub G(Optional y As Integer = 10) Implements ITest.G
    End Sub

    ' Error: Paramarray does not match.
    Sub H(z() As Integer) Implements ITest.H
    End Sub
End Class
```

A single method may implement any number of interface type members if they all meet the above criteria. For example:

```vb
Interface ITest
    Sub F(i As Integer)
    Sub G(i As Integer)
End Interface

Class Test

    Implements ITest

    Sub F(i As Integer) Implements ITest.F, ITest.G
    End Sub
End Class
```

When implementing a method in a generic interface, the implementing method must supply the type arguments that correspond to the interface's type parameters. For example:

```vb
Interface I1(Of U, V) 
    Sub M(x As U, y As List(Of V)) 
End Interface

Class C1(Of W, X)
    Implements I1(Of W, X)

    ' W corresponds to U and X corresponds to V
    Public Sub M(x As W, y As List(Of X)) Implements I1(Of W, X).M
    End Sub 
End Class

Class C2
    Implements I1(Of String, Integer)

    ' String corresponds to U and Integer corresponds to V
    Public Sub M(x As String, y As List(Of Integer)) _
        Implements I1(Of String, Integer).M
    End Sub
End Class
```

Note that it is possible that a generic interface may not be implementable for some set of type arguments.

```vb
Interface I1(Of T, U)
    Sub S1(x As T)
    Sub S1(y As U)
End Interface

Class C1
    ' Unable to implement because I1.S1 has two identical signatures
    Implements I1(Of Integer, Integer)
End Class
```

## Polymorphism

*Polymorphism* provides the ability to vary the implementation of a method or property. With polymorphism, the same method or property can perform different actions depending on the run-time type of the instance that invokes it. Methods or properties that are polymorphic are called *overridable*. By contrast, the implementation of a non-overridable method or property is invariant; the implementation is the same whether the method or property is invoked on an instance of the class in which it is declared or an instance of a derived class. When a non-overridable method or property is invoked, the compile-time type of the instance is the determining factor. For example:

```vb
Class Base
    Public Overridable Property X() As Integer
        Get
        End Get

        Set
        End Set
    End Property
End Class

Class Derived
    Inherits Base

    Public Overrides Property X() As Integer
        Get
        End Get

        Set
        End Set
    End Property
End Class

Module Test
    Sub F()
        Dim Z As Base

        Z = New Base()
        Z.X = 10            ' Calls Base.X
        Z = New Derived()
        Z.X = 10            ' Calls Derived.X
    End Sub
End Module
```

An overridable method may also be `MustOverride`, which means that it provides no method body and must be overridden. `MustOverride` methods are only allowed in `MustInherit` classes.

In the following example, the class `Shape` defines the abstract notion of a geometrical shape object that can paint itself:

```vb
MustInherit Public Class Shape
    Public MustOverride Sub Paint(g As Graphics, r As Rectangle)
End Class 

Public Class Ellipse
    Inherits Shape

    Public Overrides Sub Paint(g As Graphics, r As Rectangle)
        g.drawEllipse(r)
    End Sub 
End Class 

Public Class Box
    Inherits Shape

    Public Overrides Sub Paint(g As Graphics, r As Rectangle)
        g.drawRect(r)
    End Sub 
End Class
```

The `Paint` method is `MustOverride` because there is no meaningful default implementation. The `Ellipse` and `Box` classes are concrete `Shape` implementations. Because these classes are not `MustInherit`, they are required to override the `Paint` method and provide an actual implementation.

It is an error for a base access to reference a `MustOverride` method, as the following example demonstrates:

```vb
MustInherit Class A
    Public MustOverride Sub F()
End Class

Class B
    Inherits A

    Public Overrides Sub F()
        MyBase.F() ' Error, MyBase.F is MustOverride.
    End Sub 
End Class
```

An error is reported for the `MyBase.F()` invocation because it references a `MustOverride` method.

### Overriding Methods

A type may *override* an inherited overridable method by declaring a method with the same name and , signature, and marking the declaration with the `Overrides` modifier.There are additional requirements on overriding methods, listed below. Whereas an `Overridable` method declaration introduces a new method, an `Overrides` method declaration replaces the inherited implementation of the method.

An overriding method may be declared `NotOverridable`, which prevents any further overriding of the method in derived types. In effect, `NotOverridable` methods become non-overridable in any further derived classes.

Consider the following example:

```vb
Class A
    Public Overridable Sub F()
        Console.WriteLine("A.F")
    End Sub

    Public Overridable Sub G()
        Console.WriteLine("A.G")
    End Sub
End Class

Class B
    Inherits A

    Public Overrides NotOverridable Sub F()
        Console.WriteLine("B.F")
    End Sub

    Public Overrides Sub G()
        Console.WriteLine("B.G")
    End Sub
End Class

Class C
    Inherits B

    Public Overrides Sub G()
        Console.WriteLine("C.G")
    End Sub
End Class
```

In the example, class `B` provides two `Overrides` methods: a method `F` that has the `NotOverridable` modifier and a method `G` that does not. Use of the `NotOverridable` modifier prevents class `C` from further overriding method `F`.

An overriding method may also be declared `MustOverride`, even if the method that it is overriding is not declared `MustOverride`. This requires that the containing class be declared `MustInherit` and that any further derived classes that are not declared `MustInherit` must override the method. For example:

```vb
Class A
    Public Overridable Sub F()
        Console.WriteLine("A.F")
    End Sub
End Class

MustInherit Class B
    Inherits A

    Public Overrides MustOverride Sub F()
End Class
```

In the example, class `B` overrides `A.F` with a `MustOverride` method. This means that any classes derived from `B` will have to override `F`, unless they are declared `MustInherit` as well.

A compile-time error occurs unless all of the following are true of an overriding method:

* The declaration context contains a single accessible inherited method with the same signature and return type (if any) as the overriding method.
* The inherited method being overridden is overridable. In other words, the inherited method being overridden is not `Shared` or `NotOverridable`.
* The accessibility domain of the method being declared is the same as the accessibility domain of the inherited method being overridden. There is one exception: a `Protected Friend` method must be overridden by a `Protected` method if the other method is in another assembly that the overriding method does not have `Friend` access to.
* The parameters of the overriding method match the overridden method's parameters in regards to usage of the `ByVal`, `ByRef`, `ParamArray,` and `Optional` modifiers, including the values provided for optional parameters.
* The type parameters of the overriding method match the overridden method's type parameters in regards to type constraints.

When overriding a method in a base generic type, the overriding method must supply the type arguments that correspond to the base type parameters. For example:

```vb
Class Base(Of U, V) 
    Public Overridable Sub M(x As U, y As List(Of V)) 
    End Sub
End Class

Class Derived(Of W, X)
    Inherits Base(Of W, X)

    ' W corresponds to U and X corresponds to V
    Public Overrides Sub M(x As W, y As List(Of X)) 
    End Sub 
End Class

Class MoreDerived
    Inherits Derived(Of String, Integer)

    ' String corresponds to U and Integer corresponds to V
    Public Overrides Sub M(x As String, y As List(Of Integer))
    End Sub
End Class
```

Note that it is possible that an overridable method in a generic class may not be able to be overridden for some sets of type arguments. If the method is declared `MustOverride`, this means that some inheritance chains may not be possible. For example:

```vb
MustInherit Class Base(Of T, U)
    Public MustOverride Sub S1(x As T)
    Public MustOverride Sub S1(y As U)
End Class

Class Derived
    Inherits Base(Of Integer, Integer)

    ' Error: Can't override both S1's at once
    Public Overrides Sub S1(x As Integer)
    End Sub
End Class
```

An override declaration can access the overridden base method using a base access, as in the following example:

```vb
Class Base
    Private x As Integer

    Public Overridable Sub PrintVariables()
        Console.WriteLine("x = " & x)
    End Sub
End Class

Class Derived
    Inherits Base

    Private y As Integer

    Public Overrides Sub PrintVariables()
        MyBase.PrintVariables()
        Console.WriteLine("y = " & y)
    End Sub
End Class
```

In the example, the invocation of `MyBase.PrintVariables()` in class `Derived` invokes the `PrintVariables` method declared in class `Base`. A base access disables the overridable invocation mechanism and simply treats the base method as a non-overridable method. Had the invocation in `Derived` been written `CType(Me, Base).PrintVariables()`, it would recursively invoke the `PrintVariables` method declared in `Derived`, not the one declared in `Base`.

Only when it includes an `Overrides` modifier can a method override another method. In all other cases, a method with the same signature as an inherited method simply shadows the inherited method, as in the example below:

```vb
Class Base
    Public Overridable Sub F()
    End Sub
End Class

Class Derived
    Inherits Base

    Public Overridable Sub F() ' Warning, shadowing inherited F().
    End Sub
End Class
```

In the example, the method `F` in class `Derived` does not include an `Overrides` modifier and therefore does not override method `F` in class `Base`. Rather, method `F` in class `Derived` shadows the method in class `Base`, and a warning is reported because the declaration does not include a `Shadows` or `Overloads` modifier.

In the following example, method `F` in class `Derived` shadows the overridable method `F` inherited from class `Base`:

```vb
Class Base
    Public Overridable Sub F()
    End Sub
End Class

Class Derived
    Inherits Base

    Private Shadows Sub F() ' Shadows Base.F within Derived.
    End Sub
End Class

Class MoreDerived
    Inherits Derived

    Public Overrides Sub F() ' Ok, overrides Base.F.
    End Sub
End Class
```

Since the new method `F` in class `Derived` has `Private` access, its scope only includes the class body of `Derived` and does not extend to class `MoreDerived`. The declaration of method `F` in class `MoreDerived` is therefore permitted to override the method `F` inherited from class `Base`.

When an `Overridable` method is invoked, the most derived implementation of the instance method is called, based on the type of the instance, regardless of whether the call is to the method in the base class or the derived class. The most derived implementation of an `Overridable` method `M` with respect to a class `R` is determined as follows:

* If `R` contains the introducing `Overridable` declaration of `M`, this is the most derived implementation of `M`.

* Otherwise, if `R` contains an override of `M`, this is the most derived implementation of `M`.

* Otherwise, the most derived implementation of `M` is the same as that of the direct base class of `R`.

## Accessibility

A declaration specifies the *accessibility* of the entity it declares. An entity's accessibility does not change the scope of an entity's name. The *accessibility domain* of a declaration is the set of all declaration spaces in which the declared entity is accessible.

The five access types are `Public`, `Protected`, `Friend`, `Protected Friend`, and `Private`. `Public` is the most permissive access type, and the four other types are all subsets of `Public`. The least permissive access type is `Private`, and the four other access types are all supersets of `Private`.

```antlr
AccessModifier
    : 'Public'
    | 'Protected'
    | 'Friend'
    | 'Private'
    | 'Protected' 'Friend'
    ;
```

The access type for a declaration is specified via an optional access modifier, which can be `Public`, `Protected`, `Friend`, `Private`, or the combination of `Protected` and `Friend`. If no access modifier is specified, the default access type depends on the declaration context; the permitted access types also depend on the declaration context.

* Entities declared with the `Public` modifier have `Public` access. There are no restrictions on the use of `Public` entities.

* Entities declared with the `Protected` modifier have `Protected` access. `Protected` access can only be specified on members of classes (both regular type members and nested classes) or on `Overridable` members of standard modules and structures (which must, by definition, be inherited from `System.Object` or `System.ValueType`). A `Protected` member is accessible to a derived class, provided that either the member is not an instance member, or the access takes place through an instance of the derived class. `Protected` access is not a superset of `Friend` access.

* Entities declared with the `Friend` modifier have `Friend` access. An entity with `Friend` access is accessible only within the program that contains the entity declaration or any assemblies that have been given `Friend` access through the `System.Runtime.CompilerServices.InternalsVisibleToAttribute` attribute.

* Entities declared with the `Protected Friend` modifiers have the union of `Protected` and `Friend` access.

* Entities declared with the `Private` modifier have `Private` access. A `Private` entity is accessible only within its declaration context, including any nested entities.

The accessibility in a declaration does not depend on the accessibility of the declaration context. For example, a type declared with `Private` access may contain a type member with `Public` access.

The following code demonstrates various accessibility domains:

```vb
Public Class A
    Public Shared X As Integer
    Friend Shared Y As Integer
    Private Shared Z As Integer
End Class

Friend Class B
    Public Shared X As Integer
    Friend Shared Y As Integer
    Private Shared Z As Integer

    Public Class C
        Public Shared X As Integer
        Friend Shared Y As Integer
        Private Shared Z As Integer
    End Class

    Private Class D
        Public Shared X As Integer
        Friend Shared Y As Integer
        Private Shared Z As Integer
    End Class
End Class
```

The classes and members in this example have the following accessibility domains:

* The accessibility domain of `A` and `A.X` is unlimited.

* The accessibility domain of `A.Y`, `B`, `B.X`, `B.Y`, `B.C`, `B.C.X`, and `B.C.Y` is the containing program.

* The accessibility domain of `A.Z` is `A.`

* The accessibility domain of `B.Z`, `B.D`, `B.D.X`, and `B.D.Y` is `B`, including `B.C` and `B.D`.

* The accessibility domain of `B.C.Z` is `B.C`.

* The accessibility domain of `B.D.Z` is `B.D`.

As the example illustrates, the accessibility domain of a member is never larger than that of a containing type. For example, even though all `X` members have `Public` declared accessibility, all but `A.X` have accessibility domains that are constrained by a containing type.

Access to `Protected` instance members must be through an instance of the derived type so that unrelated types cannot gain access to each other's protected members. For example:

```vb
Class User
    Protected Password As String
End Class

Class Employee
    Inherits User
End Class

Class Guest
    Inherits User

    Public Function GetPassword(u As User) As String
        ' Error: protected access has to go through derived type.
        Return U.Password
    End Function
End Class
```

In the above example, the class `Guest` only has access to the protected `Password` field if it is qualified with an instance of `Guest`. This prevents `Guest` from gaining access to the `Password` field of an `Employee` object simply by casting it to `User`.

For the purposes of `Protected` member access in generic types, the declaration context includes type parameters. This means that a derived type with one set of type arguments does not have access to the `Protected` members of a derived type with a different set of type arguments. For example:

```vb
Class Base(Of T)
    Protected x As T
End Class

Class Derived(Of T)
    Inherits Base(Of T)

    Public Sub F(y As Derived(Of String))
        ' Error: Derived(Of T) cannot access Derived(Of String)'s 
        '     protected members
        y.x = "a"
    End Sub
End Class
```

__Note.__ The C# language (and possibly other languages) allows a generic type to access `Protected` members regardless of what type arguments are supplied. This should be kept in mind when designing generic classes that contain `Protected` members.


### Constituent Types

The *constituent types* of a declaration are the types that are referenced by the declaration. For example, the type of a constant, the return type of a method and the parameter types of a constructor are all constituent types. The accessibility domain of a constituent type of a declaration must be the same as or a superset of the accessibility domain of the declaration itself. For example:

```vb
Public Class X
    Private Class Y
    End Class

    ' Error: Exposing private class Y outside of X.
    Public Function Z() As Y
    End Function

    ' Valid: Not exposing outside of X.
    Private Function A() As Y
    End Function
End Class

Friend Class B
    Private Class C
    End Class

    ' Error: Exposing private class Y outside of B.
    Public Function D() As C
    End Function
End Class
```

## Type and Namespace Names

Many language constructs require a namespace or type to be specified; these can be specified by using a qualified form of the namespace or type's name. A *qualified name* consists of a series of identifiers separated by periods; the identifier on the right side of a period is resolved in the declaration space specified by the identifier on the left side of the period.

The *fully qualified name* of a namespace or type is a qualified name that contains the name of all containing namespaces and types. In other words, the fully qualified name of a namespace or type is `N.T`, where `T` is the name of the entity and `N` is the fully qualified name of its containing entity.

The example below shows several namespace and type declarations together with their associated fully qualified names in in-line comments.

```vb
Class A            ' A.
End Class

Namespace X        ' X.
    Class B        ' X.B.
        Class C    ' X.B.C.
        End Class
    End Class

    Namespace Y    ' X.Y.
        Class D    ' X.Y.D.
        End Class
    End Namespace 
End Namespace 

Namespace X.Y      ' X.Y.
    Class E        ' X.Y.E.
    End Class
End Namespace
```

Observe that the namespace X.Y has been declared in two different locations in the source code, but these two partial declarations constitute just a single namespace called X.Y which contains both class D and class E.

In some situations, a qualified name may begin with the keyword `Global`. The keyword represents the unnamed outermost namespace, which is useful in situations where a declaration shadows an enclosing namespace. The `Global` keyword allows "escaping" out to the outermost namespace in that situation. For example:

```vb
Namespace NS1
    Class System
    End Class

    Module Test
        Sub Main()
            ' Error: Class System does not contain Int32
            Dim x As System.Int32


            ' Legal, binds to System in outermost namespace
            Dim y As Global.System.Int32
        End Sub
    End Module
End Namespace
```

In the above example, the first method call is invalid because the identifier `System` binds to the class `System`, not the namespace `System`. The only way to access the `System` namespace is to use `Global` to escape out to the outermost namespace. `Global` cannot be used in an `Imports` statement or `Namespace` declaration.

Because other languages may introduce types and namespaces that match keywords in the language, Visual Basic recognizes keywords to be part of a qualified name as long as they follow a period. Keywords used in this way are treated as identifiers. For example, the qualified identifier `X.Default.Class` is a valid qualified identifier, while `Default.Class` is not.

### Qualified Name Resolution for namespaces and types

Given a qualified namespace or type name of the form `N.R(Of A)`, where `R` is the rightmost identifier in the qualified name and `A` is an optional type argument list, the following steps describe how to determine to which namespace or type the qualified name refers:

1. Resolve `N`, using the rules for either qualified or unqualified name resolution.

2. If resolution of `N` fails, or resolves to a type parameter, a compile-time error occurs.

3. Otherwise, if `R` matches the name of a namespace in N and no type arguments were supplied, or `R` matches an accessible type in `N` with the same number of type parameters as type arguments, if any, then the qualified name refers to that namespace or type.

4. Otherwise, if `N` contains one or more standard modules, and `R` matches the name of an accessible type with the same number of type parameters as type arguments, if any, in exactly one standard module, then the qualified name refers to that type. If `R` matches the name of accessible types with the same number of type parameters as type arguments, if any, in more than one standard module, a compile-time error occurs.

5. Otherwise, a compile-time error occurs.

__Note.__ An implication of this resolution process is that type members do not shadow namespaces or types when resolving namespace or type names.

### Unqualified Name Resolution for namespaces and types

Given an unqualified name `R(Of A)`, where `A` is an optional type argument list, the following steps describe how to determine to which namespace or type the unqualified name refers:

1. If R matches the name of a type parameter of the current method, and no type arguments were supplied, then the unqualified name refers to that type parameter.

2.  For each nested type containing the name reference, starting from the innermost type and going to the outermost:
    21. If `R` matches the name of a type parameter in the current type and no type arguments were supplied, then the unqualified name refers to that type parameter.
    22. Otherwise, if `R` matches the name of an accessible nested type with the same number of type parameters as type arguments, if any, then the unqualified name refers to that type.

3. For each nested namespace containing the name reference, starting from the innermost namespace and going to the outermost namespace:
    31. If `R` matches the name of a nested namespace in the current namespace and no type argument list is supplied, then the unqualified name refers to that nested namespace.
    32. Otherwise, if `R` matches the name of an accessible type with the same number of type parameters as type arguments, if any, in the current namespace, then the unqualified name refers to that type.
    33. Otherwise, if the namespace contains one or more accessible standard modules, and `R` matches the name of an accessible nested type with the same number of type parameters as type arguments, if any, in exactly one standard module, then the unqualified name refers to that nested type. If `R` matches the name of accessible nested types with the same number of type parameters as type arguments, if any, in more than one standard module, a compile-time error occurs.

4. If the source file has one or more import aliases, and `R` matches the name of one of them, then the unqualified name refers to that import alias. If a type argument list is supplied, a compile-time error occurs.

5. If the source file containing the name reference has one or more imports:
    51. If `R` matches the name of an accessible type with the same number of type parameters as type arguments, if any, in exactly one import, then the unqualified name refers to that type. If `R` matches the name of an accessible type with the same number of type parameters as type arguments, if any, in more than one import and all are not the same type, a compile-time error occurs.
    52. Otherwise, if no type argument list was supplied and `R` matches the name of a namespace with accessible types in exactly one import, then the unqualified name refers to that namespace. If no type argument list was supplied and `R` matches the name of a namespace with accessible types in more than one import and all are not the same namespace, a compile-time error occurs.
    53. Otherwise, if the imports contain one or more accessible standard modules, and `R` matches the name of an accessible nested type with the same number of type parameters as type arguments, if any, in exactly one standard module, then the unqualified name refers to that type. If `R` matches the name of accessible nested types with the same number of type parameters as type arguments, if any, in more than one standard module, a compile-time error occurs.

6. If the compilation environment defines one or more import aliases, and `R` matches the name of one of them, then the unqualified name refers to that import alias. If a type argument list is supplied, a compile-time error occurs.

7. If the compilation environment defines one or more imports:
    71. If `R` matches the name of an accessible type with the same number of type parameters as type arguments, if any, in exactly one import, then the unqualified name refers to that type. If `R` matches the name of an accessible type with the same number of type parameters as type arguments, if any, in more than one import, a compile-time error occurs.
    72. Otherwise, if no type argument list was supplied and `R` matches the name of a namespace with accessible types in exactly one import, then the unqualified name refers to that namespace. If no type argument list was supplied and `R` matches the name of a namespace with accessible types in more than one import, a compile-time error occurs.
    73. Otherwise, if the imports contain one or more accessible standard modules, and `R` matches the name of an accessible nested type with the same number of type parameters as type arguments, if any, in exactly one standard module, then the unqualified name refers to that type. If `R` matches the name of accessible nested types with the same number of type parameters as type arguments, if any, in more than one standard module, a compile-time error occurs.

8. Otherwise, a compile-time error occurs.

__Note.__ An implication of this resolution process is that type members do not shadow namespaces or types when resolving namespace or type names.

Normally, a name can only occur once in a particular namespace. However, because namespaces can be declared across multiple .NET assemblies, it is possible to have a situation where two assemblies define a type with the same fully qualified name. In that case, a type declared in the current set of source files is preferred over a type declared in an external .NET assembly. Otherwise, the name is ambiguous and there is no way to disambiguate the name.

## Variables

A *variable* represents a storage location. Every variable has a type that determines what values can be stored in the variable. Because Visual Basic is a type-safe language, every variable in a program has a type and the language guarantees that values stored in variables are always of the appropriate type. Variables are always initialized to the default value of their type before any reference to the variable can be made. It is not possible to access uninitialized memory.

## Generic Types and Methods

Types (except for standard modules and enumerated types) and methods can declare *type parameters*, which are types that will not be provided until an instance of the type is declared or the method is invoked. Types and methods with type parameters are also known as *generic types* and *generic methods*, respectively, because the type or method must be written generically, without specific knowledge of the types that will be supplied by code that uses the type or method.

__Note.__ At this time, even though methods and delegates can be generic, properties, events and operators cannot be generic themselves. They may, however, use type parameters from the containing class.

From the perspective of the generic type or method, a type parameter is a placeholder type that will be filled in with an actual type when the type or method is used. Type arguments are substituted for the type parameters in the type or method at the point at which the type or method is used. For example, a generic stack class could be implemented as:

```vb
Public Class Stack(Of ItemType)
    Protected Items(0 To 99) As ItemType
    Protected CurrentIndex As Integer = 0

    Public Sub Push(data As ItemType)
        If CurrentIndex = 100 Then
            Throw New ArgumentException("Stack is full.")
        End If

        Items(CurrentIndex) = Data
        CurrentIndex += 1
    End Sub

    Public Function Pop() As ItemType
        If CurrentIndex = 0 Then
            Throw New ArgumentException("Stack is empty.")
        End If

        CurrentIndex -= 1
        Return Items(CurrentIndex + 1) 
    End Function
End Class
```

Declarations that use the `Stack(Of ItemType)` class must supply a type argument for the type parameter `ItemType`. This type is then filled in wherever `ItemType` is used within the class:

```vb
Option Strict On

Module Test
    Sub Main()
        Dim s1 As New Stack(Of Integer)()
        Dim s2 As New Stack(Of Double)()

        s1.Push(10.10)   ' Error: Stack(Of Integer).Push takes an Integer
        s2.Push(10.10)   ' OK: Stack(Of Double).Push takes a Double
        Console.WriteLine(s2.Pop().GetType().ToString()) ' Prints: Double
    End Sub
End Module
```

### Type Parameters

Type parameters may be supplied on type or method declarations. Each type parameter is an identifier which is a place-holder for a type argument that is supplied to create a constructed type or method. By contrast, a type argument is the actual type that is substituted for the type parameter when a generic type or method is used.

```antlr
TypeParameterList
    : OpenParenthesis 'Of' TypeParameter ( Comma TypeParameter )* CloseParenthesis
    ;

TypeParameter
    : VarianceModifier? Identifier TypeParameterConstraints?
    ;

VarianceModifier
    : 'In' | 'Out'
    ;
```

Each type parameter in a type or method declaration defines a name in the declaration space of that type or method. Thus, it cannot have the same name as another type parameter, a type member, a method parameter, or a local variable. The scope of a type parameter on a type or method is the entire type or method. Because type parameters are scoped to the entire type declaration, nested types can use outer type parameters. This also means that type parameters must always be specified when accessing types nested inside generic types:

```vb
Public Class Outer(Of T)
    Public Class Inner
        Public Sub F(x As T)
            ...
        End Sub
    End Class
End Class

Module Test
    Sub Main()
        Dim x As New Outer(Of Integer).Inner()
        ...
    End Sub
End Module
```

Unlike other members of a class, type parameters are not inherited. Type parameters in a type can only be referred to by their simple name; in other words, they cannot be qualified with the containing type name. Although it is bad programming style, the type parameters in a nested type can hide a member or type parameter declared in the outer type:

```vb
Class Outer(Of T)
    Class Inner(Of T)
        Public t1 As T    ' Refers to Inner's T
    End Class
End Class
```

Types and methods may be overloaded based on the number of type parameters (or *arity*) that the types or methods declare. For example, the following declarations are legal:

```vb
Module C
    Sub M()
    End Sub

    Sub M(Of T)()
    End Sub

    Sub M(Of T, U)()
    End Sub
End Module

Structure C(Of T)
    Dim x As T
End Structure

Class C(Of T, U)
End Class
```

In the case of types, overloads are always matched against the number of type arguments specified. This is useful when using both generic and non-generic classes together in the same program:

```vb
Class Queue 
End Class      

Class Queue(Of T)
End Class

Class X
    Dim q1 As Queue                 ' Non-generic queue
    Dim q2 As Queue(Of Integer)     ' Generic queue
End Class
```

Rules for methods overloaded on type parameters are covered in the section on method overload resolution.

Within the containing declaration, type parameters are considered full types. Since a type parameter can be instantiated with many different actual type arguments, type parameters have slightly different operations and restrictions than other types as described below:

* A type parameter cannot be used directly to declare a base class or interface.

* The rules for member lookup on type parameters depend on the constraints, if any, applied to the type parameter.

* The available conversions for a type parameter depend on the constraints, if any, applied to the type parameters.

* In the absence of a `Structure` constraint, a value with a type represented by a type parameter can be compared with `Nothing` using `Is` and `IsNot`.

* A type parameter can only be used in a `New` expression if the type parameter is constrained by a `New` or a `Structure` constraint.

* A type parameter cannot be used anywhere within an attribute exception within a `GetType` expression.

* Type parameters can be used as type arguments to other generic types and parameters.

The following example is a generic type that extends the `Stack(Of ItemType)` class:

```vb
Class MyStack(Of ItemType)
    Inherits Stack(Of ItemType)

    Public ReadOnly Property Size() As Integer
        Get
            Return CurrentIndex
        End Get
    End Property
End Class
```

When a declaration supplies a type argument to `MyStack`, the same type argument will be applied to `Stack` as well.

As a type, type parameters are purely a compile-time construct. At run-time, each type parameter is bound to a run-time type that was specified by supplying a type argument to the generic declaration. Thus, the type of a variable declared with a type parameter will, at run-time, be a non-generic type or a specific constructed type. The run-time execution of all statements and expressions involving type parameters uses the actual type that was supplied as the type argument for that parameter.


### Type Constraints

Because a type argument can be any type in the type system, a generic type or method cannot make any assumptions about a type parameter. Thus, the members of a type parameter are considered to be the members of the type `Object`, since all types derive from `Object`.

In the case of a collection like `Stack(Of ItemType)`, this fact may not be a particularly important restriction, but there may be cases where a generic type may wish to make an assumption about the types that will be supplied as type arguments. *Type constraints* can be placed on type parameters that restrict which types can be supplied as a type parameter and allow generic types or methods to assume more about type parameters.

```antlr
TypeParameterConstraints
    : 'As' Constraint
    | 'As' OpenCurlyBrace ConstraintList CloseCurlyBrace
    ;

ConstraintList
    : Constraint ( Comma Constraint )*
    ;

Constraint
    : TypeName
    | 'New'
    | 'Structure'
    | 'Class'
    ;
```


```vb
Public Class DisposableStack(Of ItemType As IDisposable)
    Implements IDisposable

    Private _items(0 To 99) As ItemType
    Private _currentIndex As Integer = 0

    Public Sub Push(data As ItemType)
        ...
    End Sub

    Public Function Pop() As ItemType
        ...
    End Function

    Private Sub Dispose() Implements IDisposable.Dispose
        For Each item As IDisposable In _items
            If item IsNot Nothing Then
                item.Dispose()
            End If
        Next item
    End Sub
End Class
```

In this example, the `DisposableStack(Of ItemType)` constrains its type parameter to only types that implement the interface `System.IDisposable`. As a result, it can implement a `Dispose` method that disposes any objects still left in the queue.

A type constraint must be one of the special constraints `Class`, `Structure`, or `New`, or it must be a type `T` where:

* `T` is a class, an interface, or a type parameter.

* `T` is not `NotInheritable`.

* `T` is not one of, or a type inherited from one of, the following special types: `System.Array`, `System.Delegate`, `System.MulticastDelegate`, `System.Enum`, or `System.ValueType`.

* `T` is not `Object`. Since all types derive from `Object`, such a constraint would have no effect if it were permitted.

* `T` must be at least as accessible as the generic type or method being declared.

Multiple type constraints can be specified for a single type parameter by enclosing the type constraints in curly braces (`{}`).. Only one type constraint for a given type parameter can be a class. It is an error to combine a `Structure` special constraint with a named class constraint or the `Class` special constraint.

```vb
Class ControlFactory(Of T As {Control, New})
    ...
End Class
```

Type constraints can use the containing types or any of the containing types' type parameters. In the following example, the constraint requires that the type argument supplied implements a generic interface using itself as a type argument:

```vb
Class Sorter(Of V As IComparable(Of V))
    ...
End Class
```

The special type constraint `Class` constrains the supplied type argument to any reference type.

__Note.__ The special type constraint `Class` can be satisfied by an interface. And a structure can implement an interface. Therefore, the constraint `(Of T As U, U As Class)` might be satisfied with "T" a structure (which does not satisfy the `Class` special constraint), and "U" an interface that it implements (which does satisfy the `Class` special constraint).

The special type constraint `Structure` constrains the supplied type argument to any value type except `System.Nullable(Of T)`.

__Note.__ Structure constraints do not allow `System.Nullable(Of T)` so that it is not possible to supply `System.Nullable(Of T)` as a type argument to itself.

The special type constraint `New` requires that the supplied type argument must have an accessible parameterless constructor and cannot be declared `MustInherit`. For example:

```vb
Class Factory(Of T As New)
    Function CreateInstance() As T
        Return New T()
    End Function
End Class
```

A class type constraint requires that the supplied type argument must either be that type as or inherit from it. An interface type constraint requires that the supplied type argument must implement that interface. A type parameter constraint requires that the supplied type argument must derive from or implement all of the bounds given for the matching type parameter. For example:

```vb
Class List(Of T)
    Sub AddRange(Of S As T)(collection As IEnumerable(Of S))
        ...
    End Sub
End Class
```

In this example, the type parameter `S` on `AddRange` is constrained to the type parameter `T` of `List`. This means that a `List(Of Control)` would constrain `AddRange`'s type parameter to any type that is or inherits from `Control`.

A type parameter constraint `Of S As T` is resolved by transitively adding all of `T`'s constraints onto `S`, other than the special constraints (`Class`, `Structure`, `New`). It is an error to have circular constraints (e.g. `Of S As T, T As S`). It is an error to have a type parameter constraint which itself has the `Structure` constraint. After adding constraints, it is possible that a number of special situations may occur:

* If multiple class constraints exist, the most derived class is considered to be the constraint. If one or more class constraints have no inheritance relationship, the constraint is unsatisfiable and it is an error.

 * If a type parameter combines a `Structure` special constraint with a named class constraint or the `Class` special constraint, it is an error. A class constraint may be `NotInheritable`, in which case no derived types of that constraint are accepted and it is an error.

The type may be one of, or a type inherited from, the following special types: `System.Array`, `System.Delegate`, `System.MulticastDelegate`, `System.Enum`, or `System.ValueType`. In that case, only the type, or a type inherited from it, is accepted. A type parameter constrained to one of these types can only use the conversions allowed by the `DirectCast` operator. For example:

```vb
MustInherit Class Base(Of T)
    MustOverride Sub S1(Of U As T)(x As U)
End Class

Class Derived
    Inherits Base(Of Integer)

    ' The constraint of U must be Integer, which is normally not allowed.
    Overrides Sub S1(Of U As Integer)(x As U)
        Dim y As Integer = x    ' OK
        Dim z As Long = x       ' Error: Can't convert
    End Sub
End Class
```

Additionally, a type parameter constrained to a value type due to one of the above relaxations cannot call any methods defined on that value type. For example:

```vb
Class C1(Of T)
    Overridable Sub F(Of G As T)(x As G)
    End Sub
End Class

Class C2
    Inherits C1(Of IntPtr)

    Overrides Sub F(Of G As IntPtr)(ByVal x As G)
        ' Error: Cannot access structure members
         x.ToInt32()
    End Sub
End Class
```

If the constraint, after substitution, ends up as an array type, any covariant array type is allowed as well. For example:

```vb
Module Test
    Class B
    End Class

    Class D
        Inherits B
    End Class

    Function F(Of T, U As T)(x As U) As T
        Return x
    End Function

    Sub Main()
        Dim a(9) As B
        Dim b(9) As D

        a = F(Of B(), D())(b)
    End Sub
End Module
```

A type parameter with a class or interface constraint is considered to have the same members as that class or interface constraint. If a type parameter has multiple constraints, then the type parameter is considered to have the union of all the members of the constraints. If there are members with the same name in more than one constraint, then members are hidden in the following order: the class constraint hides members in interface constraints, which hide members in `System.ValueType` (if `Structure` constraint is specified), which hides members in `Object`. If a member with the same name appears in more than one interface constraint the member is unavailable (as in multiple interface inheritance) and the type parameter must be cast to the desired interface. For example:

```vb
Class C1
    Sub S1(x As Integer)
    End Sub
End Class

Interface I1
    Sub S1(x As Integer)
End Interface

Interface I2
    Sub S1(y As Double)
End Interface

Module Test
    Sub T1(Of T As {C1, I1, I2})()
        Dim a As T
        a.S1(10)       ' Calls C1.S1, which is preferred
        a.S1(10.10)    ' Also calls C1.S1, class is still preferred
    End Sub

    Sub T2(Of T As {I1, I2})()
        Dim a As T
        a.S1(10)    ' Error: Call is ambiguous between I1.S1, I2.S1
    End Sub
End Module
```

When supplying type parameters as type arguments, the type parameters must satisfy the constraints of the matching type parameters.

```vb
Class Base(Of T As Class)
End Class

Class Derived(Of V)
    ' Error: V does not satisfy the constraints of T
    Inherits Base(Of V)
End Class
```

Values of a constrained type parameter can be used to access the instance members, including instance methods, specified in the constraint.

```vb
Interface IPrintable
    Sub Print()
End Interface

Class Printer(Of V As IPrintable)
    Sub PrintOne(v1 As V)
        V1.Print()
    End Sub
End Class
```

### Type Parameter Variance

A type parameter in an interface or a delegate type declaration can optionally specify a *variance modifier*. Type parameters with variance modifiers restrict how the type parameter can be used in the interface or delegate type but allow a generic interface or delegate type to be converted to another generic type with variant compatible type arguments. For example:

```vb
Class Base
End Class

Class Derived
    Inherits Base
End Class

Module Test
    Sub Main()
        Dim x As IEnumerable(Of Derived) = ...

        ' OK, as IEnumerable(Of Base) is variant compatible
        ' with IEnumerable(Of Derived)
        Dim y As IEnumerable(Of Base) = x
    End Sub
End Module
```

Generic interfaces that have type parameters with variance modifiers have several restrictions:

* They cannot contain an event declaration that specifies a parameter list (but a custom event declaration or an event declaration with a delegate type is allowed).

* They cannot contain a nested class, structure, or enumerated type.

__Note.__ These restrictions are due to the fact that types nested in generic types implicitly copy the generic parameters of their parent. In the case of nested classes, structures, or enumerated types, those kinds of types cannot have variance modifiers on their type parameters. In the case of an event declaration with a parameter list, the generated nested delegate class could have confusing errors when a type that appears to be used in an `In` position (i.e. a parameter type) is actually used in an `Out` position (i.e. the type of the event).

A type parameter that is declared with the Out modifier is *covariant*. Informally, a covariant type parameter can only be used in an output position -- i.e. a value that is being returned from the interface or delegate type -- and cannot be used in an input position. A type `T` is considered to be *valid covariantly* if:

* `T` is a class, structure, or enumerated type.

* `T` is non-generic delegate or interface type.

* `T` is an array type whose element type is valid covariantly.

* `T` is a type parameter which was not declared as an `Out` type parameter.

* `T` is a constructed interface or delegate type `X(Of P1,...,Pn)` with type arguments `A1,...,An` such that:

  * If `Pi` was declared as an Out type parameter then `Ai` is valid covariantly.

  * If `Pi` was declared as an In type parameter then `Ai` is valid contravariantly.

The following must be valid covariantly in an interface or delegate type:

* The base interface of an interface.

* The return type of a function or the delegate type.

* The type of a property if there is a `Get` accessor.

* The type of any `ByRef` parameter.

For example:

```vb
Delegate Function D(Of Out T, U)(x As U) As T

Interface I1(Of Out T)
End Interface

Interface I2(Of Out T)
    Inherits I1(Of T)

    ' OK, T is only used in an Out position
    Function M1(x As I1(Of T)) As T

    ' Error: T is used in an In position
    Function M2(x As T) As T
End Interface
```

__Note.__ `Out` is not a reserved word.

A type parameter that is declared with the In modifier is *contravariant*. Informally, a contravariant type parameter can only be used in an input position -- i.e. a value that is being passed in to the interface or delegate type -- and cannot be used in an output position. A type `T` is considered to be *valid contravariantly* if:

* `T` is a class, structure, or enumerated type.

* `T` is a non-generic delegate or interface type.

* `T` is an array type whose element type is valid contravariantly.

* `T` is a type parameter which was not declared as an In type parameter.

* `T` is a constructed interface or delegate type `X(Of P1,...,Pn)` with type arguments `A1,...,An` such that:

  * If `Pi` was declared as an `Out` type parameter then `Ai` is valid contravariantly.

  * If `Pi` was declared as an `In` type parameter then `Ai` is valid covariantly.

The following must be valid contravariantly in an interface or delegate type:

* The type of a parameter.

* A type constraint on a method type parameter.

* The type of a property if it has a `Set` accessor.

* The type of an event.

For example:

```vb
Delegate Function D(Of T, In U)(x As U) As T

Interface I1(Of In T)
End Interface

Interface I2(Of In T)
    ' OK, T is only used in an In position
    Sub M1(x As I1(Of T))

    ' Error: T is used in an Out position
    Function M2() As T
End Interface
```

In the case where a type must be valid be contravariantly and covariantly (such as a property with both a `Get` and `Set` accessor or a `ByRef` parameter), a variant type parameter cannot be used.


Co- and contra-variance give rise to a "diamond ambiguity problem". Consider the following code:

```vb
Class C
    Implements IEnumerable(Of String)
    Implements IEnumerable(Of Exception)
     
    Public Function GetEnumerator1() As IEnumerator(Of String) _
       Implements IEnumerable(Of String).GetEnumerator
       Console.WriteLine("string")
    End Function
     
    Public Function GetEnumerator2() As IEnumerator(Of Exception) _
       Implements IEnumerable(Of Execption).GetEnumerator
       Console.WriteLine("exception")
    End Function
End Class
     
Dim c As IEnumerable(Of Object) = New C
c.GetEnumerator()
```

The class `C` can be converted to `IEnumerable(Of Object)` in two ways, both through covariant conversion from `IEnumerable(Of String)` and through covariant conversion from `IEnumerable(Of Exception)`. The CLR does not specify which of the two methods will be called by `c.GetEnumerator()`. In general, whenever a class is declared to implement a covariant interface with two different generic arguments that have a common supertype (e.g. in this case `String` and `Exception` have the common supertype `Object`), or a class is declared to implement a contravariant interface with two different generic arguments that have a common subtype, then ambiguity is likely to arise. The compiler gives a warning on such declarations.
