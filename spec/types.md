# Types

The two fundamental categories of types in Visual Basic are *value types* and *reference types*. Primitive types (except strings), enumerations, and structures are value types. Classes, strings, standard modules, interfaces, arrays, and delegates are reference types.

Every type has a *default value*, which is the value that is assigned to variables of that type upon initialization.

```antlr
TypeName
    : ArrayTypeName
    | NonArrayTypeName
    ;

NonArrayTypeName
    : SimpleTypeName
    | NullableTypeName
    ;

SimpleTypeName
    : QualifiedTypeName
    | BuiltInTypeName
    ;

QualifiedTypeName
    : Identifier TypeArguments? (Period IdentifierOrKeyword TypeArguments?)*
    | 'Global' Period IdentifierOrKeyword TypeArguments?
      (Period IdentifierOrKeyword TypeArguments?)*
    ;

TypeArguments
    : OpenParenthesis 'Of' TypeArgumentList CloseParenthesis
    ;

TypeArgumentList
    : TypeName ( Comma TypeName )*
    ;

BuiltInTypeName
    : 'Object'
    | PrimitiveTypeName
    ;

TypeModifier
    : AccessModifier
    | 'Shadows'
    ;

IdentifierModifiers
    : NullableNameModifier? ArrayNameModifier?
    ;
```

## Value Types and Reference Types

Although value types and reference types can be similar in terms of declaration syntax and usage, their semantics are distinct.

Reference types are stored on the run-time heap; they may only be accessed through a reference to that storage. Because reference types are always accessed through references, their lifetime is managed by the .NET Framework. Outstanding references to a particular instance are tracked and the instance is destroyed only when no more references remain. A variable of reference type contains a reference to a value of that type, a value of a more derived type, or a null value. A *null value* refers to nothing; it is not possible to do anything with a null value except assign it. Assignment to a variable of a reference type creates a copy of the reference rather than a copy of the referenced value. For a variable of a reference type, the default value is a null value.

Value types are stored directly on the stack, either within an array or within another type; their storage can only be accessed directly. Because value types are stored directly within variables, their lifetime is determined by the lifetime of the variable that contains them. When the location containing a value type instance is destroyed, the value type instance is also destroyed. Value types are always accessed directly; it is not possible to create a reference to a value type. Prohibiting such a reference makes it impossible to refer to a value class instance that has been destroyed. Because value types are always `NotInheritable`, a variable of a value type always contains a value of that type. Because of this, the value of a value type cannot be a null value, nor can it reference an object of a more derived type. Assignment to a variable of a value type creates a copy of the value being assigned. For a variable of a value type, the default value is the result of initializing each variable member of the type to its default value.

The following example shows the difference between reference types and value types:

```vb
Class Class1
    Public Value As Integer = 0
End Class

Module Test
    Sub Main()
        Dim val1 As Integer = 0
        Dim val2 As Integer = val1
        val2 = 123
        Dim ref1 As Class1 = New Class1()
        Dim ref2 As Class1 = ref1
        ref2.Value = 123
        Console.WriteLine("Values: " & val1 & ", " & val2)
        Console.WriteLine("Refs: " & ref1.Value & ", " & ref2.Value)
    End Sub
End Module
```

The output of the program is:

```
Values: 0, 123
Refs: 123, 123
```

The assignment to the local variable `val2` does not impact the local variable `val1` because both local variables are of a value type (the type `Integer`) and each local variable of a value type has its own storage. In contrast, the assignment `ref2.Value = 123;` affects the object that both `ref1` and `ref2` reference.

One thing to note about the .NET Framework type system is that even though structures, enumerations and primitive types (except for `String`) are value types, they all inherit from reference types. Structures and the primitive types inherit from the reference type `System.ValueType`, which inherits from `Object`. Enumerated types inherit from the reference type `System.Enum`, which inherits from `System.ValueType`.

### Nullable Value Types

For value types, a `?` modifier can be added to a type name to represent the *nullable* version of that type.

```antlr
NullableTypeName
    : NonArrayTypeName '?'
    ;

NullableNameModifier
    : '?'
    ;
```

A nullable value type can contain the same values as the non-nullable version of the type as well as the null value. Thus, for a nullable value type, assigning `Nothing` to a variable of the type sets the value of the variable to the null value, not the zero value of the value type. For example:

```vb
Dim x As Integer = Nothing
Dim y As Integer? = Nothing

' Prints zero
Console.WriteLine(x)
' Prints nothing (because the value of y is the null value)
Console.WriteLine(y)
```

A variable may also be declared to be of a nullable value type by putting a nullable type modifier on the variable name. For clarity, it is not valid to have a nullable type modifier on both a variable name and a type name in the same declaration. Since nullable types are implemented using the type `System.Nullable(Of T)`, the type `T?` is synonymous to the type `System.Nullable(Of T)`, and the two names can be used interchangeably. The `?` modifier cannot be placed on a type that is already nullable; thus, it is not possible to declare the type `Integer??` or `System.Nullable(Of Integer)?`.

A nullable value type `T?` has the members of `System.Nullable(Of T)` as well as any operators or conversions *lifted* from the underlying type `T` into the type `T?`. Lifting copies operators and conversions from the underlying type, in most cases substituting nullable value types for non-nullable value types. This allows many of the same conversions and operations that apply to `T` to apply to `T?` as well.


## Interface Implementation

Structure and class declarations may declare that they implement a set of interface types through one or more `Implements` clauses.

```antlr
TypeImplementsClause
    : 'Implements' TypeImplements StatementTerminator
    ;

TypeImplements
    : NonArrayTypeName ( Comma NonArrayTypeName )*
    ;
```

All the types specified in the `Implements` clause must be interfaces, and the type must implement all members of the interfaces. For example:

```vb
Interface ICloneable
    Function Clone() As Object
End Interface 

Interface IComparable
    Function CompareTo(other As Object) As Integer
End Interface 

Structure ListEntry
    Implements ICloneable, IComparable

    ...

    Public Function Clone() As Object Implements ICloneable.Clone
        ...
    End Function 

    Public Function CompareTo(other As Object) As Integer _
        Implements IComparable.CompareTo
        ...
    End Function 
End Structure
```

A type that implements an interface also implicitly implements all of the interface's base interfaces. This is true even if the type does not explicitly list all base interfaces in the `Implements` clause. In this example, the `TextBox` structure implements both `IControl` and `ITextBox`.

```vb
Interface IControl
    Sub Paint()
End Interface 

Interface ITextBox
    Inherits IControl

    Sub SetText(text As String)
End Interface 

Structure TextBox
    Implements ITextBox

    ...

    Public Sub Paint() Implements ITextBox.Paint
        ...
    End Sub

    Public Sub SetText(text As String) Implements ITextBox.SetText
        ...
    End Sub
End Structure
```

Declaring that a type implements an interface in and of itself does not declare anything in the declaration space of the type. Thus, it is valid to implement two interfaces with a method by the same name.

Types cannot implement a type parameter on its own, although it may involve the type parameters that are in scope.

```vb
Class C1(Of V)
    Implements V  ' Error, can't implement type parameter directly
    Implements IEnumerable(Of V)  ' OK, not directly implementing

    ...
End Class
```

Generic interfaces can be implemented multiple times using different type arguments. However, a generic type cannot implement a generic interface using a type parameter if the supplied type parameter (regardless of type constraints) could overlap with another implementation of that interface. For example:

```vb
Interface I1(Of T)
End Interface

Class C1
    Implements I1(Of Integer)
    Implements I1(Of Double)    ' OK, no overlap
End Class

Class C2(Of T)
    Implements I1(Of Integer)
    Implements I1(Of T)         ' Error, T could be Integer
End Class
```


## Primitive Types

The *primitive types* are identified through keywords, which are aliases for predefined types in the `System` namespace. A primitive type is completely indistinguishable from the type it aliases: writing the reserved word `Byte` is exactly the same as writing `System.Byte`. Primitive types are also known as *intrinsic types*.

```antlr
PrimitiveTypeName
    : NumericTypeName
    | 'Boolean'
    | 'Date'
    | 'Char'
    | 'String'
    ;

NumericTypeName
    : IntegralTypeName
    | FloatingPointTypeName
    | 'Decimal'
    ;

IntegralTypeName
    : 'Byte' | 'SByte' | 'UShort' | 'Short' | 'UInteger'
    | 'Integer' | 'ULong' | 'Long'
    ;

FloatingPointTypeName
    : 'Single' | 'Double'
    ;
```


Because a primitive type aliases a regular type, every primitive type has members. For example, `Integer` has the members declared in `System.Int32`. Literals can be treated as instances of their corresponding types.

* The primitive types differ from other structure types in that they permit certain additional operations:

* Primitive types permit values to be created by writing literals. For example, `123I` is a literal of type `Integer`.

* It is possible to declare constants of the primitive types.

* When the operands of an expression are all primitive type constants, it is possible for the compiler to evaluate the expression at compile time. Such an expression is known as a constant expression.

Visual Basic defines the following primitive types:

* The integral value types `Byte` (1-byte unsigned integer), `SByte` (1-byte signed integer), `UShort` (2-byte unsigned integer), `Short` (2-byte signed integer), `UInteger` (4-byte unsigned integer), `Integer` (4-byte signed integer), `ULong` (8-byte unsigned integer), and `Long` (8-byte signed integer). These types map to `System.Byte`, `System.SByte`, `System.UInt16`, `System.Int16`, `System.UInt32`, `System.Int32`, `System.UInt64` and `System.Int64`, respectively. The default value of an integral type is equivalent to the literal `0`.

* The floating-point value types `Single` (4-byte floating point) and `Double` (8-byte floating point). These types map to `System.Single` and `System.Double`, respectively. The default value of a floating-point type is equivalent to the literal `0`.

* The `Decimal` type (16-byte decimal value), which maps to `System.Decimal`. The default value of decimal is equivalent to the literal `0D`.

* The `Boolean` value type, which represents a truth value, typically the result of a relational or logical operation. The literal is of type `System.Boolean`. The default value of the `Boolean` type is equivalent to the literal `False`.

* The `Date` value type, which represents a date and/or a time and maps to `System.DateTime`. The default value of the `Date` type is equivalent to the literal `# 01/01/0001 12:00:00AM #`.

* The `Char` value type, which represents a single Unicode character and maps to `System.Char`. The default value of the `Char` type is equivalent to the constant expression `ChrW(0)`.

* The `String` reference type, which represents a sequence of Unicode characters and maps to `System.String`. The default value of the `String` type is a null value.



## Enumerations

*Enumerations* are value types that inherit from `System.Enum` and symbolically represent a set of values of one of the primitive integral types.

```antlr
EnumDeclaration
    : Attributes? TypeModifier* 'Enum' Identifier
      ( 'As' NonArrayTypeName )? StatementTerminator
      EnumMemberDeclaration+
      'End' 'Enum' StatementTerminator
    ;
```

For an enumeration type `E`, the default value is the value produced by the expression `CType(0, E)`.

The underlying type of an enumeration must be an integral type that can represent all the enumerator values defined in the enumeration. If an underlying type is specified, it must be `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, or one of their corresponding types in the `System` namespace. If no underlying type is explicitly specified, the default is `Integer`.

The following example declares an enumeration with an underlying type of `Long`:

```vb
Enum Color As Long
    Red
    Green
    Blue
End Enum
```

A developer might choose to use an underlying type of `Long`, as in the example, to enable the use of values that are in the range of `Long`, but not in the range of `Integer`, or to preserve this option for the future.


### Enumeration Members

The members of an enumeration are the enumerated values declared in the enumeration and the members inherited from class `System.Enum`.

The scope of an enumeration member is the enumeration declaration body. This means that outside of an enumeration declaration, an enumeration member must always be qualified (unless the type is specifically imported into a namespace through a namespace import).

Declaration order for enumeration member declarations is significant when constant expression values are omitted. Enumeration members implicitly have `Public` access only; no access modifiers are allowed on enumeration member declarations.

```antlr
EnumMemberDeclaration
    : Attributes? Identifier ( Equals ConstantExpression )? StatementTerminator
    ;
```

### Enumeration Values

The enumerated values in an enumeration member list are declared as constants typed as the underlying enumeration type, and they can appear wherever constants are required. An enumeration member definition with `=` gives the associated member the value indicated by the constant expression. The constant expression must evaluate to an integral type that is implicitly convertible to the underlying type and must be within the range of values that can be represented by the underlying type. The following example is in error because the constant values `1.5`, `2.3`, and `3.3` are not implicitly convertible to the underlying integral type `Long` with strict semantics.

```vb
Option Strict On

Enum Color As Long
    Red = 1.5
    Green = 2.3
    Blue = 3.3
End Enum
```

Multiple enumeration members may share the same associated value, as shown below:

```vb
Enum Color
    Red
    Green
    Blue
    Max = Blue
End Enum
```

The example shows an enumeration that has two enumeration members -- `Blue` and `Max` -- that have the same associated value.

If the first enumerator value definition in the enumeration has no initializer, the value of the corresponding constant is `0`. An enumeration value definition without an initializer gives the enumerator the value obtained by increasing the value of the previous enumeration value by `1`. This increased value must be within the range of values that can be represented by the underlying type.

```vb
Enum Color
    Red
    Green = 10
    Blue
End Enum 

Module Test
    Sub Main()
        Console.WriteLine(StringFromColor(Color.Red))
        Console.WriteLine(StringFromColor(Color.Green))
        Console.WriteLine(StringFromColor(Color.Blue))
    End Sub

    Function StringFromColor(c As Color) As String
        Select Case c
            Case Color.Red
                Return String.Format("Red = " & CInt(c))

            Case Color.Green
                Return String.Format("Green = " & CInt(c))

            Case Color.Blue
                Return String.Format("Blue = " & CInt(c))

            Case Else
                Return "Invalid color"
        End Select
    End Function
End Module
```

The example above prints the enumeration values and their associated values. The output is:

```
Red = 0
Green = 10
Blue = 11
```

The reasons for the values are as follows:

* The enumeration value `Red` is automatically assigned the value `0` (since it has no initializer and is the first enumeration value member).

* The enumeration value `Green` is explicitly given the value `10`.

* The enumeration value `Blue` is automatically assigned the value one greater than the enumeration value that textually precedes it.

The constant expression may not directly or indirectly use the value of its own associated enumeration value (that is, circularity in the constant expression is not allowed). The following example is invalid because the declarations of `A` and `B` are circular.

```vb
Enum Circular
    A = B
    B
End Enum
```

`A` depends on `B` explicitly, and `B` depends on `A` implicitly.

## Classes

A *class* is a data structure that may contain data members (constants, variables, and events), function members (methods, properties, indexers, operators, and constructors), and nested types. Classes are reference types.

```antlr
ClassDeclaration
    : Attributes? ClassModifier* 'Class' Identifier TypeParameterList? StatementTerminator
      ClassBase?
      TypeImplementsClause*
      ClassMemberDeclaration*
      'End' 'Class' StatementTerminator
    ;

ClassModifier
    : TypeModifier
    | 'MustInherit'
    | 'NotInheritable'
    | 'Partial'
    ;
```

The following example shows a class that contains each kind of member:

```vb
Class AClass
    Public Sub New()
        Console.WriteLine("Constructor")
    End Sub

    Public Sub New(value As Integer)
        MyVariable = value
        Console.WriteLine("Constructor")
    End Sub

    Public Const MyConst As Integer = 12
    Public MyVariable As Integer = 34

    Public Sub MyMethod()
        Console.WriteLine("MyClass.MyMethod")
    End Sub

    Public Property MyProperty() As Integer
        Get
            Return MyVariable
        End Get

        Set (value As Integer)
            MyVariable = value
        End Set
    End Property

    Default Public Property Item(index As Integer) As Integer
        Get
            Return 0
        End Get

        Set (value As Integer)
            Console.WriteLine("Item(" & index & ") = " & value)
        End Set
    End Property

    Public Event MyEvent()

    Friend Class MyNestedClass
    End Class 
End Class
```

The following example shows uses of these members:

```vb
Module Test

    ' Event usage.
    Dim WithEvents aInstance As AClass

    Sub Main()
        ' Constructor usage.
        Dim a As AClass = New AClass()
        Dim b As AClass = New AClass(123)

        ' Constant usage.
        Console.WriteLine("MyConst = " & AClass.MyConst)

        ' Variable usage.
        a.MyVariable += 1
        Console.WriteLine("a.MyVariable = " & a.MyVariable)

        ' Method usage.
        a.MyMethod()

        ' Property usage.
        a.MyProperty += 1
        Console.WriteLine("a.MyProperty = " & a.MyProperty)
        a(1) = 1

        ' Event usage.
        aInstance = a
    End Sub 

    Sub MyHandler() Handles aInstance.MyEvent
        Console.WriteLine("Test.MyHandler")
    End Sub 
End Module
```

There are two class-specific modifiers, `MustInherit` and `NotInheritable`. It is invalid to specify them both.


### Class Base Specification

A class declaration may include a base type specification that defines the direct base type of the class.

```antlr
ClassBase
    : 'Inherits' NonArrayTypeName StatementTerminator
    ;
```

If a class declaration has no explicit base type, the direct base type is implicitly `Object`. For example:

```vb
Class Base
End Class

Class Derived
    Inherits Base
End Class
```

Base types cannot be a type parameter on its own, although it may involve the type parameters that are in scope.

```vb
Class C1(Of V) 
End Class

Class C2(Of V)
    Inherits V    ' Error, type parameter used as base class 
End Class

Class C3(Of V)
    Inherits C1(Of V)    ' OK: not directly inheriting from V.
End Class
```

Classes may only derive from `Object` and classes. It is invalid for a class to derive from `System.ValueType`, `System.Enum`, `System.Array`, `System.MulticastDelegate` or `System.Delegate`. A generic class cannot derive from `System.Attribute` or from a class that derives from it.

Every class has exactly one direct base class, and circularity in derivation is prohibited. It is not possible to derive from a `NotInheritable` class, and the accessibility domain of the base class must be the same as or a superset of the accessibility domain of the class itself.


### Class Members

The members of a class consist of the members introduced by its class member declarations and the members inherited from its direct base class.

```antlr
ClassMemberDeclaration
    : NonModuleDeclaration
    | EventMemberDeclaration
    | VariableMemberDeclaration
    | ConstantMemberDeclaration
    | MethodMemberDeclaration
    | PropertyMemberDeclaration
    | ConstructorMemberDeclaration
    | OperatorDeclaration
    ;
```

A class member declaration may have `Public`, `Protected`, `Friend`, `Protected Friend`, or `Private` access. When a class member declaration does not include an access modifier, the declaration defaults to `Public` access, unless it is a variable declaration; in that case it defaults to `Private` access.

The scope of a class member is the class body in which the member declaration occurs, plus the constraint list of that class (if it is generic and has constraints). If the member has `Friend` access, its scope extends to the class body of any derived class in the same program or any assembly that has been given `Friend` access, and if the member has `Public`, `Protected`, or `Protected Friend` access, its scope extends to the class body of any derived class in any program.


## Structures

*Structures* are value types that inherit from `System.ValueType`. Structures are similar to classes in that they represent data structures that can contain data members and function members. Unlike classes, however, structures do not require heap allocation.

```antlr
StructureDeclaration
    : Attributes? StructureModifier* 'Structure' Identifier
      TypeParameterList? StatementTerminator
      TypeImplementsClause*
      StructMemberDeclaration*
      'End' 'Structure' StatementTerminator
    ;

StructureModifier
    : TypeModifier
    | 'Partial'
    ;
```

In the case of classes, it is possible for two variables to reference the same object, and thus possible for operations on one variable to affect the object referenced by the other variable. With structures, the variables each have their own copy of the non-`Shared` data, so it is not possible for operations on one to affect the other, as the following example illustrates:

```vb
Structure Point
    Public x, y As Integer

    Public Sub New(x As Integer, y As Integer)
        Me.x = x
        Me.y = y
    End Sub
End Structure
```

Given the above declaration the following code outputs the value `10`:

```vb
Module Test
    Sub Main()
        Dim a As New Point(10, 10)
        Dim b As Point = a

        a.x = 100
        Console.WriteLine(b.x)
    End Sub
End Module
```

The assignment of `a` to `b` creates a copy of the value, and `b` is thus unaffected by the assignment to `a.x`. Had `Point` instead been declared as a class, the output would be `100` because `a` and `b` would reference the same object.


### Structure Members

The members of a structure are the members introduced by its structure member declarations and the members inherited from `System.ValueType`.

```antlr
StructMemberDeclaration
    : NonModuleDeclaration
    | VariableMemberDeclaration
    | ConstantMemberDeclaration
    | EventMemberDeclaration
    | MethodMemberDeclaration
    | PropertyMemberDeclaration
    | ConstructorMemberDeclaration
    | OperatorDeclaration
    ;
```

Every structure implicitly has a `Public` parameterless instance constructor that produces the default value of the structure. As a result, it is not possible for a structure type declaration to declare a parameterless instance constructor. A structure type is, however, permitted to declare *parameterized* instance constructors, as in the following example:

```vb
Structure Point
    Private x, y As Integer

    Public Sub New(x As Integer, y As Integer)
        Me.x = x
        Me.y = y
    End Sub
End Structure
```

Given the above declaration, the following statements both create a `Point` with `x` and `y` initialized to zero.

```vb
Dim p1 As Point = New Point()
Dim p2 As Point = New Point(0, 0)
```

Because structures directly contain their field values (rather than references to those values), structures cannot contain fields that directly or indirectly reference themselves. For example, the following code is not valid:

```vb
Structure S1
    Dim f1 As S2
End Structure

Structure S2
    ' This would require S1 to contain itself.
    Dim f1 As S1
End Structure
```

Normally, a structure member declaration may only have `Public`, `Friend`, or `Private` access, but when overriding members inherited from `Object`, `Protected` and `Protected Friend` access may also be used. When a structure member declaration does not include an access modifier, the declaration defaults to `Public` access. The scope of a member declared by a structure is the structure body in which the declaration occurs, plus the constraints of that structure (if it was generic and had constraints).


## Standard Modules

A *standard module* is a type whose members are implicitly `Shared` and scoped to the declaration space of the standard module's containing namespace, rather than just to the standard module declaration itself. Standard modules may never be instantiated. It is an error to declare a variable of a standard module type.

```antlr
ModuleDeclaration
    : Attributes? TypeModifier* 'Module' Identifier StatementTerminator
      ModuleMemberDeclaration*
      'End' 'Module' StatementTerminator
    ;
```

A member of a standard module has two fully qualified names, one without the standard module name and one with the standard module name. More than one standard module in a namespace may define a member with a particular name; unqualified references to the name outside of either module are ambiguous. For example:

```vb
Namespace N1
    Module M1
        Sub S1()
        End Sub

        Sub S2()
        End Sub
    End Module

    Module M2
        Sub S2()
        End Sub
    End Module

    Module M3
        Sub Main()
            S1()       ' Valid: Calls N1.M1.S1.
            N1.S1()    ' Valid: Calls N1.M1.S1.
            S2()       ' Not valid: ambiguous.
            N1.S2()    ' Not valid: ambiguous.
            N1.M2.S2() ' Valid: Calls N1.M2.S2.
        End Sub
    End Module
End Namespace
```

A module may only be declared in a namespace and may not be nested in another type. Standard modules may not implement interfaces, they implicitly derive from `Object`, and they have only `Shared` constructors.


### Standard Module Members

The members of a standard module are the members introduced by its member declarations and the members inherited from `Object`. Standard modules may have any type of member except instance constructors. All standard module type members are implicitly `Shared`.

```antlr
ModuleMemberDeclaration
    : NonModuleDeclaration
    | VariableMemberDeclaration
    | ConstantMemberDeclaration
    | EventMemberDeclaration
    | MethodMemberDeclaration
    | PropertyMemberDeclaration
    | ConstructorMemberDeclaration
    ;
```

Normally, a standard module member declaration may only have `Public`, `Friend`, or `Private` access, but when overriding members inherited from `Object`, the `Protected` and `Protected Friend` access modifiers may be specified. When a standard module member declaration does not include an access modifier, the declaration defaults to `Public` access, unless it is a variable, which defaults to `Private` access.

As previously noted, the scope of a standard module member is the declaration containing the standard module declaration. Members inherited from `Object` are not included in this special scoping; those members have no scope and must always be qualified with the name of the module. If the member has `Friend` access, its scope extends only to namespace members declared in the same program or assemblies that have been given `Friend` access.


## Interfaces

*Interfaces* are reference types that other types implement to guarantee that they support certain methods. An interface is never directly created and has no actual representation -- other types must be converted to an interface type. An interface defines a contract. A class or structure that implements an interface must adhere to its contract.

```antlr
InterfaceDeclaration
    : Attributes? TypeModifier* 'Interface' Identifier
      TypeParameterList? StatementTerminator
      InterfaceBase*
      InterfaceMemberDeclaration*
      'End' 'Interface' StatementTerminator
    ;
```


The following example shows an interface that contains a default property `Item`, an event `E`, a method `F`, and a property `P`:

```vb
Interface IExample
    Default Property Item(index As Integer) As String

    Event E()

    Sub F(value As Integer)

    Property P() As String
End Interface
```

Interfaces may employ multiple inheritance. In the following example, the interface `IComboBox` inherits from both `ITextBox` and `IListBox`:

```vb
Interface IControl
    Sub Paint()
End Interface 

Interface ITextBox
    Inherits IControl

    Sub SetText(text As String)
End Interface 

Interface IListBox
    Inherits IControl

    Sub SetItems(items() As String)
End Interface 

Interface IComboBox
    Inherits ITextBox, IListBox 
End Interface
```

Classes and structures can implement multiple interfaces. In the following example, the class `EditBox` derives from the class `Control` and implements both `IControl` and `IDataBound`:

```vb
Interface IDataBound
    Sub Bind(b As Binder)
End Interface 

Public Class EditBox
    Inherits Control
    Implements IControl, IDataBound

    Public Sub Paint() Implements IControl.Paint
        ...
    End Sub

    Public Sub Bind(b As Binder) Implements IDataBound.Bind
        ...
    End Sub
End Class
```


### Interface Inheritance

The base interfaces of an interface are the explicit base interfaces and their base interfaces. In other words, the set of base interfaces is the complete transitive closure of the explicit base interfaces, their explicit base interfaces, and so on. If an interface declaration has no explicit interface base, then there is no base interface for the type -- interfaces do not inherit from `Object` (although they do have a widening conversion to `Object`).

```antlr
InterfaceBase
    : 'Inherits' InterfaceBases StatementTerminator
    ;

InterfaceBases
    : NonArrayTypeName ( Comma NonArrayTypeName )*
    ;
```

In the following example, the base interfaces of `IComboBox` are `IControl`, `ITextBox`, and `IListBox`.

```vb
Interface IControl
    Sub Paint()
End Interface 

Interface ITextBox
    Inherits IControl

    Sub SetText(text As String)
End Interface 

Interface IListBox
    Inherits IControl

    Sub SetItems(items() As String)
End Interface 

Interface IComboBox
    Inherits ITextBox, IListBox 
End Interface
```

An interface inherits all members of its base interfaces. In other words, the `IComboBox` interface above inherits members `SetText` and `SetItems` as well as `Paint`.

A class or structure that implements an interface also implicitly implements all of the interface's base interfaces.

If an interface appears more than once in the transitive closure of the base interfaces, it only contributes its members to the derived interface once. A type implementing the derived interface only has to implement the methods of the multiply defined base interface once. In the following example, `Paint` only needs to be implemented once, even though the class implements `IComboBox` and `IControl`.

```vb
Class ComboBox
    Implements IControl, IComboBox

    Sub SetText(text As String) Implements IComboBox.SetText
    End Sub

    Sub SetItems(items() As String) Implements IComboBox.SetItems
    End Sub

    Sub Print() Implements IComboBox.Paint
    End Sub
End Class
```

An `Inherits` clause has no effect on other `Inherits` clauses. In the following example, `IDerived` must qualify the name of `INested` with `IBase`.

```vb
Interface IBase
    Interface INested
        Sub Nested()
    End Interface

    Sub Base()
End Interface

Interface IDerived
    Inherits IBase, INested   ' Error: Must specify IBase.INested.
End Interface
```

The accessibility domain of a base interface must be the same as or a superset of the accessibility domain of the interface itself.


### Interface Members

The members of an interface consist of the members introduced by its member declarations and the members inherited from its base interfaces.

```antlr
InterfaceMemberDeclaration
    : NonModuleDeclaration
    | InterfaceEventMemberDeclaration
    | InterfaceMethodMemberDeclaration
    | InterfacePropertyMemberDeclaration
    ;
```

Although interfaces do not inherit members from `Object`, because every class or structure that implements an interface does inherit from `Object`, the members of `Object`, including extension methods, are considered members of an interface and can be called on an interface directly without requiring a cast to `Object`. For example:

```vb
Interface I1
End Interface

Class C1
    Implements I1
End Class

Module Test
    Sub Main()
        Dim i As I1 = New C1()
        Dim h As Integer = i.GetHashCode()
    End Sub
End Module
```

Members of an interface with the same name as members of `Object` implicitly shadow `Object` members. Only nested types, methods, properties, and events may be members of an interface. Methods and properties may not have a body. Interface members are implicitly `Public` and may not specify an access modifier. The scope of a member declared in an interface is the interface body in which the declaration occurs, plus the constraint list of that interface (if it is generic and has constraints).


## Arrays

An *array* is a reference type that contains variables accessed through *indices* corresponding in a one-to-one fashion with the order of the variables in the array. The variables contained in an array, also called the *elements* of the array, must all be of the same type, and this type is called the *element type* of the array.

```antlr
ArrayTypeName
    : NonArrayTypeName ArrayTypeModifiers
    ;

ArrayTypeModifiers
    : ArrayTypeModifier+
    ;

ArrayTypeModifier
    : OpenParenthesis RankList? CloseParenthesis
    ;

RankList
    : Comma*
    ;

ArrayNameModifier
    : ArrayTypeModifiers
    | ArraySizeInitializationModifier
    ;
```

The elements of an array come into existence when an array instance is created, and cease to exist when the array instance is destroyed. Each element of an array is initialized to the default value of its type. The type `System.Array` is the base type of all array types and may not be instantiated. Every array type inherits the members declared by the `System.Array` type and is convertible to it (and `Object`). A one-dimensional array type with element `T` also implements the interfaces `System.Collections.Generic.IList(Of T)` and `IReadOnlyList(Of T)`; if `T` is a reference type, then the array type also implements `IList(Of U)` and `IReadOnlyList(Of U)` for any `U` that has a widening  reference conversion from `T`.

An array has a *rank* that determines the number of indices associated with each array element. The rank of an array determines the number of *dimensions* of the array. For example, an array with a rank of one is called a single-dimensional array, and an array with a rank greater than one is called a multidimensional array.

The following example creates a single-dimensional array of integer values, initializes the array elements, and then prints each of them out:

```vb
Module Test
    Sub Main()
        Dim arr(5) As Integer
        Dim i As Integer

        For i = 0 To arr.Length - 1
            arr(i) = i * i
        Next i

        For i = 0 To arr.Length - 1
            Console.WriteLine("arr(" & i & ") = " & arr(i))
        Next i
    End Sub
End Module
```

The program outputs the following:

```
arr(0) = 0
arr(1) = 1
arr(2) = 4
arr(3) = 9
arr(4) = 16
arr(5) = 25
```

Each dimension of an array has an associated length. Dimension lengths are not part of the type of the array, but rather are established when an instance of the array type is created at run time. The length of a dimension determines the valid range of indices for that dimension: for a dimension of length `N`, indices can range from zero to `N-1`. If a dimension is of length zero, there are no valid indices for that dimension. The total number of elements in an array is the product of the lengths of each dimension in the array. If any of the dimensions of an array has a length of zero, the array is said to be empty. The element type of an array can be any type.

Array types are specified by adding a modifier to an existing type name. The modifier consists of a left parenthesis, a set of zero or more commas, and a right parenthesis. The type modified is the element type of the array, and the number of dimensions is the number of commas plus one. If more than one modifier is specified, then the element type of the array is an array. The modifiers are read left to right, with the leftmost modifier being the outermost array. In the example

```vb
Module Test
    Dim arr As Integer(,)(,,)()
End Module
```

the element type of `arr` is a two-dimensional array of three-dimensional arrays of one-dimensional arrays of `Integer`.

A variable may also be declared to be of an array type by putting an array type modifier or an array-size initialization modifier on the variable name. In that case, the array element type is the type given in the declaration, and the array dimensions are determined by the variable name modifier. For clarity, it is not valid to have an array type modifier on both a variable name and a type name in the same declaration.

The following example shows a variety of local variable declarations that use array types with `Integer` as the element type:

```vb
Module Test
    Sub Main()
        Dim a1() As Integer    ' Declares 1-dimensional array of integers.
        Dim a2(,) As Integer   ' Declares 2-dimensional array of integers.
        Dim a3(,,) As Integer  ' Declares 3-dimensional array of integers.

        Dim a4 As Integer()    ' Declares 1-dimensional array of integers.
        Dim a5 As Integer(,)   ' Declares 2-dimensional array of integers.
        Dim a6 As Integer(,,)  ' Declares 3-dimensional array of integers.

        ' Declare 1-dimensional array of 2-dimensional arrays of integers 
        Dim a7()(,) As Integer
        ' Declare 2-dimensional array of 1-dimensional arrays of integers.
        Dim a8(,)() As Integer

        Dim a9() As Integer() ' Not allowed.
    End Sub
End Module
```

An array type name modifier extends to all sets of parentheses that follow it. This means that in the situations where a set of arguments enclosed in parenthesis is allowed after a type name, it is not possible to specify the arguments for an array type name. For example:

```vb
Module Test
    Sub Main()
        ' This calls the Integer constructor.
        Dim x As New Integer(3)

        ' This declares a variable of Integer().
        Dim y As Integer()

        ' This gives an error.
        ' Array sizes can not be specified in a type name.
        Dim z As Integer()(3)
    End Sub
End Module
```

In the last case, `(3)` is interpreted as part of the type name rather than as a set of constructor arguments.


## Delegates

A *delegate* is a reference type that refers to a `Shared` method of a type or to an instance method of an object.

```antlr
DelegateDeclaration
    : Attributes? TypeModifier* 'Delegate' MethodSignature StatementTerminator
    ;

MethodSignature
    : SubSignature
    | FunctionSignature
    ;
```

 The closest equivalent of a delegate in other languages is a function pointer, but whereas a function pointer can only reference `Shared` functions, a delegate can reference both `Shared` and instance methods. In the latter case, the delegate stores not only a reference to the method's entry point, but also a reference to the object instance with which to invoke the method.

The delegate declaration may not have  a `Handles` clause, an `Implements` clause, a method body, or an `End` construct. The parameter list of the delegate declaration may not contain `Optional` or `ParamArray` parameters. The accessibility domain of the return type and parameter types must be the same as or a superset of the accessibility domain of the delegate itself.

The members of a delegate are the members inherited from class `System.Delegate`. A delegate also defines the following methods:

* A constructor that takes two parameters, one of type `Object` and one of type `System.IntPtr`.

* An `Invoke` method that has the same signature as the delegate.

* A `BeginInvoke` method whose signature is the delegate signature, with three differences. First, the return type is changed to `System.IAsyncResult`. Second, two additional parameters are added to the end of the parameter list: the first of type `System.AsyncCallback` and the second of type `Object`. And finally, all `ByRef` parameters are changed to be `ByVal`.

* An `EndInvoke` method whose return type is the same as the delegate. The parameters of the method are only the delegate parameters exactly that are `ByRef` parameters, in the same order they occur in the delegate signature.  In addition to those parameters, there is an additional parameter of type `System.IAsyncResult` at the end of the parameter list.

There are three steps in defining and using delegates: declaration, instantiation, and invocation.

Delegates are declared using delegate declaration syntax. The following example declares a delegate named `SimpleDelegate` that takes no arguments:

```vb
Delegate Sub SimpleDelegate()
```

The next example creates a `SimpleDelegate` instance and then immediately calls it:

```vb
Module Test
    Sub F()
        System.Console.WriteLine("Test.F")
    End Sub 

    Sub Main()
        Dim d As SimpleDelegate = AddressOf F
        d()
    End Sub 
End Module
```

There is not much point in instantiating a delegate for a method and then immediately calling via the delegate, as it would be simpler to call the method directly. Delegates show their usefulness when their anonymity is used. The next example shows a `MultiCall` method that repeatedly calls a `SimpleDelegate` instance:

```vb
Sub MultiCall(d As SimpleDelegate, count As Integer)
    Dim i As Integer

    For i = 0 To count - 1
        d()
    Next i
End Sub
```

It is unimportant to the `MultiCall` method what the target method for the `SimpleDelegate` is, what accessibility this method has, or whether the method is `Shared` or not. All that matters is that the signature of the target method is compatible with `SimpleDelegate`.


## Partial types

Class and structure declarations can be *partial* declarations. A partial declaration may or may not fully describe the declared type within the declaration. Instead, the declaration of the type may be spread across multiple partial declarations within the program; partial types cannot be declared across program boundaries. A partial type declaration specifies the `Partial` modifier on the declaration. Then, any other declarations in the program for a type with the same fully-qualified name will be merged together with the partial declaration at compile-time to form a single type declaration. For example, the following code declares a single class `Test` with members `Test.C1` and `Test.C2`.

a.vb:

```vb
Public Partial Class Test
    Public Sub S1()
    End Sub
End Class
```

b.vb:

```vb
Public Class Test
    Public Sub S2()
    End Sub
End Class
```

When combining partial type declarations, at least one of the declarations must have a `Partial` modifier, otherwise a compile-time error results.

__Note.__ Although it is possible to specify `Partial` on only one declaration among many partial declarations, it is better form to specify it on all partial declarations. In the situation where one partial declaration is visible but one or more partial declarations are hidden (such as the case of extending tool-generated code), it is acceptable to leave the `Partial` modifier off of the visible declaration but specify it on the hidden declarations.

Only classes and structures can be declared using partial declarations. The arity of a type is considered when matching partial declarations together: two classes with the same name but different numbers of type parameters are not considered to be partial declarations of the same time. Partial declarations can specify attributes, class modifiers, `Inherits` statement or `Implements` statement. At compile time, all of the pieces of the partial declarations are combined together and used as a part of the type declaration. If there are any conflicts between attributes, modifiers, bases, interfaces, or type members, a compile-time error results. For example:

```vb
Public Partial Class Test1
    Implements IDisposable
End Class

Class Test1
    Inherits Object
    Implements IComparable
End Class

Public Partial Class Test2
End Class

Private Partial Class Test2
End Class
```

The previous example declares a type `Test1` that is `Public`, inherits from `Object` and implements `System.IDisposable` and `System.IComparable`. The partial declarations of `Test2` will cause a compile-time error because one of the declarations says that `Test2` is `Public` and another says that `Test2` is `Private`.

Partial types with type parameters can declare constraints and variance for the type parameters, but the constraints and variance from each partial declaration must match. Thus, constraints and variance are special in that they are not automatically combined like other modifiers:

```vb
Partial Public Class List(Of T As IEnumerable)
End Class

' Error: Constraints on T don't match
Class List(Of T As IComparable)
End Class
```

The fact that a type is declared using multiple partial declarations does not affect the name lookup rules within the type. As a result, a partial type declaration can use members declared in other partial type declarations, or may implement methods on interfaces declared in other partial type declarations. For example:

```vb
Public Partial Class Test1
    Implements IDisposable

    Private IsDisposed As Boolean = False
End Class

Class Test1
    Private Sub Dispose() Implements IDisposable.Dispose
        If Not IsDisposed Then
            ...
        End If
    End Sub
End Class
```

Nested types can have partial declarations as well. For example:

```vb
Public Partial Class Test
    Public Partial Class NestedTest
        Public Sub S1()
        End Sub
    End Class
End Class

Public Partial Class Test
    Public Partial Class NestedTest
        Public Sub S2()
        End Sub
    End Class
End Class
```

Initializers within a partial declaration will still be executed in declaration order; however, there is no guaranteed order of execution for initializers that occur in separate partial declarations.

## Constructed Types

A generic type declaration, by itself, does not denote a type. Instead, a generic type declaration can be used as a "blueprint" to form many different types by applying type arguments. A generic type that has type arguments applied to it is called a *constructed type*. The type arguments in a constructed type must always satisfy the constraints placed on the type parameters they match to.

A type name might identify a constructed type even though it doesn't specify type parameters directly. This can occur where a type is nested within a generic class declaration, and the instance type of the containing declaration is implicitly used for name lookup:

```vb
Class Outer(Of T) 
    Public Class Inner 
    End Class

    ' Type of i is the constructed type Outer(Of T).Inner
    Public i As Inner 
End Class
```

A constructed type `C(Of T1,...,Tn)` is accessible when the generic type and all the type arguments are accessible. For instance, if the generic type `C` is `Public` and all of the type arguments `T1,...,Tn` are `Public`, then the constructed type is `Public`. If either the type name or one of the type arguments is `Private`, however, then the accessibility of the constructed type is `Private`. If one type argument of the constructed type is `Protected` and another type argument is `Friend`, then the constructed type is accessible only in the class and its subclasses in this assembly or any assembly that has been given `Friend` access. In other words, the accessibility domain for a constructed type is the intersection of the accessibility domains of its constituent parts.

__Note.__ The fact that the accessibility domain of constructed type is the intersection of its constituted parts has the interesting side effect of defining a new accessibility level. A constructed type that contains an element that is `Protected` and an element that is `Friend` can only be accessed in contexts that can access *both* `Friend` *and* `Protected` members. However, there is no way to express this accessibility level in the language, as the accessibility `Protected Friend` means that an entity can be accessed in a context that can access *either* `Friend` *or* `Protected` members.

The base, implemented interfaces and members of constructed types are determined by substituting the supplied type arguments for each occurrence of the type parameter in the generic type.

### Open Types and Closed Types

A constructed type for who one or more type arguments are type parameters of a containing type or method is called an *open type*. This is because some of the type parameters of the type are still not known, so the actual shape of the type is not yet fully known. In contrast, a generic type whose type arguments are all non-type parameters is called a *closed type*. The shape of a closed type is always fully known. For example:

```vb
Class Base(Of T, V)
End Class

Class Derived(Of V)
    Inherits Base(Of Integer, V)
End Class

Class MoreDerived
    Inherits Derived(Of Double)
End Class
```

The constructed type `Base(Of Integer, V)` is an open type because although the type parameter `T` has been supplied, the type parameter `U` has been supplied another type parameter. Thus, the full shape of the type is not yet known. The constructed type `Derived(Of Double)`, however, is a closed type because all type parameters in the inheritance hierarchy have been supplied.

Open types are defined as follows:

* A type parameter is an open type.

* An array type is an open type if its element type is an open type.

* A constructed type is an open type if one or more of its type arguments are an open type.

* A closed type is a type that is not an open type.

Because the program entry point cannot be in a generic type, all types used at run-time will be closed types.

## Special Types

The .NET Framework contains a number of classes that are treated specially by the .NET Framework and by the Visual Basic language:

The type `System.Void`, which represents a void type in the .NET Framework, can be directly referenced only in `GetType` expressions.

The types `System.RuntimeArgumentHandle`, `System.ArgIterator` and `System.TypedReference` all can contain pointers into the stack and so cannot appear on the .NET Framework heap. Therefore, they cannot be used as array element types, return types, field types, generic type arguments, nullable types, `ByRef` parameter types, the type of a value being converted to `Object` or `System.ValueType`, the target of a call to instance members of `Object` or `System.ValueType`, or lifted into a closure.
