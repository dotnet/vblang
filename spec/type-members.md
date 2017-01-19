# Type Members

Type members define storage locations and executable code. They can be methods, constructors, events, constants, variables, and properties.

## Interface Method Implementation

Methods, events, and properties can implement interface members. To implement an interface member, a member declaration specifies the `Implements` keyword and lists one or more interface members.

```antlr
ImplementsClause
    : ( 'Implements' ImplementsList )?
    ;

ImplementsList
    : InterfaceMemberSpecifier ( Comma InterfaceMemberSpecifier )*
    ;

InterfaceMemberSpecifier
    : NonArrayTypeName Period IdentifierOrKeyword
    ;
```

Methods and properties that implement interface members are implicitly `NotOverridable` unless declared to be `MustOverride`, `Overridable`, or overriding another member. It is an error for a member implementing an interface member to be `Shared`. A member's accessibility has no effect on its ability to implement interface members.

For an interface implementation to be valid, the implements list of the containing type must name an interface that contains a compatible member. A compatible member is one whose signature matches the signature of the implementing member. If a generic interface is being implemented, then the type argument supplied in the Implements clause is substituted into the signature when checking compatibility. For example:

```vb
Interface I1(Of T)
    Sub F(x As T)
End Interface

Class C1
    Implements I1(Of Integer)

    Sub F(x As Integer) Implements I1(Of Integer).F
    End Sub
End Class

Class C2(Of U)
    Implements I1(Of U)

    Sub F(x As U) Implements I1(Of U).F
    End Sub
End Class
```

If an event declared using a delegate type is implementing an interface event, then a compatible event is one whose underlying delegate type is the same type. Otherwise, the event uses the delegate type from the interface event it is implementing. If such an event implements multiple interface events, all the interface events must have the same underlying delegate type. For example:

```vb
Interface ClickEvents
    Event LeftClick(x As Integer, y As Integer)
    Event RightClick(x As Integer, y As Integer)
End Interface

Class Button
    Implements ClickEvents

    ' OK. Signatures match, delegate type = ClickEvents.LeftClickHandler.
    Event LeftClick(x As Integer, y As Integer) _
        Implements ClickEvents.LeftClick

    ' OK. Signatures match, delegate type = ClickEvents.RightClickHandler.
    Event RightClick(x As Integer, y As Integer) _
        Implements ClickEvents.RightClick
End Class

Class Label
    Implements ClickEvents

    ' Error. Signatures match, but can't be both delegate types.
    Event Click(x As Integer, y As Integer) _
        Implements ClickEvents.LeftClick, ClickEvents.RightClick
End Class
```

An interface member in the implements list is specified using a type name, a period, and an identifier. The type name must be an interface in the implements list or a base interface of an interface in the implements list, and the identifier must be a member of the specified interface. A single member can implement more than one matching interface member.

```vb
Interface ILeft
    Sub F()
End Interface

Interface IRight
    Sub F()
End Interface

Class Test
    Implements ILeft, IRight

    Sub F() Implements ILeft.F, IRight.F
    End Sub
End Class
```

If the interface member being implemented is unavailable in all explicitly implemented interfaces because of multiple interface inheritance, the implementing member must explicitly reference a base interface on which the member is available. For example, if `I1` and `I2` contain a member `M`, and `I3` inherits from `I1` and `I2`, a type implementing `I3` will implement `I1.M` and `I2.M`. If an interface shadows multiply inherited members, an implementing type will have to implement the inherited members and the member(s) shadowing them.

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

Class Test
    Implements ILeftRight

    Sub LeftF() Implements ILeft.F
    End Sub

    Sub RightF() Implements IRight.F
    End Sub

    Sub LeftRightF() Implements ILeftRight.F
    End Sub
End Class
```

If the containing interface of the interface member be implemented is generic, the same type arguments as the interface being implements must be supplied. For example:

```vb
Interface I1(Of T)
    Function F() As T
End Interface

Class C1
    Implements I1(Of Integer)
    Implements I1(Of Double)

    Function F1() As Integer Implements I1(Of Integer).F
    End Function

    Function F2() As Double Implements I1(Of Double).F
    End Function

    ' Error: I1(Of String) is not implemented by C1
    Function F3() As String Implements I1(Of String).F
    End Function
End Class

Class C2(Of U)
    Implements I1(Of U)

    Function F() As U Implements I1(Of U).F
    End Function
End Class
```


## Methods

Methods contain the executable statements of a program.

```antlr
MethodMemberDeclaration
    : MethodDeclaration
    | ExternalMethodDeclaration
    ;

InterfaceMethodMemberDeclaration
    : InterfaceMethodDeclaration
    ;

MethodDeclaration
    : SubDeclaration
    | MustOverrideSubDeclaration
    | FunctionDeclaration
    | MustOverrideFunctionDeclaration
    ;

InterfaceMethodDeclaration
    : InterfaceSubDeclaration
    | InterfaceFunctionDeclaration
    ;

SubSignature
    : 'Sub' Identifier TypeParameterList?
      ( OpenParenthesis ParameterList? CloseParenthesis )?
    ;

FunctionSignature
    : 'Function' Identifier TypeParameterList?
      ( OpenParenthesis ParameterList? CloseParenthesis )?
      ( 'As' Attributes? TypeName )?
    ;

SubDeclaration
    : Attributes? ProcedureModifier* SubSignature
      HandlesOrImplements? LineTerminator
      Block
      'End' 'Sub' StatementTerminator
    ;

MustOverrideSubDeclaration
    : Attributes? MustOverrideProcedureModifier+ SubSignature
      HandlesOrImplements? StatementTerminator
    ;

InterfaceSubDeclaration
    : Attributes? InterfaceProcedureModifier* SubSignature StatementTerminator
    ;

FunctionDeclaration
    : Attributes? ProcedureModifier* FunctionSignature
      HandlesOrImplements? LineTerminator
      Block
      'End' 'Function' StatementTerminator
    ;

MustOverrideFunctionDeclaration
    : Attributes? MustOverrideProcedureModifier+ FunctionSignature
      HandlesOrImplements? StatementTerminator
    ;

InterfaceFunctionDeclaration
    : Attributes? InterfaceProcedureModifier* FunctionSignature StatementTerminator
    ;

ProcedureModifier
    : AccessModifier | 'Shadows' | 'Shared' | 'Overridable' | 'NotOverridable' | 'Overrides'
    | 'Overloads' | 'Partial' | 'Iterator' | 'Async'
    ;

MustOverrideProcedureModifier
    : ProcedureModifier
    | 'MustOverride'
    ;

InterfaceProcedureModifier
    : 'Shadows' | 'Overloads'
    ;

HandlesOrImplements
    : HandlesClause
    | ImplementsClause
    ;
```

Methods, which have an optional list of parameters and an optional return value, are either shared or not shared. Shared methods are accessed through the class or instances of the class. Non-shared methods, also called instance methods, are accessed through instances of the class. The following example shows a class `Stack` that has several shared methods (`Clone` and `Flip`), and several instance methods (`Push`, `Pop`, and `ToString`):

```vb
Public Class Stack
    Public Shared Function Clone(s As Stack) As Stack
        ...
    End Function

    Public Shared Function Flip(s As Stack) As Stack
        ...
    End Function

    Public Function Pop() As Object
        ...
    End Function

    Public Sub Push(o As Object)
        ...
    End Sub 

    Public Overrides Function ToString() As String
        ...
    End Function 
End Class 

Module Test
    Sub Main()
        Dim s As Stack = New Stack()
        Dim i As Integer

        While i < 10
            s.Push(i)
        End While

        Dim flipped As Stack = Stack.Flip(s)
        Dim cloned As Stack = Stack.Clone(s)

        Console.WriteLine("Original stack: " & s.ToString())
        Console.WriteLine("Flipped stack: " & flipped.ToString())
        Console.WriteLine("Cloned stack: " & cloned.ToString())
    End Sub
End Module
```

Methods can be overloaded, which means that multiple methods may have the same name so long as they have unique signatures. The signature of a method consists of the number and types of its parameters. The signature of a method specifically does not include the return type or parameter modifiers such as Optional, ByRef or ParamArray. The following example shows a class with a number of overloads:

```vb
Module Test
    Sub F()
        Console.WriteLine("F()")
    End Sub 

    Sub F(o As Object)
        Console.WriteLine("F(Object)")
    End Sub

    Sub F(value As Integer)
        Console.WriteLine("F(Integer)")
    End Sub 

    Sub F(a As Integer, b As Integer)
        Console.WriteLine("F(Integer, Integer)")
    End Sub 

    Sub F(values() As Integer)
        Console.WriteLine("F(Integer())")
    End Sub 

    Sub G(s As String, Optional s2 As String = 5)
        Console.WriteLine("G(String, Optional String")
    End Sub

    Sub G(s As String)
        Console.WriteLine("G(String)")
    End Sub


    Sub Main()
        F()
        F(1)
        F(CType(1, Object))
        F(1, 2)
        F(New Integer() { 1, 2, 3 })
        G("hello")
        G("hello", "world")
    End Sub
End Module
```

The output of the program is:

```vb
F()
F(Integer)
F(Object)
F(Integer, Integer)
F(Integer())
G(String)
G(String, Optional String)
```

Overloads that differ only in optional parameters can be used for "versioning" of libraries. For instance, v1 of a library might include a function with optional parameters:

```vb
Sub fopen(fileName As String, Optional accessMode as Integer = 0)
```

Then v2 of the library wants to add another optional parameter "password", and it wants to do so without breaking source compatibility (so applications that used to target v1 can be recompiled), and without breaking binary compatibility (so applications that used to reference v1 can now reference v2 without recompilation). This is how v2 will look:

```vb
Sub fopen(file As String, mode as Integer)
Sub fopen(file As String, Optional mode as Integer = 0, Optional pword As String = "")
```

Note that optional parameters in a public API are not CLS-compliant. However, they can be consumed at least by Visual Basic and C#4 and F#.



### Regular, Async and Iterator Method Declarations

There are two types of methods: *subroutines*, which do not return values, and *functions*, which do. The body and `End` construct of a method may only be omitted if the method is defined in an interface or has the `MustOverride` modifier. If no return type is specified on a function and strict semantics are being used, a compile-time error occurs; otherwise the type is implicitly `Object` or the type of the method's type character. The accessibility domain of the return type and parameter types of a method must be the same as or a superset of the accessibility domain of the method itself.

A __regular method__ is one with neither `Async` nor `Iterator` modifiers. It may be a subroutine or a function. Section [Regular Methods](statements.md#regular-methods) details what happens when a regular method is invoked.

An __iterator method__ is one with the `Iterator` modifier and no `Async` modifier. It must be a function, and its return type must be `IEnumerator`, `IEnumerable`, or `IEnumerator(Of T)` or `IEnumerable(Of T)` for some `T`, and it must have no `ByRef` parameters. Section [Iterator Methods](statements.md#iterator-methods) details what happens when an iterator method is invoked.

An __async method__ is one with the `Async` modifier and no `Iterator` modifier. It must be either a subroutine, or a function with return type `Task` or `Task(Of T)` for some `T`, and must have no `ByRef` parameters. Section [Async Methods](statements.md#async-methods) details what happens when an async method is invoked.

It is a compile-time error if a method is not one of these three kinds of method.

Subroutine and function declarations are special in that their beginning and end statements must each start at the beginning of a logical line. Additionally, the body of a non-`MustOverride` subroutine or function declaration must start at the beginning of a logical line. For example:

```vb
Module Test
    ' Illegal: Subroutine doesn't start the line
    Public x As Integer : Sub F() : End Sub

    ' Illegal: First statement doesn't start the line
    Sub G() : Console.WriteLine("G")
    End Sub

    ' Illegal: End Sub doesn't start the line
    Sub H() : End Sub
End Module
```


### External Method Declarations

An external method declaration introduces a new method whose implementation is provided external to the program.

```antlr
ExternalMethodDeclaration
    : ExternalSubDeclaration
    | ExternalFunctionDeclaration
    ;

ExternalSubDeclaration
    : Attributes? ExternalMethodModifier* 'Declare' CharsetModifier? 'Sub'
      Identifier LibraryClause AliasClause?
      ( OpenParenthesis ParameterList? CloseParenthesis )? StatementTerminator
    ;

ExternalFunctionDeclaration
    : Attributes? ExternalMethodModifier* 'Declare' CharsetModifier? 'Function'
      Identifier LibraryClause AliasClause?
      ( OpenParenthesis ParameterList? CloseParenthesis )?
      ( 'As' Attributes? TypeName )?
      StatementTerminator
    ;

ExternalMethodModifier
    : AccessModifier
    | 'Shadows'
    | 'Overloads'
    ;

CharsetModifier
    : 'Ansi' | 'Unicode' | 'Auto'
    ;

LibraryClause
    : 'Lib' StringLiteral
    ;

AliasClause
    : 'Alias' StringLiteral
    ;
```

Because an external method declaration provides no actual implementation, it has no method body or `End` construct. External methods are implicitly shared, may not have type parameters, and may not handle events or implement interface members. If no return type is specified on a function and strict semantics are being used, a compile-time error occurs. Otherwise the type is implicitly `Object` or the type of the method's type character. The accessibility domain of the return type and parameter types of an external method must be the same as or a superset of the accessibility domain of the external method itself.

The library clause of an external method declaration specifies the name of the external file that implements the method. The optional alias clause is a string that specifies the numeric ordinal (prefixed by a `#` character) or name of the method in the external file. A single-character set modifier may also be specified, which governs the character set used to marshal strings during a call to the external method. The `Unicode` modifier marshals all strings to Unicode values, the `Ansi` modifier marshals all strings to ANSI values, and the `Auto` modifier marshals the strings according to .NET Framework rules based on the name of the method, or the alias name if specified. If no modifier is specified, the default is `Ansi`.

If `Ansi` or `Unicode` is specified, then the method name is looked up in the external file with no modification. If `Auto` is specified, then method name lookup depends on the platform. If the platform is considered to be ANSI (for example, Windows 95, Windows 98, Windows ME), then the method name is looked up with no modification. If the lookup fails, an `A` is appended and the lookup tried again. If the platform is considered to be Unicode (for example, Windows NT, Windows 2000, Windows XP), then a `W` is appended and the name is looked up. If the lookup fails, the lookup is tried again without the `W`. For example:

```vb
Module Test
    ' All platforms bind to "ExternSub".
    Declare Ansi Sub ExternSub Lib "ExternDLL" ()

    ' All platforms bind to "ExternSub".
    Declare Unicode Sub ExternSub Lib "ExternDLL" ()

    ' ANSI platforms: bind to "ExternSub" then "ExternSubA".
    ' Unicode platforms: bind to "ExternSubW" then "ExternSub".
    Declare Auto Sub ExternSub Lib "ExternDLL" ()
End Module
```

Data types being passed to external methods are marshaled according to the .NET Framework data marshalling conventions with one exception. String variables that are passed by value (that is, `ByVal x As String`) are marshaled to the OLE Automation BSTR type, and changes made to the BSTR in the external method are reflected back in the string argument. This is because the type `String` in external methods is mutable, and this special marshalling mimics that behavior. String parameters that are passed by reference (i.e. `ByRef x As String`) are marshaled as a pointer to the OLE Automation BSTR type. It is possible to override these special behaviors by specifying the `System.Runtime.InteropServices.MarshalAsAttribute` attribute on the parameter.

The example demonstrates use of external methods:

```vb
Class Path
    Declare Function CreateDirectory Lib "kernel32" ( _
        Name As String, sa As SecurityAttributes) As Boolean
    Declare Function RemoveDirectory Lib "kernel32" ( _
        Name As String) As Boolean
    Declare Function GetCurrentDirectory Lib "kernel32" ( _
        BufSize As Integer, Buf As String) As Integer
    Declare Function SetCurrentDirectory Lib "kernel32" ( _
        Name As String) As Boolean
End Class
```


### Overridable Methods

The `Overridable` modifier indicates that a method is overridable. The `Overrides` modifier indicates that a method overrides a base-type overridable method that has the same signature. The `NotOverridable` modifier indicates that an overridable method cannot be further overridden. The `MustOverride` modifier indicates that a method must be overridden in derived classes.

Certain combinations of these modifiers are not valid:

* `Overridable` and `NotOverridable` are mutually exclusive and cannot be combined.

* `MustOverride` implies `Overridable` (and so cannot specify it) and cannot be combined with `NotOverridable`.

* `NotOverridable` cannot be combined with `Overridable` or `MustOverride` and must be combined with `Overrides`.

* `Overrides` implies `Overridable` (and so cannot specify it) and cannot be combined with `MustOverride`.

There are also additional restrictions on overridable methods:

* A `MustOverride` method may not include a method body or an `End` construct, may not override another method, and may only appear in `MustInherit` classes.

* If a method specifies `Overrides` and there is no matching base method to override, a compile-time error occurs. An overriding method may not specify `Shadows`.

* A method may not override another method if the overriding method's accessibility domain is not equal to the accessibility domain of the method being overridden. The one exception is that a method overriding a `Protected Friend` method in another assembly that does not have `Friend` access must specify `Protected` (not `Protected Friend`).

* `Private` methods may not be `Overridable`, `NotOverridable`, or `MustOverride`, nor may they override other methods.

* Methods in `NotInheritable` classes may not be declared `Overridable` or `MustOverride`.

The following example illustrates the differences between overridable and nonoverridable methods:

```vb
Class Base
    Public Sub F()
        Console.WriteLine("Base.F")
    End Sub

    Public Overridable Sub G()
        Console.WriteLine("Base.G")
    End Sub
End Class

Class Derived
    Inherits Base

    Public Shadows Sub F()
        Console.WriteLine("Derived.F")
    End Sub

    Public Overrides Sub G()
        Console.WriteLine("Derived.G")
    End Sub
End Class

Module Test
    Sub Main()
        Dim d As Derived = New Derived()
        Dim b As Base = d

        b.F()
        d.F()
        b.G()
        d.G()
    End Sub
End Module
```

In the example, class `Base` introduces a method `F` and an `Overridable` method `G`. The class `Derived` introduces a new method `F`, thus shadowing the inherited `F`, and also overrides the inherited method `G`. The example produces the following output:

```
Base.F
Derived.F
Derived.G
Derived.G
```

Notice that the statement `b.G()` invokes `Derived.G`, not `Base.G`. This is because the run-time type of the instance (which is `Derived`) rather than the compile-time type of the instance (which is `Base`) determines the actual method implementation to invoke.

### Shared Methods

The `Shared` modifier indicates a method is a *shared method*. A shared method does not operate on a specific instance of a type and may be invoked directly from a type rather than through a particular instance of a type. It is valid, however, to use an instance to qualify a shared method. It is invalid to refer to `Me`, `MyClass`, or `MyBase` in a shared method. Shared methods may not be `Overridable`, `NotOverridable`, or `MustOverride`, and they may not override methods. Methods defined in standard modules and interfaces may not specify `Shared`, because they are implicitly `Shared` already.

A method declared in a structure or class without a `Shared` modifier is an *instance method*. An instance method operates on a given instance of a type. Instance methods can only be invoked through an instance of a type and may refer to the instance through the `Me` expression.

The following example illustrates the rules for accessing shared and instance members:

```vb
Class Test
    Private x As Integer
    Private Shared y As Integer

    Sub F()
        x = 1 ' Ok, same as Me.x = 1.
        y = 1 ' Ok, same as Test.y = 1.
    End Sub

    Shared Sub G()
        x = 1 ' Error, cannot access Me.x.
        y = 1 ' Ok, same as Test.y = 1.
    End Sub

    Shared Sub Main()
        Dim t As Test = New Test()

        t.x = 1 ' Ok.
        t.y = 1 ' Ok.
        Test.x = 1 ' Error, cannot access instance member through type.
        Test.y = 1 ' Ok.
    End Sub
End Class
```

Method `F` shows that in an instance function member, an identifier can be used to access both instance members and shared members. Method `G` shows that in a shared function member, it is an error to access an instance member through an identifier. Method `Main` shows that in a member access expression, instance members must be accessed through instances, but shared members can be accessed through types or instances.

### Method Parameters

A *parameter* is a variable that can be used to pass information into and out of a method. Parameters of a method are declared by the method's parameter list, which consists of one or more parameters separated by commas.

```antlr
ParameterList
    : Parameter ( Comma Parameter )*
    ;

Parameter
    : Attributes? ParameterModifier* ParameterIdentifier ( 'As' TypeName )?
      ( Equals ConstantExpression )?
    ;

ParameterModifier
    : 'ByVal' | 'ByRef' | 'Optional' | 'ParamArray'
    ;

ParameterIdentifier
    : Identifier IdentifierModifiers
    ;
```

If no type is specified for a parameter and strict semantics are used, a compile-time error occurs. Otherwise the default type is `Object` or the type of the parameter's type character. Even under permissive semantics, if one parameter includes an `As` clause, all parameters must specify types.

Parameters are specified as value, reference, optional, or paramarray parameters by the modifiers `ByVal`, `ByRef`, `Optional`, and `ParamArray`, respectively. A parameter that does not specify `ByRef` or `ByVal` defaults to `ByVal`.

Parameter names are scoped to the entire body of the method and are always publicly accessible. A method invocation creates a copy, specific to that invocation, of the parameters, and the argument list of the invocation assigns values or variable references to the newly created parameters. Because external method declarations and delegate declarations have no body, duplicate parameter names are allowed in parameter lists, but discouraged.

The identifier may be followed by the nullable name modifier `?` to indicate that it is nullable, and also by array name modifiers to indicate that it is an array. They may be combined, e.g. "`ByVal x?() As Integer`". It is not allowed to use explicit array bounds; also, if the nullable name modifier is present then an `As` clause must be present.


#### Value Parameters

A *value parameter* is declared with an explicit `ByVal` modifier. If the `ByVal` modifier is used, the `ByRef` modifier may not be specified. A value parameter comes into existence with the invocation of the member the parameter belongs to, and is initialized with the value of the argument given in the invocation. A value parameter ceases to exist upon return of the member.

A method is permitted to assign new values to a value parameter. Such assignments only affect the local storage location represented by the value parameter; they have no effect on the actual argument given in the method invocation.

A value parameter is used when the value of an argument is passed into a method, and modifications of the parameter do not impact the original argument. A value parameter refers to its own variable, one that is distinct from the variable of the corresponding argument. This variable is initialized by copying the value of the corresponding argument. The following example shows a method `F` that has a value parameter named `p`:

```vb
Module Test
    Sub F(p As Integer)
        Console.WriteLine("p = " & p)
        p += 1
    End Sub 

    Sub Main()
        Dim a As Integer = 1

        Console.WriteLine("pre: a = " & a)
        F(a)
        Console.WriteLine("post: a = " & a)
    End Sub
End Module
```

The example produces the following output, even though the value parameter `p` is modified:

```
pre: a = 1
p = 1
post: a = 1
```

#### Reference Parameters

A reference parameter is a parameter declared with a `ByRef` modifier. If the `ByRef` modifier is specified, the `ByVal` modifier may not be used. A reference parameter does not create a new storage location. Instead, a reference parameter represents the variable given as the argument in the method or constructor invocation. Conceptually, the value of a reference parameter is always the same as the underlying variable.

Reference parameters act in two modes, either as *aliases* or through *copy-in copy-back.*

__Aliases.__ A reference parameter is used when the parameter acts as an alias for a caller-provided argument. A reference parameter does not itself define a variable, but rather refers to the variable of the corresponding argument. Modifications of a reference parameter directly and immediately impact the corresponding argument. The following example shows a method `Swap` that has two reference parameters:

```vb
Module Test
    Sub Swap(ByRef a As Integer, ByRef b As Integer)
        Dim t As Integer = a
        a = b
        b = t
    End Sub 

    Sub Main()
        Dim x As Integer = 1
        Dim y As Integer = 2

        Console.WriteLine("pre: x = " & x & ", y = " & y)
        Swap(x, y)
        Console.WriteLine("post: x = " & x & ", y = " & y)
    End Sub 
End Module
```

The output of the program is:

```
pre: x = 1, y = 2
post: x = 2, y = 1
```

For the invocation of method `Swap` in class `Main`, `a` represents `x,` and `b` represents `y`. Thus, the invocation has the effect of swapping the values of `x` and `y`.

In a method that takes reference parameters, it is possible for multiple names to represent the same storage location:

```vb
Module Test
    Private s As String

    Sub F(ByRef a As String, ByRef b As String)
        s = "One"
        a = "Two"
        b = "Three"
    End Sub

    Sub G()
        F(s, s)
    End Sub
End Module
```

In the example the invocation of method `F` in `G` passes a reference to `s` for both `a` and `b`. Thus, for that invocation, the names `s`, `a`, and `b` all refer to the same storage location, and the three assignments all modify the instance variable `s`.

__Copy-in copy-back.__ If the type of the variable being passed to a reference parameter is not compatible with the reference parameter's type, or if a non-variable (e.g. a property) is passed as an argument to a reference parameter, or if the invocation is late-bound, then a temporary variable is allocated and passed to the reference parameter. The value being passed in will be copied into this temporary variable before the method is invoked and will be copied back to the original variable (if there is one and if it's writable) when the method returns. Thus, a reference parameter may not necessarily contain a reference to the exact storage of the variable being passed in, and any changes to the reference parameter may not be reflected in the variable until the method exits. For example:

```vb
Class Base
End Class

Class Derived
    Inherits Base
End Class

Module Test
    Sub F(ByRef b As Base)
        b = New Base()
    End Sub

    Property G() As Base
        Get
        End Get
        Set
        End Set
    End Property

    Sub Main()
        Dim d As Derived

        F(G)   ' OK.
        F(d)   ' Throws System.InvalidCastException after F returns.
    End Sub
End Module
```

In the case of the first invocation of `F`, a temporary variable is created and the value of the property `G` is assigned to it and passed into `F`. Upon return from `F`, the value in the temporary variable is assigned back to the property of `G`. In the second case, another temporary variable is created and the value of `d` is assigned to it and passed into `F`. When returning from `F`, the value in the temporary variable is cast back to the type of the variable, `Derived`, and assigned to `d`. Since the value being passed back cannot be cast to `Derived`, an exception is thrown at run time.

#### Optional Parameters

An optional parameter is declared with the `Optional` modifier. Parameters that follow an optional parameter in the formal parameter list must be optional as well; failure to specify the `Optional` modifier on the following parameters will trigger a compile-time error. An optional parameter of some type nullable type `T?` or non-nullable type `T` must specify a constant expression `e` to be used as a default value if no argument is specified. If `e` evaluates to `Nothing` of type Object, then the default value of the *parameter type* will be used as the default for the parameter. Otherwise, `CType(e, T)` must be a constant expression and it is taken as the default for the parameter.

Optional parameters are the only situation in which an initializer on a parameter is valid. The initialization is always done as a part of the invocation expression, not within the method body itself.

```vb
Module Test
    Sub F(x As Integer, Optional y As Integer = 20)
        Console.WriteLine("x = " & x & ", y = " & y)
    End Sub

    Sub Main()
        F(10)
        F(30,40)
    End Sub
End Module
```

The output of the program is:

```
x = 10, y = 20
x = 30, y = 40
```

Optional parameters may not be specified in delegate or event declarations, nor in lambda expressions.

#### ParamArray Parameters

`ParamArray` parameters are declared with the `ParamArray` modifier. If the `ParamArray` modifier is present, the `ByVal` modifier must be specified, and no other parameter may use the `ParamArray` modifier. The `ParamArray` parameter's type must be a one-dimensional array, and it must be the last parameter in the parameter list.

A `ParamArray` parameter represents an indeterminate number of parameters of the type of the `ParamArray`. Within the method itself, a `ParamArray` parameter is treated as its declared type and has no special semantics. A `ParamArray` parameter is implicitly optional, with a default value of an empty one-dimensional array of the type of the `ParamArray`.

A `ParamArray` permits arguments to be specified in one of two ways in a method invocation:

* The argument given for a `ParamArray` can be a single expression of a type that widens to the `ParamArray` type. In this case, the `ParamArray` acts precisely like a value parameter.

* Alternatively, the invocation can specify zero or more arguments for the `ParamArray`, where each argument is an expression of a type that is implicitly convertible to the element type of the `ParamArray`. In this case, the invocation creates an instance of the `ParamArray` type with a length corresponding to the number of arguments, initializes the elements of the array instance with the given argument values, and uses the newly created array instance as the actual argument.

Except for allowing a variable number of arguments in an invocation, a `ParamArray` is precisely equivalent to a value parameter of the same type, as the following example illustrates.

```vb
Module Test
    Sub F(ParamArray args() As Integer)
        Dim i As Integer

        Console.Write("Array contains " & args.Length & " elements:")
        For Each i In args
            Console.Write(" " & i)
        Next i
        Console.WriteLine()
    End Sub

    Sub Main()
        Dim a As Integer() = { 1, 2, 3 }

        F(a)
        F(10, 20, 30, 40)
        F()
    End Sub
End Module
```

The example produces the output

```
Array contains 3 elements: 1 2 3
Array contains 4 elements: 10 20 30 40
Array contains 0 elements:
```

The first invocation of `F` simply passes the array `a` as a value parameter. The second invocation of `F` automatically creates a four-element array with the given element values and passes that array instance as a value parameter. Likewise, the third invocation of `F` creates a zero-element array and passes that instance as a value parameter. The second and third invocations are precisely equivalent to writing:

```vb
F(New Integer() {10, 20, 30, 40})
F(New Integer() {})
```

`ParamArray` parameters may not be specified in delegate or event declarations.

### Event Handling

Methods can declaratively handle events raised by objects in instance or shared variables. To handle events, a method declaration specifies the `Handles` keyword and lists one or more events.

```antlr
HandlesClause
    : ( 'Handles' EventHandlesList )?
    ;

EventHandlesList
    : EventMemberSpecifier ( Comma EventMemberSpecifier )*
    ;

EventMemberSpecifier
    : Identifier Period IdentifierOrKeyword
    | 'MyBase' Period IdentifierOrKeyword
    | 'MyClass' Period IdentifierOrKeyword
    | 'Me' Period IdentifierOrKeyword
    ;
```

An event in the `Handles` list is specified by two identifiers separated by a period:

* The first identifier must be an instance or shared variable in the containing type that specifies the `WithEvents` modifier or the `MyBase` or `MyClass` or `Me` keyword; otherwise, a compile-time error occurs. This variable contains the object that will raise the events handled by this method.

* The second identifier must specify a member of the type of the first identifier. The member must be an event, and may be shared. If a shared variable is specified for the first identifier, then the event must be shared, or an error results.

A handler method `M` is considered a valid event handler for an event `E` if the statement `AddHandler E, AddressOf M` would also be valid. Unlike an `AddHandler` statement, however, explicit event handlers allow handling an event with a method with no arguments regardless of whether strict semantics are being used or not:

```vb
Option Strict On

Class C1
    Event E(x As Integer)
End Class

Class C2
    withEvents C1 As New C1()

    ' Valid
    Sub M1() Handles C1.E
    End Sub

    Sub M2()
        ' Invalid
        AddHandler C1.E, AddressOf M1
    End Sub
End Class
```

A single member can handle multiple matching events, and multiple methods may handle a single event. A method's accessibility has no effect on its ability to handle events. The following example shows how a method can handle events:

```vb
Class Raiser
    Event E1()

    Sub Raise()
        RaiseEvent E1
    End Sub
End Class

Module Test
    WithEvents x As Raiser

    Sub E1Handler() Handles x.E1
        Console.WriteLine("Raised")
    End Sub

    Sub Main()
        x = New Raiser()
        x.Raise()
        x.Raise()
    End Sub
End Module
```

This will print out:

```
Raised
Raised
```

A type inherits all event handlers provided by its base type. A derived type cannot in any way alter the event mappings it inherits from its base types, but may add additional handlers to the event.


### Extension Methods

Methods can be added to types from outside of the type declaration using *extension methods*. Extension methods are methods with the `System.Runtime.CompilerServices.ExtensionAttribute` attribute applied to them. They can only be declared in standard modules and must have at least one parameter, which specifies the type the method extends. For example, the following extension method extends the type `String`:

```vb
Imports System.Runtime.CompilerServices

Module StringExtensions
    <Extension> _
    Sub Print(s As String)
        Console.WriteLine(s)
    End Sub
End Module
```

__Note.__ Although Visual Basic requires extension methods to be declared in a standard module, other languages such as C# may allow them to be declared in other kinds of types. As long as the methods follow the other conventions outlined here and the containing type is not an open generic type and cannot be instantiated, Visual Basic will recognize the extension methods.

When an extension method is invoked, the instance it is being invoked on is passed to the first parameter. The first parameter cannot be declared `Optional` or `ParamArray`. Any type, including a type parameter, can appear as the first parameter of an extension method. For example, the following methods extend the types `Integer()`, any type that implements `System.Collections.Generic.IEnumerable(Of T)`, and any type at all:

```vb
Imports System.Runtime.CompilerServices

Module Extensions
    <Extension> _
    Sub PrintArray(a() As Integer)
        ...
    End Sub

    <Extension> _
    Sub PrintList(Of T)(a As IEnumerable(Of T))
        ...
    End Sub

    <Extension> _
    Sub Print(Of T)(a As T)
        ...
    End Sub
End Module
```

As the previous example shows, interfaces can be extended. Interface extension methods supply the implementation of the method, so types that implement an interface that has extension methods defined on it still only implement the members originally declared by the interface. For example:

```vb
Imports System.Runtime.CompilerServices

Interface IAction
  Sub DoAction()
End Interface

Module IActionExtensions 
    <Extension> _
    Public Sub DoAnotherAction(i As IAction) 
        i.DoAction()
    End Sub
End Module

Class C
  Implements IAction

  Sub DoAction() Implements IAction.DoAction
    ...
  End Sub

  ' ERROR: Cannot implement extension method IAction.DoAnotherAction
  Sub DoAnotherAction() Implements IAction.DoAnotherAction
    ...
  End Sub
End Class
```

Extension methods can also have type constraints on their type parameters and, just as with non-extension generic methods, type argument can be inferred:

```vb
Imports System.Runtime.CompilerServices

Module IEnumerableComparableExtensions
    <Extension> _
    Public Function Sort(Of T As IComparable(Of T))(i As IEnumerable(Of T)) _
        As IEnumerable(Of T)
        ...
    End Function
End Module
```

Extension methods can also be accessed through implicit instance expressions within the type being extended:

```vb
Imports System.Runtime.CompilerServices

Class C1
    Sub M1()
        Me.M2()
        M2()
    End Sub
End Class

Module C1Extensions
    <Extension>
    Sub M2(c As C1)
        ...
    End Sub
End Module
```

For the purposes of accessibility, extension methods are also treated as members of the standard module they are declared in -- they have no extra access to the members of the type they are extending beyond the access they have by virtue of their declaration context.

Extensions methods are only available when the standard module method is in scope. Otherwise, the original type will not appear to have been extended. For example:

```vb
Imports System.Runtime.CompilerServices

Class C1
End Class

Namespace N1
    Module C1Extensions
        <Extension> _
        Sub M1(c As C1)
            ...
        End Sub
    End Module
End Namespace

Module Test
    Sub Main()
        Dim c As New C1()

        ' Error: c has no member named "M1"
        c.M1()
    End Sub
End Module
```

Referring to a type when only an extension method on the type is available will still produce a compile-time error.

It is important to note that extension methods are considered to be members of the type in all contexts where members are bound, such as the strongly-typed `For Each` pattern. For example:

```vb
Imports System.Runtime.CompilerServices

Class C1
End Class

Class C1Enumerator
    ReadOnly Property Current() As C1
        Get
            ...
        End Get
    End Property

    Function MoveNext() As Boolean
        ...
    End Function
End Class

Module C1Extensions
    <Extension> _
    Function GetEnumerator(c As C1) As C1Enumerator
        ...
    End Function
End Module

Module Test
    Sub Main()
        Dim c As New C1()

        ' Valid
        For Each o As Object In c
            ...
        Next o
    End Sub
End Module
```

Delegates can also be created that refer to extension methods. Thus, the code:

```vb
Delegate Sub D1()

Module Test
    Sub Main()
        Dim s As String = "Hello, World!"
        Dim d As D1

        d = AddressOf s.Print
        d()
    End Sub
End Module
```

is roughly equivalent to:

```vb
Delegate Sub D1()

Module Test
    Sub Main()
      Dim s As String = "Hello, World!"
      Dim d As D1

      d = CType([Delegate].CreateDelegate(GetType(D1), s, _
                GetType(StringExtensions).GetMethod("Print")), D1)
      d()
    End Sub
End Module
```

__Note.__ Visual Basic normally inserts a check on an instance method call that causes a `System.NullReferenceException` to occur if the instance the method is being invoked on is `Nothing`. In the case of extension methods, there is no efficient way to insert this check, so extension methods will need to explicitly check for `Nothing`. 

__Note.__ A value type will be boxed when being passed as a `ByVal` argument to a parameter typed as an interface.  This implies that side effects of the extension method will operate on a copy of the structure instead of the original. While the language puts no restrictions on the first argument of an extension method, it is recommended that extension methods are not used to extend value types or that when extending value types, the first parameter is passed `ByRef` to ensure that side effects operate on the original value.

### Partial Methods

A *partial method* is a method that specifies a signature but not the body of the method. The body of the method can be supplied by another method declaration with the same name and signature, most likely in another partial declaration of the type. For example:

a.vb:

```vb
' Designer generated code
Public Partial Class MyForm
    Private Partial Sub ValidateControls()
    End Sub

    Public Sub New()
        ' Initialize controls
        ...

        ValidateControls()
    End Sub    
End Class
```

b.vb:

```vb
Public Partial Class MyForm
    Public Sub ValidateControls()
        ' Validation logic goes here
        ...
    End Sub
End Class
```

In this example, a partial declaration of the class `MyForm` declares a partial method `ValidateControls` with no implementation. The constructor in the partial declaration calls the partial method, even though there is no body supplied in the file. The other partial declaration of `MyForm` then supplies the implementation of the method.

Partial methods can be called regardless of whether a body has been supplied; if no method body is supplied, the call is ignored. For example:

```vb
Public Class C1
    Private Partial Sub M1()
    End Sub

    Public Sub New()
        ' Since no implementation is supplied, this call will not be made.
        M1()
    End Sub
End Class
```

Any expressions that are passed in as arguments to a partial method call that is ignored are ignored also and not evaluated. (__Note.__ This means that partial methods are a very efficient way of providing behavior that is defined across two partial types, since the partial methods have no cost if they are not used.)

The partial method declaration must be declared as `Private` and must always be a subroutine with no statements in its body. Partial methods cannot themselves implement interface methods, although the method that supplies their body can.

Only one method can supply a body to a partial method. A method supplying a body to a partial method must have the same signature as the partial method, the same constraints on any type parameters, the same declaration modifiers, and the same parameter and type parameter names. Attributes on the partial method and the method that supplies its body are merged, as are any attributes on the methods' parameters. Similarly, the list of events that the methods handle is merged. For example:

```vb
Class C1
    Event E1()
    Event E2()

    Private Partial Sub S() Handles Me.E1
    End Sub

    ' Handles both E1 and E2
    Private Sub S() Handles Me.E2
        ...
    End Sub
End Class
```

## Constructors

*Constructors* are special methods that allow control over initialization. They are run after the program begins or when an instance of a type is created. Unlike other members, constructors are not inherited and do not introduce a name into a type's declaration space. Constructors may only be invoked by object-creation expressions or by the .NET Framework; they may never be directly invoked.

__Note.__ Constructors have the same restriction on line placement that subroutines have. The beginning statement, end statement and block must all appear at the beginning of a logical line.

```antlr
ConstructorMemberDeclaration
    : Attributes? ConstructorModifier* 'Sub' 'New'
      ( OpenParenthesis ParameterList? CloseParenthesis )? LineTerminator
      Block?
      'End' 'Sub' StatementTerminator
    ;

ConstructorModifier
    : AccessModifier
    | 'Shared'
    ;
```

### Instance Constructors

*Instance constructors* initialize instances of a type and are run by the .NET Framework when an instance is created. The parameter list of a constructor is subject to the same rules as the parameter list of a method. Instance constructors may be overloaded.

All constructors in reference types must invoke another constructor. If the invocation is explicit, it must be the first statement in the constructor method body. The statement can either invoke another of the type's instance constructors -- for example, `Me.New(...)` or `MyClass.New(...)` -- or if it is not a structure it can invoke an instance constructor of the type's base type -- for example, `MyBase.New(...)`. It is invalid for a constructor to invoke itself. If a constructor omits a call to another constructor, `MyBase.New()` is implicit. If there is no parameterless base type constructor, a compile-time error occurs. Because `Me` is not considered to be constructed until after the call to a base class constructor, the parameters to a constructor invocation statement cannot reference `Me`, `MyClass`, or `MyBase` implicitly or explicitly.

When a constructor's first statement is of the form `MyBase.New(...)`, the constructor implicitly performs the initializations specified by the variable initializers of the instance variables declared in the type. This corresponds to a sequence of assignments that are executed immediately after invoking the direct base type constructor. Such ordering ensures that all base instance variables are initialized by their variable initializers before any statements that have access to the instance are executed. For example:

```vb
Class A
    Protected x As Integer = 1
End Class

Class B
    Inherits A

    Private y As Integer = x

    Public Sub New()
        Console.WriteLine("x = " & x & ", y = " & y)
    End Sub
End Class
```

When `New B()` is used to create an instance of `B`, the following output is produced:

```
x = 1, y = 1
```

The value of `y` is `1` because the variable initializer is executed after the base class constructor is invoked. Variable initializers are executed in the textual order they appear in the type declaration.

When a type declares only `Private` constructors, it is not possible in general for other types to derive from the type or create instances of the type; the only exception is types nested within the type. `Private` constructors are commonly used in types that contain only `Shared` members.

If a type contains no instance constructor declarations, a default constructor is automatically provided. The default constructor simply invokes the parameterless constructor of the direct base type. If the direct base type does not have an accessible parameterless constructor, a compile-time error occurs. The declared access type for the default constructor is `Public` unless the type is `MustInherit`, in which case the default constructor is `Protected`.

__Note.__ The default access for a `MustInherit` type's default constructor is `Protected` because `MustInherit` classes cannot be created directly. So there is no point in making the default constructor `Public`.

In the following example a default constructor is provided because the class contains no constructor declarations:

```vb
Class Message
    Dim sender As Object
    Dim text As String
End Class
```

Thus, the example is precisely equivalent to the following:

```vb
Class Message
    Dim sender As Object
    Dim text As String

    Sub New()
    End Sub
End Class
```

Default constructors that are emitted into a designer generated class marked with the attribute `Microsoft.VisualBasic.CompilerServices.DesignerGeneratedAttribute` will call the method `Sub InitializeComponent()`, if it exists, after the call to the base constructor. (__Note.__ This allows designer generated files, such as those created by the WinForms designer, to omit the constructor in the designer file. This enables the programmer to specify it themselves, if they so choose.)

### Shared Constructors

*Shared constructors* initialize a type's shared variables; they are run after the program begins executing, but before any references to a member of the type. A shared constructor specifies the `Shared` modifier, unless it is in a standard module in which case the `Shared` modifier is implied.

Unlike instance constructors, shared constructors have implicit public access, have no parameters, and may not call other constructors. Before the first statement in a shared constructor, the shared constructor implicitly performs the initializations specified by the variable initializers of the shared variables declared in the type. This corresponds to a sequence of assignments that are executed immediately upon entry to the constructor. The variable initializers are executed in the textual order they appear in the type declaration.

The following example shows an `Employee` class with a shared constructor that initializes a shared variable:

```vb
Imports System.Data

Class Employee
    Private Shared ds As DataSet

    Shared Sub New()
        ds = New DataSet()
    End Sub

    Public Name As String
    Public Salary As Decimal
End Class
```

A separate shared constructor exists for each closed generic type. Because the shared constructor is executed exactly once for each closed type, it is a convenient place to enforce run-time checks on the type parameter that cannot be checked at compile-time via constraints. For example, the following type uses a shared constructor to enforce that the type parameter is `Integer` or `Double`:

```vb
Class EnumHolder(Of T)
    Shared Sub New() 
        If Not GetType(T).IsEnum() Then
            Throw New ArgumentException("T must be an enumerated type.")
        End If
    End Sub
End Class
```

Exactly when shared constructors are run is mostly implementation dependent, though several guarantees are provided if a shared constructor is explicitly defined:

* Shared constructors are run before the first access to any static field of the type.

* Shared constructors are run before the first invocation of any static method of the type.

* Shared constructors are run before the first invocation of any constructor for the type.

The above guarantees do not apply in the situation where a shared constructor is implicitly created for shared initializers. The output from the following example is uncertain, because the exact ordering of loading and therefore of shared constructor execution is not defined:

```vb
Module Test
    Sub Main()
        A.F()
        B.F()
    End Sub
End Module

Class A
    Shared Sub New()
        Console.WriteLine("Init A")
    End Sub

    Public Shared Sub F()
        Console.WriteLine("A.F")
    End Sub
End Class

Class B
    Shared Sub New()
        Console.WriteLine("Init B")
    End Sub

    Public Shared Sub F()
        Console.WriteLine("B.F")
    End Sub
End Class
```

The output could be either of the following:

```
Init A
A.F
Init B
B.F
```

or

```
Init B
Init A
A.F
B.F
```

By contrast, the following example produces predictable output. Note that the `Shared` constructor for the class `A` never executes, even though class `B` derives from it:

```vb
Module Test
    Sub Main()
        B.G()
    End Sub
End Module

Class A
    Shared Sub New()
        Console.WriteLine("Init A")
    End Sub
End Class

Class B
    Inherits A

    Shared Sub New()
        Console.WriteLine("Init B")
    End Sub

    Public Shared Sub G()
        Console.WriteLine("B.G")
    End Sub
End Class
```

The output is:

```
Init B
B.G
```

It is also possible to construct circular dependencies that allow `Shared` variables with variable initializers to be observed in their default value state, as in the following example:

```vb
Class A
    Public Shared X As Integer = B.Y + 1
End Class

Class B
    Public Shared Y As Integer = A.X + 1

    Shared Sub Main()
        Console.WriteLine("X = " & A.X & ", Y = " & B.Y)
    End Sub
End Class
```

This produces the output:

```
X = 1, Y = 2
```

To execute the `Main` method, the system first loads class `B`. The `Shared` constructor of class `B` proceeds to compute the initial value of `Y`, which recursively causes class `A` to be loaded because the value of `A.X` is referenced. The `Shared` constructor of class `A` in turn proceeds to compute the initial value of `X`, and in doing so fetches the *default* value of `Y`, which is zero. `A.X` is thus initialized to `1`. The process of loading `A` then completes, returning to the calculation of the initial value of `Y`, the result of which becomes `2`.

Had the `Main` method instead been located in class `A`, the example would have produced the following output:

```
X = 2, Y = 1
```

Avoid circular references in `Shared` variable initializers since it is generally impossible to determine the order in which classes containing such references are loaded.

## Events

Events are used to notify code of a particular occurrence. An event declaration consists of an identifier, either a delegate type or a parameter list, and an optional `Implements` clause.

```antlr
EventMemberDeclaration
    : RegularEventMemberDeclaration
    | CustomEventMemberDeclaration
    ;

RegularEventMemberDeclaration
    : Attributes? EventModifiers* 'Event'
      Identifier ParametersOrType ImplementsClause? StatementTerminator
    ;

InterfaceEventMemberDeclaration
    : Attributes? InterfaceEventModifiers* 'Event'
      Identifier ParametersOrType StatementTerminator
    ;

ParametersOrType
    : ( OpenParenthesis ParameterList? CloseParenthesis )?
    | 'As' NonArrayTypeName
    ;

EventModifiers
    : AccessModifier
    | 'Shadows'
    | 'Shared'
    ;

InterfaceEventModifiers
    : 'Shadows'
    ;
```

If a delegate type is specified, the delegate type may not have a return type. If a parameter list is specified, it may not contain `Optional` or `ParamArray` parameters. The accessibility domain of the parameter types and/or delegate type must be the same as, or a superset of, the accessibility domain of the event itself. Events may be shared by specifying the `Shared` modifier.

In addition to the member name added to the type's declaration space, an event declaration implicitly declares several other members. Given an event named `X`, the following members are added to the declaration space:

* If the form of the declaration is a method declaration, a nested delegate class named `XEventHandler` is introduced. The nested delegate class matches the method declaration and has the same accessibility as the event. The attributes in the parameter list apply to the parameters of the delegate class.

* A `Private` instance variable typed as the delegate, named `XEvent`.

* Two methods named `add_X` and `remove_X` which cannot be invoked, overridden or overloaded.

If a type attempts to declare a name that matches one of the above names, a compile-time error will result, and the implicit `add_X` and `remove_X` declarations are ignored for the purposes of name binding. It is not possible to override or overload any of the introduced members, although it is possible to shadow them in derived types. For example, the class declaration

```vb
Class Raiser
    Public Event Constructed(i As Integer)
End Class
```

is equivalent to the following declaration

```vb
Class Raiser
    Public Delegate Sub ConstructedEventHandler(i As Integer)

    Protected ConstructedEvent As ConstructedEventHandler

    Public Sub add_Constructed(d As ConstructedEventHandler)
        ConstructedEvent = _
            CType( _
                [Delegate].Combine(ConstructedEvent, d), _
                    Raiser.ConstructedEventHandler)
    End Sub

    Public Sub remove_Constructed(d As ConstructedEventHandler)
        ConstructedEvent = _
            CType( _
                [Delegate].Remove(ConstructedEvent, d), _
                    Raiser.ConstructedEventHandler)
    End Sub
End Class
```

Declaring an event without specifying a delegate type is the simplest and most compact syntax, but has the disadvantage of declaring a new delegate type for each event. For example, in the following example, three hidden delegate types are created, even though all three events have the same parameter list:

```vb
Public Class Button
    Public Event Click(sender As Object, e As EventArgs)
    Public Event DoubleClick(sender As Object, e As EventArgs)
    Public Event RightClick(sender As Object, e As EventArgs)
End Class
```

In the following example, the events simply use the same delegate, `EventHandler`:

```vb
Public Delegate Sub EventHandler(sender As Object, e As EventArgs)

Public Class Button
    Public Event Click As EventHandler
    Public Event DoubleClick As EventHandler
    Public Event RightClick As EventHandler
End Class
```

Events can be handled in one of two ways: statically or dynamically. Statically handling events is simpler and only requires a `WithEvents` variable and a `Handles` clause. In the following example, class `Form1` statically handles the event `Click` of object `Button`:

```vb
Public Class Form1
    Public WithEvents Button1 As New Button()

    Public Sub Button1_Click(sender As Object, e As EventArgs) _
           Handles Button1.Click
        Console.WriteLine("Button1 was clicked!")
    End Sub
End Class
```

Dynamically handling events is more complex because the event must be explicitly connected and disconnected to in code. The statement `AddHandler` adds a handler for an event, and the statement `RemoveHandler` removes a handler for an event. The next example shows a class `Form1` that adds `Button1_Click` as an event handler for `Button1`'s `Click` event:

```vb
Public Class Form1
    Public Sub New()
        ' Add Button1_Click as an event handler for Button1's Click event.
        AddHandler Button1.Click, AddressOf Button1_Click
    End Sub 

    Private Button1 As Button = New Button()

    Sub Button1_Click(sender As Object, e As EventArgs)
        Console.WriteLine("Button1 was clicked!")
    End Sub

    Public Sub Disconnect()
        RemoveHandler Button1.Click, AddressOf Button1_Click
    End Sub 
End Class
```

In method `Disconnect`, the event handler is removed.


### Custom Events

As discussed in the previous section, event declarations implicitly define a field, an `add_` method, and a `remove_` method that are used to keep track of event handlers. In some situations, however, it may be desirable to provide custom code for tracking event handlers. For example, if a class defines forty events of which only a few will ever be handled, using a hash table instead of forty fields to track the handlers for each event may be more efficient. *Custom events* allow the `add_X` and `remove_X` methods to be defined explicitly, which enables custom storage for event handlers.

Custom events are declared in the same way that events that specify a delegate type are declared, with the exception that the keyword `Custom` must precede the `Event` keyword. A custom event declaration contains three declarations: an `AddHandler` declaration, a `RemoveHandler` declaration and a `RaiseEvent` declaration. None of the declarations can have any modifiers, although they can have attributes.

```antlr
CustomEventMemberDeclaration
    : Attributes? EventModifiers* 'Custom' 'Event'
      Identifier 'As' TypeName ImplementsClause? StatementTerminator
      EventAccessorDeclaration+
      'End' 'Event' StatementTerminator
    ;

EventAccessorDeclaration
    : AddHandlerDeclaration
    | RemoveHandlerDeclaration
    | RaiseEventDeclaration
    ;

AddHandlerDeclaration
    : Attributes? 'AddHandler'
      OpenParenthesis ParameterList CloseParenthesis LineTerminator
      Block?
      'End' 'AddHandler' StatementTerminator
    ;

RemoveHandlerDeclaration
    : Attributes? 'RemoveHandler'
      OpenParenthesis ParameterList CloseParenthesis LineTerminator
      Block?
      'End' 'RemoveHandler' StatementTerminator
    ;

RaiseEventDeclaration
    : Attributes? 'RaiseEvent'
      OpenParenthesis ParameterList CloseParenthesis LineTerminator
      Block?
      'End' 'RaiseEvent' StatementTerminator
    ;
```

For example:

```vb
Class Test
    Private Handlers As EventHandler

    Public Custom Event TestEvent As EventHandler
        AddHandler(value As EventHandler)
            Handlers = CType([Delegate].Combine(Handlers, value), _
                EventHandler)
        End AddHandler

        RemoveHandler(value as EventHandler)
            Handlers = CType([Delegate].Remove(Handlers, value), _
                EventHandler)
        End RemoveHandler

        RaiseEvent(sender As Object, e As EventArgs)
            Dim TempHandlers As EventHandler = Handlers

            If TempHandlers IsNot Nothing Then
                TempHandlers(sender, e)
            End If
        End RaiseEvent
    End Event
End Class
```

The `AddHandler` and `RemoveHandler` declaration take one `ByVal` parameter, which must be of the delegate type of the event. When an `AddHandler` or `RemoveHandler` statement is executed (or a `Handles` clause automatically handles an event), the corresponding declaration will be called. The `RaiseEvent` declaration takes the same parameters as the event delegate and will be called when a `RaiseEvent` statement is executed. All of the declarations must be provided and are considered to be subroutines.

Note that `AddHandler`, `RemoveHandler` and `RaiseEvent` declarations have the same restriction on line placement that subroutines have. The beginning statement, end statement and block must all appear at the beginning of a logical line.

In addition to the member name added to the type's declaration space, a custom event declaration implicitly declares several other members. Given an event named `X`, the following members are added to the declaration space:

* A method named `add_X`, corresponding to the `AddHandler` declaration.

* A method named `remove_X`, corresponding to the `RemoveHandler` declaration.

* A method named `fire_X`, corresponding to the `RaiseEvent` declaration.

If a type attempts to declare a name that matches one of the above names, a compile-time error will result, and the implicit declarations are all ignored for the purposes of name binding. It is not possible to override or overload any of the introduced members, although it is possible to shadow them in derived types.

__Note.__ `Custom` is not a reserved word.

#### Custom events in WinRT assemblies

As of Microsoft Visual Basic 11.0, events declared in a file compiled with `/target:winmdobj`, or declared in an interface in such a file and then implemented elsewhere, are treated a little differently.

* External tools used to build the winmd will typically allow only certain delegate types such as `System.EventHandler(Of T)` or `System.TypedEventHandle(Of T, U)`, and will disallow others.

* The `XEvent` field has type `System.Runtime.InteropServices.WindowsRuntime.EventRegistrationTokenTable(Of T)` where `T` is the delegate type.

* The AddHandler accessor returns a `System.Runtime.InteropServices.WindowsRuntime.EventRegistrationToken`, and the RemoveHandler accessor takes a single parameter of the same type.

Here is an example of such a custom event.

```vb
Imports System.Runtime.InteropServices.WindowsRuntime

Public NotInheritable Class ClassInWinMD
    Private XEvent As EventRegistrationTokenTable(Of EventHandler(Of Integer))

    Public Custom Event X As EventHandler(Of Integer)
        AddHandler(handler As EventHandler(Of Integer))
            Return EventRegistrationTokenTable(Of EventHandler(Of Integer)).
                   GetOrCreateEventRegistrationTokenTable(XEvent).
                   AddEventHandler(handler)
        End AddHandler

        RemoveHandler(token As EventRegistrationToken)
            EventRegistrationTokenTable(Of EventHandler(Of Integer)).
                GetOrCreateEventRegistrationTokenTable(XEvent).
                RemoveEventHandler(token)
        End RemoveHandler

        RaiseEvent(sender As Object, i As Integer)
            Dim table = EventRegistrationTokenTable(Of EventHandler(Of Integer)).
                GetOrCreateEventRegistrationTokenTable(XEvent).
                InvocationList
            If table IsNot Nothing Then table(sender, i)
        End RaiseEvent
    End Event
End Class
```


## Constants

A *constant* is a constant value that is a member of a type.

```antlr
ConstantMemberDeclaration
    : Attributes? ConstantModifier* 'Const' ConstantDeclarators StatementTerminator
    ;

ConstantModifier
    : AccessModifier
    | 'Shadows'
    ;

ConstantDeclarators
    : ConstantDeclarator ( Comma ConstantDeclarator )*
    ;

ConstantDeclarator
    : Identifier ( 'As' TypeName )? Equals ConstantExpression StatementTerminator
    ;
```

Constants are implicitly shared. If the declaration contains an `As` clause, the clause specifies the type of the member introduced by the declaration. If the type is omitted then the type of the constant is inferred. The type of a constant may only be a primitive type or `Object`. If a constant is typed as `Object` and there is no type character, the real type of the constant will be the type of the constant expression. Otherwise, the type of the constant is the type of the constant's type character.

The following example shows a class named `Constants` that has two public constants:

```vb
Class Constants
    Public Const A As Integer = 1
    Public Const B As Integer = A + 1
End Class
```

Constants can be accessed through the class, as in the following example, which prints out the values of `Constants.A` and `Constants.B`.

```vb
Module Test
    Sub Main()
        Console.WriteLine(Constants.A & ", " & Constants.B)
    End Sub 
End Module
```

A constant declaration that declares multiple constants is equivalent to multiple declarations of single constants. The following example declares three constants in one declaration statement.

```vb
Class A
    Protected Const x As Integer = 1, y As Long = 2, z As Short = 3
End Class
```

This declaration is equivalent to the following:

```vb
Class A
    Protected Const x As Integer = 1
    Protected Const y As Long = 2
    Protected Const z As Short = 3
End Class
```

The accessibility domain of the type of the constant must be the same as or a superset of the accessibility domain of the constant itself. The constant expression must yield a value of the constant's type or of a type that is implicitly convertible to the constant's type. The constant expression may not be circular; that is, a constant may not be defined in terms of itself.

The compiler automatically evaluates the constant declarations in the appropriate order. In the following example, the compiler first evaluates `Y`, then `Z`, and finally `X`, producing the values 10, 11, and 12, respectively.

```vb
Class A
    Public Const X As Integer = B.Z + 1
    Public Const Y As Integer = 10
End Class

Class B
    Public Const Z As Integer = A.Y + 1
End Class
```

When a symbolic name for a constant value is desired, but the type of the value is not permitted in a constant declaration or when the value cannot be computed at compile time by a constant expression, a read-only variable may be used instead.


## Instance and Shared Variables

An instance or shared variable is a member of a type that can store information.

```antlr
VariableMemberDeclaration
    : Attributes? VariableModifier+ VariableDeclarators StatementTerminator
    ;

VariableModifier
    : AccessModifier
    | 'Shadows'
    | 'Shared'
    | 'ReadOnly'
    | 'WithEvents'
    | 'Dim'
    ;

VariableDeclarators
    : VariableDeclarator ( Comma VariableDeclarator )*
    ;

VariableDeclarator
    : VariableIdentifiers 'As' ObjectCreationExpression
    | VariableIdentifiers ( 'As' TypeName )? ( Equals Expression )?
    ;

VariableIdentifiers
    : VariableIdentifier ( Comma VariableIdentifier )*
    ;

VariableIdentifier
    : Identifier IdentifierModifiers
    ;
```

The `Dim` modifier must be specified if no modifiers are specified, but may be omitted otherwise. A single variable declaration may include multiple variable declarators; each variable declarator introduces a new instance or shared member.

If an initializer is specified, only one instance or shared variable may be declared by the variable declarator:

```vb
Class Test
    Dim a, b, c, d As Integer = 10  ' Invalid: multiple initialization
End Class
```

This restriction does not apply to object initializers:

```vb
Class Test
    Dim a, b, c, d As New Collection() ' OK
End Class
```

A variable declared with the `Shared` modifier is a *shared variable*. A shared variable identifies exactly one storage location regardless of the number of instances of the type that are created. A shared variable comes into existence when a program begins executing, and ceases to exist when the program terminates.

A shared variable is shared only among instances of a particular closed generic type. For example, the program:

```vb
Class C(Of V) 
    Shared InstanceCount As Integer = 0

    Public Sub New()  
        InstanceCount += 1 
    End Sub

    Public Shared ReadOnly Property Count() As Integer 
        Get
            Return InstanceCount
        End Get
    End Property
End Class

Class Application 
    Shared Sub Main() 
        Dim x1 As New C(Of Integer)()
        Console.WriteLine(C(Of Integer).Count)

        Dim x2 As New C(Of Double)() 
        Console.WriteLine(C(Of Integer).Count)

        Dim x3 As New C(Of Integer)() 
        Console.WriteLine(C(Of Integer).Count)
    End Sub
End Class
```

Prints out:

```
1
1
2
```

A variable declared without the `Shared` modifier is called an *instance variable*. Every instance of a class contains a separate copy of all instance variables of the class. An instance variable of a reference type comes into existence when a new instance of that type is created, and ceases to exist when there are no references to that instance and the `Finalize` method has executed. An instance variable of a value type has exactly the same lifetime as the variable to which it belongs. In other words, when a variable of a value type comes into existence or ceases to exist, so does the instance variable of the value type.

If the declarator contains an `As` clause, the clause specifies the type of the members introduced by the declaration. If the type is omitted and strict semantics are being used, a compile-time error occurs. Otherwise the type of the members is implicitly `Object` or the type of the members' type character.

__Note.__ There is no ambiguity in the syntax: if a declarator omits a type, it will always use the type of a following declarator.

The accessibility domain of an instance or shared variable's type or array element type must be the same as or a superset of the accessibility domain of the instance or shared variable itself.

The following example shows a `Color` class that has internal instance variables named `redPart`, `greenPart`, and `bluePart`:

```vb
Class Color
    Friend redPart As Short
    Friend bluePart As Short
    Friend greenPart As Short

    Public Sub New(red As Short, blue As Short, green As Short)
        redPart = red
        bluePart = blue
        greenPart = green
    End Sub
End Class
```


### Read-Only Variables

When an instance or shared variable declaration includes a `ReadOnly` modifier, assignments to the variables introduced by the declaration may only occur as part of the declaration or in a constructor in the same class. Specifically, assignments to a read-only instance or shared variable are permitted only in the following situations:

* In the variable declaration that introduces the instance or shared variable (by including a variable initializer in the declaration).

* For an instance variable, in the instance constructors of the class that contains the variable declaration. The instance variable can only be accessed in an unqualified manner or through `Me` or `MyClass`.

* For a shared variable, in the shared constructor of the class that contains the shared variable declaration.

A shared read-only variable is useful when a symbolic name for a constant value is desired, but when the type of the value is not permitted in a constant declaration, or when the value cannot be computed at compile time by a constant expression.

An example of the first such application follows, in which color shared variables are declared `ReadOnly` to prevent them from being changed by other programs:

```vb
Class Color
    Friend redPart As Short
    Friend bluePart As Short
    Friend greenPart As Short

    Public Sub New(red As Short, blue As Short, green As Short)
        redPart = red
        bluePart = blue
        greenPart = green
    End Sub 

    Public Shared ReadOnly Red As Color = New Color(&HFF, 0, 0)
    Public Shared ReadOnly Blue As Color = New Color(0, &HFF, 0)
    Public Shared ReadOnly Green As Color = New Color(0, 0, &HFF)
    Public Shared ReadOnly White As Color = New Color(&HFF, &HFF, &HFF)
End Class
```

Constants and read-only shared variables have different semantics. When an expression references a constant, the value of the constant is obtained at compile time, but when an expression references a read-only shared variable, the value of the shared variable is not obtained until run time. Consider the following application, which consists of two separate programs.

file1.vb:

```vb
Namespace Program1
    Public Class Utils
        Public Shared ReadOnly X As Integer = 1
    End Class
End Namespace
```

file2.vb:

```vb
Namespace Program2
    Module Test
        Sub Main()
            Console.WriteLine(Program1.Utils.X)
        End Sub
    End Module
End Namespace
```

The namespaces `Program1` and `Program2` denote two programs that are compiled separately. Because variable `Program1.Utils.X` is declared as `Shared ReadOnly`, the value output by the `Console.WriteLine` statement is not known at compile time, but rather is obtained at run time. Thus, if the value of `X` is changed and `Program1` is recompiled, the `Console.WriteLine` statement will output the new value even if `Program2` is not recompiled. However, if `X` had been a constant, the value of `X` would have been obtained at the time `Program2` was compiled, and would have remained unaffected by changes in `Program1` until `Program2` was recompiled.

### WithEvents Variables

A type can declare that it handles some set of events raised by one of its instance or shared variables by declaring the instance or shared variable that raises the events with the `WithEvents` modifier. For example:

```vb
Class Raiser
    Public Event E1()

    Public Sub Raise()
        RaiseEvent E1
    End Sub
End Class

Module Test
    Private WithEvents x As Raiser

    Private Sub E1Handler() Handles x.E1
        Console.WriteLine("Raised")
    End Sub

    Public Sub Main()
        x = New Raiser()
    End Sub
End Module
```

In this example, the method `E1Handler` handles the event `E1` that is raised by the instance of the type `Raiser` stored in the instance variable `x`.

The `WithEvents` modifier causes the variable to be renamed with a leading underscore and replaced with a property of the same name that does the event hookup. For example, if the variable's name is `F`, it is renamed to `_F` and a property `F` is implicitly declared. If there is a collision between the variable's new name and another declaration, a compile-time error will be reported. Any attributes applied to the variable are carried over to the renamed variable.

The implicit property created by a `WithEvents` declaration takes care of hooking and unhooking the relevant event handlers. When a value is assigned to the variable, the property first calls the `remove` method for the event on the instance currently in the variable (unhooking the existing event handler, if any). Next the assignment is made, and the property calls the `add` method for the event on the new instance in the variable (hooking up the new event handler). The following code is equivalent to the code above for the standard module `Test`:

```vb
Module Test
    Private _x As Raiser

    Public Property x() As Raiser
        Get
            Return _x
        End Get

        Set (Value As Raiser)
            ' Unhook any existing handlers.
            If _x IsNot Nothing Then
                RemoveHandler _x.E1, AddressOf E1Handler
            End If

            ' Change value.
            _x = Value

            ' Hook-up new handlers.
            If _x IsNot Nothing Then
                AddHandler _x.E1, AddressOf E1Handler
            End If
        End Set
    End Property

    Sub E1Handler()
        Console.WriteLine("Raised")
    End Sub

    Sub Main()
        x = New Raiser()
    End Sub
End Module
```

It is not valid to declare an instance or shared variable as `WithEvents` if the variable is typed as a structure. In addition, `WithEvents` may not be specified in a structure, and `WithEvents` and `ReadOnly` cannot be combined.

### Variable Initializers

Instance and shared variable declarations in classes and instance variable declarations (but not shared variable declarations) in structures may include variable initializers. For `Shared` variables, variable initializers correspond to assignment statements that are executed after the program begins, but before the `Shared` variable is first referenced. For instance variables, variable initializers correspond to assignment statements that are executed when an instance of the class is created. Structures cannot have instance variable initializers because their parameterless constructors cannot be modified.

Consider the following example:

```vb
Class Test
    Public Shared x As Double = Math.Sqrt(2.0)
    Public i As Integer = 100
    Public s As String = "Hello"
End Class

Module TestModule
    Sub Main()
        Dim a As New Test()

        Console.WriteLine("x = " & Test.x & ", i = " & a.i & ", s = " & a.s)
    End Sub
End Module
```

The example produces the following output:

```
x = 1.4142135623731, i = 100, s = Hello
```

An assignment to `x` occurs when the class is loaded, and assignments to `i` and `s` occur when a new instance of the class is created.

It is useful to think of variable initializers as assignment statements that are automatically inserted in the block of the type's constructor. The following example contains several instance variable initializers.

```vb
Class A
    Private x As Integer = 1
    Private y As Integer = -1
    Private count As Integer

    Public Sub New()
        count = 0
    End Sub

    Public Sub New(n As Integer)
        count = n
    End Sub
End Class

Class B
    Inherits A

    Private sqrt2 As Double = Math.Sqrt(2.0)
    Private items As ArrayList = New ArrayList(100)
    Private max As Integer

    Public Sub New()
        Me.New(100)
        items.Add("default")
    End Sub

    Public Sub New(n As Integer)
        MyBase.New(n - 1)
        max = n
    End Sub
End Class
```

The example corresponds to the code shown below, where each comment indicates an automatically inserted statement.

```vb
Class A
    Private x, y, count As Integer

    Public Sub New()
        MyBase.New ' Invoke object() constructor.
        x = 1 ' This is a variable initializer.
        y = -1 ' This is a variable initializer.
        count = 0
    End Sub

    Public Sub New(n As Integer)
        MyBase.New ' Invoke object() constructor. 
        x = 1 ' This is a variable initializer.
        y = - 1 ' This is a variable initializer.
        count = n
    End Sub
End Class

Class B
    Inherits A

    Private sqrt2 As Double
    Private items As ArrayList
    Private max As Integer

    Public Sub New()
        Me.New(100) 
        items.Add("default")
    End Sub

    Public Sub New(n As Integer)
        MyBase.New(n - 1) 
        sqrt2 = Math.Sqrt(2.0) ' This is a variable initializer.
        items = New ArrayList(100) ' This is a variable initializer.
        max = n
    End Sub
End Class
```

All variables are initialized to the default value of their type before any variable initializers are executed. For example:

```vb
Class Test
    Public Shared b As Boolean
    Public i As Integer
End Class

Module TestModule
    Sub Main()
        Dim t As New Test()
        Console.WriteLine("b = " & Test.b & ", i = " & t.i)
    End Sub
End Module
```

Because `b` is automatically initialized to its default value when the class is loaded and `i` is automatically initialized to its default value when an instance of the class is created, the preceding code produces the following output:

```
b = False, i = 0
```

Each variable initializer must yield a value of the variable's type or of a type that is implicitly convertible to the variable's type. A variable initializer may be circular or refer to a variable that will be initialized after it, in which case the value of the referenced variable is its default value for the purposes of the initializer. Such an initializer is of dubious value.

There are three forms of variable initializers: regular initializers, array-size initializers, and object initializers. The first two forms appear after an equal sign that follows the type name, the latter two are part of the declaration itself. Only one form of initializer may be used on any particular declaration.

#### Regular Initializers

A *regular initializer* is an expression that is implicitly convertible to the type of the variable. It appears after an equal sign that follows the type name and must be classified as a value. For example:

```vb
Module Test
    Dim x As Integer = 10
    Dim y As Integer = 20

    Sub Main()
        Console.WriteLine("x = " & x & ", y = " & y)
    End Sub
End Module
```

This program produces the output:

```vb
x = 10, y = 20
```

If a variable declaration has a regular initializer, then only a single variable can be declared at a time. For example:

```vb
Module Test
    Sub Main()
        ' OK, only one variable declared at a time.
        Dim x As Integer = 10, y As Integer = 20

        ' Error: Can't initialize multiple variables at once.
        Dim a, b As Integer = 10
    End Sub
End Module
```

#### Object Initializers

An *object initializer* is specified using an object creation expression in the place of the type name. An object initializer is equivalent to a regular initializer assigning the result of the object creation expression to the variable. So

```vb
Module TestModule
    Sub Main()
        Dim x As New Test(10)
    End Sub
End Module
```

is equivalent to

```vb
Module TestModule
    Sub Main()
        Dim x As Test = New Test(10)
    End Sub
End Module
```

The parenthesis in an object initializer is always interpreted as the argument list for the constructor and never as array type modifiers. A variable name with an object initializer cannot have an array type modifier or a nullable type modifier.

#### Array-Size Initializers

An *array-size initializer* is a modifier on the name of the variable that gives a set of dimension upper bounds denoted by expressions.

```antlr
ArraySizeInitializationModifier
    : OpenParenthesis BoundList CloseParenthesis ArrayTypeModifiers?
    ;

BoundList
    : Bound ( Comma Bound )*
    ;

Bound
    : Expression
    | '0' 'To' Expression
    ;
```

The upper bound expressions must be classified as values and must be implicitly convertible to `Integer`. The set of upper bounds is equivalent to a variable initializer of an array-creation expression with the given upper bounds. The number of dimensions of the array type is inferred from the array size initializer. So

```vb
Module Test
    Sub Main()
        Dim x(5, 10) As Integer
    End Sub
End Module
```

is equivalent to

```vb
Module Test
    Sub Main()
        Dim x As Integer(,) = New Integer(5, 10) {}
    End Sub
End Module
```

All upper bounds must be equal to or greater than -1, and all dimensions must have an upper bound specified. If the element type of the array being initialized is itself an array type, the array-type modifiers go to the right of the array-size initializer. For example

```vb
Module Test
    Sub Main()
        Dim x(5,10)(,,) As Integer
    End Sub
End Module
```

declares a local variable `x` whose type is a two-dimensional array of three-dimensional arrays of `Integer`, initialized to an array with bounds of `0..5` in the first dimension and `0..10` in the second dimension. It is not possible to use an array size initializer to initialize the elements of a variable whose type is an array of arrays.

A variable declaration with an array-size initializer cannot have an array type modifier on its type or a regular initializer.


### System.MarshalByRefObject Classes

Classes that derive from the class `System.MarshalByRefObject` are marshaled across context boundaries using proxies (that is, by reference) rather than through copying (that is, by value). This means that an instance of such a class may not be a true instance but instead may just be a stub that marshals variable accesses and method calls across a context boundary.

As a result, it is not possible to create a reference to the storage location of variables defined on such classes. This means that variables typed as classes derived from `System.MarshalByRefObject` cannot be passed to reference parameters, and methods and variables of variables typed as value types may not be accessed. Instead, Visual Basic treats variables defined on such classes as if they were properties (since the restrictions are the same on properties).

There is one exception to this rule: a member implicitly or explicitly qualified with `Me` is exempt from the above restrictions, because `Me` is always guaranteed to be an actual object, not a proxy.

## Properties

*Properties* are a natural extension of variables; both are named members with associated types, and the syntax for accessing variables and properties is the same. Unlike variables, however, properties do not denote storage locations. Instead, properties have *accessors*, which specify the statements to execute in order to read or write their values.

Properties are defined with property declarations. The first part of a property declaration resembles a field declaration. The second part includes a `Get` accessor and/or a `Set` accessor.

```antlr
PropertyMemberDeclaration
    : RegularPropertyMemberDeclaration
    | MustOverridePropertyMemberDeclaration
    | AutoPropertyMemberDeclaration
    ;

PropertySignature
    : 'Property'
      Identifier ( OpenParenthesis ParameterList? CloseParenthesis )?
      ( 'As' Attributes? TypeName )?
    ;

RegularPropertyMemberDeclaration
    : Attributes? PropertyModifier* PropertySignature
      ImplementsClause? LineTerminator
      PropertyAccessorDeclaration+
      'End' 'Property' StatementTerminator
    ;

MustOverridePropertyMemberDeclaration
    : Attributes? MustOverridePropertyModifier+ PropertySignature
      ImplementsClause? StatementTerminator
    ;

AutoPropertyMemberDeclaration
    : Attributes? AutoPropertyModifier* 'Property' Identifier
      ( OpenParenthesis ParameterList? CloseParenthesis )?
      ( 'As' Attributes? TypeName )? ( Equals Expression )?
      ImplementsClause? LineTerminator
    | Attributes? AutoPropertyModifier* 'Property' Identifier
      ( OpenParenthesis ParameterList? CloseParenthesis )?
      'As' Attributes? 'New'
      ( NonArrayTypeName ( OpenParenthesis ArgumentList? CloseParenthesis )? )?
      ObjectCreationExpressionInitializer?
      ImplementsClause? LineTerminator
    ;

InterfacePropertyMemberDeclaration
    : Attributes? InterfacePropertyModifier* PropertySignature StatementTerminator
    ;

AutoPropertyModifier
    : AccessModifier
    | 'Shadows'
    | 'Shared'
    | 'Overridable'
    | 'NotOverridable'
    | 'Overrides'
    | 'Overloads'
    ;

PropertyModifier
    : AutoPropertyModifier
    | 'Default'
    | 'ReadOnly'
    | 'WriteOnly'
    | 'Iterator'
    ;

MustOverridePropertyModifier
    : PropertyModifier
    | 'MustOverride'
    ;

InterfacePropertyModifier
    : 'Shadows'
    | 'Overloads'
    | 'Default'
    | 'ReadOnly'
    | 'WriteOnly'
    ;

PropertyAccessorDeclaration
    : PropertyGetDeclaration
    | PropertySetDeclaration
    ;
```

In the example below, the `Button` class defines a `Caption` property.

```vb
Public Class Button
    Private captionValue As String

    Public Property Caption() As String
        Get
            Return captionValue
        End Get

        Set (Value As String)
            captionValue = value
            Repaint()
        End Set
    End Property

    ...
End Class
```

Based on the `Button` class above, the following is an example of use of the `Caption` property:

```vb
Dim okButton As Button = New Button()

okButton.Caption = "OK" ' Invokes Set accessor.
Dim s As String = okButton.Caption ' Invokes Get accessor.
```

Here, the `Set` accessor is invoked by assigning a value to the property, and the `Get` accessor is invoked by referencing the property in an expression.

If no type is specified for a property and strict semantics are being used, a compile-time error occurs; otherwise the type of the property is implicitly `Object` or the type of the property's type character. A property declaration may contain either a `Get` accessor, which retrieves the value of the property, a `Set` accessor, which stores the value of the property, or both. Because a property implicitly declares methods, a property may be declared with the same modifiers as a method. If the property is defined in an interface or defined with the `MustOverride` modifier, the property body and the `End` construct must be omitted; otherwise, a compile-time error occurs.

The index parameter list makes up the signature of the property, so properties may be overloaded on index parameters but not on the type of the property. The index parameter list is the same as for a regular method. However, none of the parameters may be modified with the `ByRef` modifier and none of them may be named `Value` (which is reserved for the implicit value parameter in the `Set` accessor).

A property may be declared as follows:

* If the property specifies no property type modifier, the property must have both a `Get` accessor and a `Set` accessor. The property is said to be a read-write property.

* If the property specifies the `ReadOnly` modifier, the property must have a `Get` accessor and may not have a `Set` accessor. The property is said to be read-only property. It is a compile-time error for a read-only property to be the target of an assignment.

* If the property specifies the `WriteOnly` modifier, the property must have a `Set` accessor and may not have a `Get` accessor. The property is said to be write-only property. It is a compile-time error to reference a write-only property in an expression except as the target of an assignment or as an argument to a method.

The `Get` and `Set` accessors of a property are not distinct members, and it is not possible to declare the accessors of a property separately. The following example does not declare a single read-write property. Rather, it declares two properties with the same name, one read-only and one write-only:

```vb
Class A
    Private nameValue As String

    ' Error, contains a duplicate member name.
    Public ReadOnly Property Name() As String 
        Get
            Return nameValue
        End Get
    End Property

    ' Error, contains a duplicate member name.
    Public WriteOnly Property Name() As String 
        Set (Value As String)
            nameValue = value
        End Set
    End Property
End Class
```

Since two members declared in the same class cannot have the same name, the example causes a compile-time error.

By default, the accessibility of a property's `Get` and `Set` accessors is the same as the accessibility of the property itself. However, the `Get` and `Set` accessors can also specify accessibility separately from the property. In that case, the accessibility of an accessor must be more restrictive than the accessibility of the property and only one accessor can have a different accessibility level from the property. Access types are considered more or less restrictive as follows:

* `Private` is more restrictive than `Public`, `Protected Friend`, `Protected`, or `Friend`.

* `Friend` is more restrictive than `Protected Friend` or `Public`.

* `Protected` is more restrictive than `Protected Friend` or `Public`.

* `Protected Friend` is more restrictive than `Public`.

When one of a property's accessors is accessible but the other one is not, the property is treated as if it was read-only or write-only. For example:

```vb
Class A
    Public Property P() As Integer
        Get
            ...
        End Get

        Private Set (Value As Integer)
            ...
        End Set
    End Property
End Class

Module Test
    Sub Main()
        Dim a As A = New A()

        ' Error: A.P is read-only in this context.
        a.P = 10
    End Sub
End Module
```

When a derived type shadows a property, the derived property hides the shadowed property with respect to both reading and writing. In the following example, the `P` property in `B` hides the `P` property in `A` with respect to both reading and writing:

```vb
Class A
    Public WriteOnly Property P() As Integer
        Set (Value As Integer)
        End Set
    End Property
End Class

Class B
    Inherits A

    Public Shadows ReadOnly Property P() As Integer
       Get
       End Get
    End Property
End Class

Module Test
    Sub Main()
        Dim x As B = New B

        B.P = 10     ' Error, B.P is read-only.
    End Sub
End Module
```

The accessibility domain of the return type or parameter types must be the same as or a superset of the accessibility domain of the property itself. A property may only have one `Set` accessor and one `Get` accessor.

Except for differences in declaration and invocation syntax, `Overridable`, `NotOverridable`, `Overrides`, `MustOverride`, and `MustInherit` properties behave exactly like `Overridable`, `NotOverridable`, `Overrides`, `MustOverride`, and `MustInherit` methods. When a property is overridden, the overriding property must be of the same type (read-write, read-only, write-only). An `Overridable` property cannot contain a `Private` accessor.

In the following example `X` is an `Overridable` read-only property, `Y` is an `Overridable` read-write property, and `Z` is a `MustOverride` read-write property.

```vb
MustInherit Class A
    Private _y As Integer

    Public Overridable ReadOnly Property X() As Integer
        Get
            Return 0
        End Get
    End Property

    Public Overridable Property Y() As Integer
        Get
            Return _y
         End Get
        Set (Value As Integer)
            _y = value
        End Set
    End Property

    Public MustOverride Property Z() As Integer
End Class
```

Because `Z` is `MustOverride`, the containing class `A` must be declared `MustInherit`.

By contrast, a class that derives from class `A` is shown below:

```vb
Class B
    Inherits A

    Private _z As Integer

    Public Overrides ReadOnly Property X() As Integer
        Get
            Return MyBase.X + 1
        End Get
    End Property

    Public Overrides Property Y() As Integer
        Get
            Return MyBase.Y
        End Get
        Set (Value As Integer)
            If value < 0 Then
                MyBase.Y = 0
            Else
                MyBase.Y = Value
            End If
        End Set
    End Property

    Public Overrides Property Z() As Integer
        Get
            Return _z
        End Get
        Set (Value As Integer)
            _z = Value
        End Set
    End Property
End Class
```

Here, the declarations of properties `X`,`Y`, and `Z` override the base properties. Each property declaration exactly matches the accessibility modifiers, type, and name of the corresponding inherited property. The `Get` accessor of property `X` and the `Set` accessor of property `Y` use the `MyBase` keyword to access the inherited properties. The declaration of property `Z` overrides the `MustOverride` property -- thus, there are no outstanding `MustOverride` members in class `B`, and `B` is permitted to be a regular class.

Properties can be used to delay initialization of a resource until the moment it is first referenced. For example:

```vb
Imports System.IO

Public Class ConsoleStreams
    Private Shared reader As TextReader
    Private Shared writer As TextWriter
    Private Shared errors As TextWriter

    Public Shared ReadOnly Property [In]() As TextReader
        Get
            If reader Is Nothing Then
                reader = Console.In
            End If
            Return reader
        End Get
    End Property

    Public Shared ReadOnly Property Out() As TextWriter
        Get
            If writer Is Nothing Then
                writer = Console.Out
            End If
            Return writer
        End Get
    End Property

    Public Shared ReadOnly Property [Error]() As TextWriter
        Get
            If errors Is Nothing Then
                errors = Console.Error
            End If
            Return errors
        End Get
    End Property
End Class
```

The `ConsoleStreams` class contains three properties, `In`, `Out`, and `Error`, that represent the standard input, output, and error devices, respectively. By exposing these members as properties, the `ConsoleStreams` class can delay their initialization until they are actually used. For example, upon first referencing the `Out` property, as in `ConsoleStreams.Out.WriteLine("hello, world")`, the underlying `TextWriter` for the output device is initialized. But if the application makes no reference to the `In` and `Error` properties, then no objects are created for those devices.


### Get Accessor Declarations

A `Get` accessor (getter) is declared by using a property `Get` declaration. A property `Get` declaration consists of the keyword `Get` followed by a statement block. Given a property named `P`, a `Get` accessor declaration implicitly declares a method with the name `get_P` with the same modifiers, type, and parameter list as the property. If the type contains a declaration with that name, a compile-time error results, but the implicit declaration is ignored for the purposes of name binding.

A special local variable, which is implicitly declared in the `Get` accessor body's declaration space with the same name as the property, represents the return value of the property. The local variable has special name resolution semantics when used in expressions. If the local variable is used in a context that expects an expression that is classified as a method group, such as an invocation expression, then the name resolves to the function rather than to the local variable. For example:

```vb
ReadOnly Property F(i As Integer) As Integer
    Get
        If i = 0 Then
            F = 1    ' Sets the return value.
        Else
            F = F(i - 1) ' Recursive call.
        End If
    End Get
End Property
```

The use of parentheses can cause ambiguous situations (such as `F(1)` where `F` is a property whose type is a one-dimensional array). In all ambiguous situations, the name resolves to the property rather than the local variable. For example:

```vb
ReadOnly Property F(i As Integer) As Integer()
    Get
        If i = 0 Then
            F = new Integer(2) { 1, 2, 3 }
        Else
            F = F(i - 1) ' Recursive call, not index.
        End If
    End Get
End Property
```

When control flow leaves the `Get` accessor body, the value of the local variable is passed back to the invocation expression. Because invoking a `Get` accessor is conceptually equivalent to reading the value of a variable, it is considered bad programming style for `Get` accessors to have observable side effects, as illustrated in the following example:

```vb
Class Counter
    Private Value As Integer

    Public ReadOnly Property NextValue() As Integer
        Get
            Value += 1
            Return Value
        End Get
    End Property
End Class
```

The value of the `NextValue` property depends on the number of times the property has previously been accessed. Thus, accessing the property produces an observable side effect, and the property should instead be implemented as a method.

The "no side effects" convention for `Get` accessors does not mean that `Get` accessors should always be written to simply return values stored in variables. Indeed, `Get` accessors often compute the value of a property by accessing multiple variables or invoking methods. However, a properly designed `Get` accessor performs no actions that cause observable changes in the state of the object.

__Note.__ `Get` accessors have the same restriction on line placement that subroutines have. The beginning statement, end statement and block must all appear at the beginning of a logical line.

```antlr
PropertyGetDeclaration
    : Attributes? AccessModifier? 'Get' LineTerminator
      Block?
      'End' 'Get' StatementTerminator
    ;
```

### Set Accessor Declarations

A `Set` accessor (setter) is declared by using a property set declaration. A property set declaration consists of the keyword `Set`, an optional parameter list, and a statement block. Given a property named `P`, a setter declaration implicitly declares a method with the name `set_P` with the same modifiers and parameter list as the property. If the type contains a declaration with that name, a compile-time error results, but the implicit declaration is ignored for the purposes of name binding.

If a parameter list is specified, it must have one member, that member must have no modifiers except `ByVal`, and its type must be the same as the type of the property. The parameter represents the property value being set. If the parameter is omitted, a parameter named `Value` is implicitly declared.

__Note.__ `Set` accessors have the same restriction on line placement that subroutines have. The beginning statement, end statement and block must all appear at the beginning of a logical line.

```antlr
PropertySetDeclaration
    : Attributes? AccessModifier? 'Set'
      ( OpenParenthesis ParameterList? CloseParenthesis )? LineTerminator
      Block?
      'End' 'Set' StatementTerminator
    ;
```

### Default Properties

A property that specifies the modifier `Default` is called a *default property*. Any type that allows properties may have a default property, including interfaces. The default property may be referenced without having to qualify the instance with the name of the property. Thus, given a class

```vb
Class Test
    Public Default ReadOnly Property Item(i As Integer) As Integer
        Get
            Return i
        End Get
    End Property
End Class
```

the code

```vb
Module TestModule
    Sub Main()
        Dim x As Test = New Test()
        Dim y As Integer

        y = x(10)
    End Sub
End Module
```

is equivalent to

```vb
Module TestModule
    Sub Main()
        Dim x As Test = New Test()
        Dim y As Integer

        y = x.Item(10)
    End Sub
End Module
```

Once a property is declared `Default`, all of the properties overloaded on that name in the inheritance hierarchy become the default property, whether they have been declared `Default` or not. Declaring a property `Default` in a derived class when the base class declared a default property by another name does not require any other modifiers such as `Shadows` or `Overrides`. This is because the default property has no identity or signature and so cannot be shadowed or overloaded. For example:

```vb
Class Base
    Public ReadOnly Default Property Item(i As Integer) As Integer
        Get
            Console.WriteLine("Base = " & i)
        End Get
    End Property
End Class

Class Derived
    Inherits Base

    ' This hides Item, but does not change the default property.
    Public Shadows ReadOnly Property Item(i As Integer) As Integer
        Get
            Console.WriteLine("Derived = " & i)
        End Get
    End Property
End Class

Class MoreDerived
    Inherits Derived

    ' This declares a new default property, but not Item.
    ' This does not need to be declared Shadows
    Public ReadOnly Default Property Value(i As Integer) As Integer
        Get
            Console.WriteLine("MoreDerived = " & i)
        End Get
    End Property
End Class

Module Test
    Sub Main()
        Dim x As MoreDerived = New MoreDerived()
        Dim y As Integer
        Dim z As Derived = x

        y = x(10)        ' Calls MoreDerived.Value.
        y = x.Item(10)   ' Calls Derived.Item
        y = z(10)        ' Calls Base.Item
    End Sub
End Module
```

This program will produce the output:

```
MoreDerived = 10
Derived = 10
Base = 10
```

All default properties declared within a type must have the same name and, for clarity, must specify the `Default` modifier. Because a default property with no index parameters would cause an ambiguous situation when assigning instances of the containing class, default properties must have index parameters. Furthermore, if one property overloaded on a particular name includes the `Default` modifier, all properties overloaded on that name must specify it. Default properties may not be `Shared`, and at least one accessor of the property must not be `Private`.

### Automatically Implemented Properties

If a property omits declaration of any accessors, an implementation of the property will be automatically supplied unless the property is declared in an interface or is declared `MustOverride`. Only read/write properties with no arguments can be automatically implemented; otherwise, a compile-time error occurs.

An automatically implemented property `x`, even one overriding another property, introduces a private local variable `_x` with the same type as the property. If there is a collision between the local variable's name and another declaration, a compile-time error will be reported. The automatically implemented property's `Get` accessor returns the value of the local and the property's `Set` accessor that sets the value of the local. For example, the declaration:

```vb
Public Property x() As Integer
```

is roughly equivalent to:

```vb
Private _x As Integer
Public Property x() As Integer
    Get
        Return _x
    End Get
    Set (value As Integer)
        _x = value
    End Set
End Property
```

As with variable declarations, an automatically implemented property can include an initializer. For example:

```vb
Public Property x() As Integer = 10
Public Shared Property y() As New Customer() With { .Name = "Bob" }
```

__Note.__ When an automatically implemented property is initialized, it is initialized through the property, not the underlying field. This is so overriding properties can intercept the initialization if they need to.

Array initializers are allowed on automatically implemented properties, except that there is no way to specify the array bounds explicitly.  For example:

```vb
' Valid
Property x As Integer() = {1, 2, 3}
Property y As Integer(,) = {{1, 2, 3}, {12, 13, 14}, {11, 10, 9}}

' Invalid
Property x4(5) As Short
```

### Iterator Properties

An *iterator property* is a property with the `Iterator` modifier. It is used for the same reason an iterator method (Section [Iterator Methods](statements.md#iterator-methods)) is used -- as a convenient way to generate a sequence, one which can be consumed by the `For Each` statement. The `Get` accessor of an iterator property is interpreted in the same way as an iterator method.

An iterator property must have an explicit `Get` accessor, and its type must be `IEnumerator`, or `IEnumerable`, or `IEnumerator(Of T)` or `IEnumerable(Of T)` for some `T`.

Here is an example of an iterator property:

```vb
Class Family
    Property Daughters As New List(Of String) From {"Beth", "Diane"}
    Property Sons As New List(Of String) From {"Abe", "Carl"}

    ReadOnly Iterator Property Children As IEnumerable(Of String)
        Get
            For Each name In Daughters : Yield name : Next
            For Each name In Sons : Yield name : Next
        End Get
    End Property
End Class

Module Module1
    Sub Main()
        Dim x As New Family
        For Each c In x.Children
            Console.WriteLine(c) ' prints Beth, Diane, Abe, Carl
        Next
    End Sub
End Module
```

## Operators

*Operators* are methods that define the meaning of an existing Visual Basic operator for the containing class. When the operator is applied to the class in an expression, the operator is compiled into a call to the operator method defined in the class. Defining an operator for a class is also known as *overloading* the operator.

```antlr
OperatorDeclaration
    : Attributes? OperatorModifier* 'Operator' OverloadableOperator
      OpenParenthesis ParameterList CloseParenthesis
      ( 'As' Attributes? TypeName )? LineTerminator
      Block?
      'End' 'Operator' StatementTerminator
    ;

OperatorModifier
    : 'Public' | 'Shared' | 'Overloads' | 'Shadows' | 'Widening' | 'Narrowing'
    ;

OverloadableOperator
    : '+' | '-' | '*' | '/' | '\\' | '&' | 'Like' | 'Mod' | 'And' | 'Or' | 'Xor'
    | '^' | '<' '<' | '>' '>' | '=' | '<' '>' | '>' | '<' | '>' '=' | '<' '='
    | 'Not' | 'IsTrue' | 'IsFalse' | 'CType'
    ;
```

It is not possible to overload an operator that already exists; in practice, this primarily applies to conversion operators. For example, it is not possible to overload the conversion from a derived class to a base class:

```vb
Class Base
End Class

Class Derived
    ' Cannot redefine conversion from Derived to Base,
    ' conversion will be ignored.
    Public Shared Widening Operator CType(s As Derived) As Base
        ...
    End Operator
End Class
```

Operators can also be overloaded in the common sense of the word:

```vb
Class Base
    Public Shared Widening Operator CType(b As Base) As Integer
        ...
    End Operator

    Public Shared Narrowing Operator CType(i As Integer) As Base
        ...
    End Operator
End Class
```

Operator declarations do not explicitly add names to the containing type's declaration space; however they do implicitly declare a corresponding method starting with the characters "op_". The following sections list the corresponding method names with each operator.

There are three classes of operators that can be defined: unary operators, binary operators and conversion operators. All operator declarations share certain restrictions:

* Operator declarations must always be `Public` and `Shared`. The `Public` modifier can be omitted in contexts where the modifier will be assumed.

* The parameters of an operator cannot be declared `ByRef`, `Optional` or `ParamArray`.

* The type of at least one of the operands or the return value must be the type that contains the operator.

* There is no function return variable defined for operators. Therefore, the `Return` statement must be used to return values from an operator body.

The only exception to these restrictions applies to nullable value types. Since nullable value types do not have an actual type definition, a value type can declare user-defined operators for the nullable version of the type. When determining whether a type can declare a particular user-defined operator, the `?` modifiers are first dropped off of all of the types involved in the declaration for the purposes of validity checking. This relaxation does not apply to the return type of the `IsTrue` and `IsFalse` operators; they must still return `Boolean`, not `Boolean?`.

The precedence and associativity of an operator cannot be modified by an operator declaration.

__Note.__ Operators have the same restriction on line placement that subroutines have. The beginning statement, end statement and block must all appear at the beginning of a logical line.


### Unary Operators

The following unary operators can be overloaded:

* The unary plus operator `+` (corresponding method: `op_UnaryPlus`)

* The unary minus operator `-` (corresponding method: `op_UnaryNegation`)

* The logical `Not` operator (corresponding method: `op_OnesComplement`)

* The `IsTrue` and `IsFalse` operators (corresponding methods: `op_True`, `op_False`)

All overloaded unary operators must take a single parameter of the containing type and may return any type, except for `IsTrue` and `IsFalse`, which must return `Boolean`. If the containing type is a generic type, the type parameters must match the containing type's type parameters. For example,

```vb
Structure Complex
    ...

    Public Shared Operator +(v As Complex) As Complex
        Return v
    End Operator
End Structure
```

If a type overloads one of `IsTrue` or `IsFalse`, then it must overload the other as well. If only one is overloaded, a compile-time error results.

__Note.__ `IsTrue` and `IsFalse` are not reserved words.

### Binary Operators

The following binary operators can be overloaded:

* The addition `+`, subtraction `-`, multiplication `*`, division `/`, integral division `\`, modulo `Mod` and exponentiation `^` operators (corresponding method: `op_Addition`, `op_Subtraction`, `op_Multiply`, `op_Division`, `op_IntegerDivision`, `op_Modulus`, `op_Exponent`)

* The relational operators `=`, `<>`, `<`, `>`, `<=`, `>=` (corresponding methods: `op_Equality`, `op_Inequality`, `op_LessThan`, `op_GreaterThan`, `op_LessThanOrEqual`, `op_GreaterThanOrEqual`). __Note.__ While the equality operator can be overloaded, the assignment operator (used only in assignment statements) cannot be overloaded.

* The `Like` operator (corresponding method: `op_Like`)

* The concatenation operator `&` (corresponding method: `op_Concatenate`)

* The logical `And`, `Or` and `Xor` operators (corresponding methods: `op_BitwiseAnd`, `op_BitwiseOr`, `op_ExclusiveOr`)

* The shift operators `<<` and `>>` (corresponding methods: `op_LeftShift`, `op_RightShift`)

All overloaded binary operators must take the containing type as one of the parameters. If the containing type is a generic type, the type parameters must match the containing type's type parameters. The shift operators further restrict this rule to require the first parameter to be of the containing type; the second parameter must always be of type `Integer`.

The following binary operators must be declared in pairs:

* Operator `=` and operator `<>`

* Operator `>` and operator `<`

* Operator `>=` and operator `<=`

If one of the pair is declared, then the other must also be declared with matching parameter and return types, or a compile-time error will result. (__Note.__ The purpose of requiring paired declarations of relational operators is to try and ensure at least a minimum level of logical consistency in overloaded operators.)

In contrast to the relational operators, overloading both the division and integral division operators is strongly discouraged, although not an error. (__Note.__ In general, the two types of division should be entirely distinct: a type that supports division is either integral (in which case it should support `\`) or not (in which case it should support `/`). We considered making it an error to define both operators, but because their languages do not generally distinguish between two types of division the way Visual Basic does, we felt it was safest to allow the practice but strongly discourage it.)

Compound assignment operators cannot be overloaded directly. Instead, when the corresponding binary operator is overloaded, the compound assignment operator will use the overloaded operator. For example:

```vb
Structure Complex
    ...

    Public Shared Operator +(x As Complex, y As Complex) _
        As Complex
        ...
    End Operator
End Structure

Module Test
    Sub Main()
        Dim c1, c2 As Complex
        ' Calls the overloaded + operator
        c1 += c2
    End Sub
End Module
```

### Conversion Operators

Conversion operators define new conversions between types. These new conversions are called *user-defined conversions*. A conversion operator converts from a source type, indicated by the parameter type of the conversion operator, to a target type, indicated by the return type of the conversion operator. Conversions must be classified as either widening or narrowing. A conversion operator declaration that includes the `Widening` keyword introduces a user-defined widening conversion (corresponding method: `op_Implicit`). A conversion operator declaration that includes the `Narrowing` keyword introduces a user-defined narrowing conversion (corresponding method: `op_Explicit`).

In general, user-defined widening conversions should be designed to never throw exceptions and never lose information. If a user-defined conversion can cause exceptions (for example, because the source argument is out of range) or loss of information (such as discarding high-order bits), then that conversion should be defined as a narrowing conversion. In the example:

```vb
Structure Digit
    Dim value As Byte

    Public Sub New(value As Byte)
        if value < 0 OrElse value > 9 Then Throw New ArgumentException()
        Me.value = value
    End Sub

    Public Shared Widening Operator CType(d As Digit) As Byte
        Return d.value
    End Operator

    Public Shared Narrowing Operator CType(b As Byte) As Digit
        Return New Digit(b)
    End Operator
End Structure
```

the conversion from `Digit` to `Byte` is a widening conversion because it never throws exceptions or loses information, but the conversion from `Byte` to `Digit` is a narrowing conversion since `Digit` can only represent a subset of the possible values of a `Byte`.

Unlike all other type members that can be overloaded, the signature of a conversion operator includes the target type of the conversion. This is the only type member for which the return type participates in the signature. The widening or narrowing classification of a conversion operator, however, is not part of the operator's signature. Thus, a class or structure cannot declare both a widening conversion operator and a narrowing conversion operator with the same source and target types.

A user-defined conversion operator must convert either to or from the containing type -- for example, it is possible for a class `C` to define a conversion from `C` to `Integer` and from `Integer` to `C`, but not from `Integer` to `Boolean`. If the containing type is a generic type, the type parameters must match the containing type's type parameters. Also, it is not possible to redefine an intrinsic (i.e. non-user-defined) conversion. As a result, a type cannot declare a conversion where:

* The source type and the destination type are the same.

* Both the source type and the destination type are not the type that defines the conversion operator.

* The source type or the destination type is an interface type.

* The source type and destination types are related by inheritance (including `Object`).

The only exception to these rules applies to nullable value types. Since nullable value types do not have an actual type definition, a value type can declare user-defined conversions for the nullable version of the type. When determining whether a type can declare a particular user-defined conversion, the `?` modifiers are first dropped off of all of the types involved in the declaration for the purposes of validity checking. Thus, the following declaration is valid because `S` can define a conversion from `S` to `T`:

```vb
Structure T
    ...
End Structure

Structure S
    Public Shared Widening Operator CType(ByVal v As S?) As T
    ...
    End Operator
End Structure
```

The following declaration is not valid, however, because structure `S` cannot define a conversion from `S` to `S`:

```vb
Structure S
    Public Shared Widening Operator CType(ByVal v As S) As S?
        ...
    End Operator
End Structure
```

### Operator Mapping

Because the set of operators that Visual Basic supports may not exactly match the set of operators that other languages on the .NET Framework, some operators are mapped specially onto other operators when being defined or used. Specifically:

* Defining an integral division operator will automatically define a regular division operator (usable only from other languages) that will call the integral division operator.

* Overloading the `Not`, `And`, and `Or` operators will overload only the bitwise operator from the perspective of other languages that distinguish between logical and bitwise operators.

* A class that overloads only the logical operators in a language that distinguishes between logical and bitwise operators (i.e. a languages that uses `op_LogicalNot`, `op_LogicalAnd`, and `op_LogicalOr` for `Not`, `And`, and `Or`, respectively) will have their logical operators mapped onto the Visual Basic logical operators. If both the logical and bitwise operators are overloaded, only the bitwise operators will be used.

* Overloading the `<<` and `>>` operators will overload only the signed operators from the perspective of other languages that distinguish between signed and unsigned shift operators.

* A class that overloads only an unsigned shift operator will have the unsigned shift operator mapped onto the corresponding Visual Basic shift operator. If both an unsigned and signed shift operator is overloaded, only the signed shift operator will be used.
