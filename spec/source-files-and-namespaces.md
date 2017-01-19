# Source Files and Namespaces

A Visual Basic program consists of one or more source files. When a program is compiled, all of the source files are processed together; thus, source files can depend on each other, possibly in a circular fashion, without any forward-declaration requirement. The textual order of declarations in the program text is generally of no significance.

A source file consists of an optional set of option statements, import statements, and attributes, which are followed by a namespace body. The attributes, which must each have either the `Assembly` or `Module` modifier, apply to the .NET assembly or module produced by the compilation. The body of the source file functions as an implicit namespace declaration for the global namespace, meaning that all declarations at the top level of a source file are placed in the global namespace. For example:

FileA.vb:

```vb
Class A
End Class
```

FileB.vb:

```vb
Class B
End Class
```

The two source files contribute to the global namespace, in this case declaring two classes with the fully qualified names `A` and `B`. Because the two source files contribute to the same declaration space, it would have been an error if each contained a declaration of a member with the same name.

__Note.__ The compilation environment may override the namespace declarations into which a source file is implicitly placed.

Except where noted, statements within a Visual Basic program can be terminated either by a line terminator or by a colon.

```antlr
Start
    : OptionStatement* ImportsStatement* AttributesStatement* NamespaceMemberDeclaration*
    ;

StatementTerminator
    : LineTerminator
    | ':'
    ;

AttributesStatement
    : Attributes StatementTerminator
    ;
```

## Program Startup and Termination

Program startup occurs when the execution environment executes a designated method, which is referred to as the program's *entry point*. This entry point method, which must always be named `Main`, must be shared, cannot be contained in a generic type, cannot have the async modifier, and must have one of the following signatures:

```vb
Sub Main()
Sub Main(args() As String)
Function Main() As Integer
Function Main(args() As String) As Integer
```

The accessibility of the entry point method is irrelevant. If a program contains more than one suitable entry point, the compilation environment must designate one as the entry point. Otherwise, a compile-time error occurs. The compilation environment may also create an entry point method if one does not exist.

When a program begins, if the entry point has a parameter, the argument supplied by the execution environment contains the command-line arguments to the program represented as strings. If the entry point has a return type of `Integer`, then the value returned from the function is returned to the execution environment as the result of the program.

In all other respects, entry point methods behave in the same manner as other methods. When execution leaves the invocation of the entry point method made by the execution environment, the program terminates.

## Compilation Options

A source file can specify compilation options in the source code using *option statements*.

```antlr
OptionStatement
    : OptionExplicitStatement
    | OptionStrictStatement
    | OptionCompareStatement
    | OptionInferStatement
    ;
```

An `Option` statement applies only to the source file in which it appears, and only one of each type of `Option` statement may appear in a source file. For example:

```vb
Option Strict On
Option Compare Text
Option Strict Off    ' Not allowed, Option Strict is already specified.
Option Compare Text  ' Not allowed, Option Compare is already specified.
```

There are four compilation options: strict type semantics, explicit declaration semantics, comparison semantics, and local variable type inference semantics. If a source file does not include a particular `Option` statement, then the compilation environment determines which particular set of semantics will be used. There is also a fifth compilation option, integer overflow checks, which can only be specified through the compilation environment.


### Option Explicit Statement

The `Option Explicit` statement determines whether local variables may be implicitly declared. The keywords `On` or `Off` may follow the statement; if neither is specified, the default is `On`. If no statement is specified in a file, the compilation environment determines which will be used.

```antlr
OptionExplicitStatement
    : 'Option' 'Explicit' OnOff? StatementTerminator
    ;

OnOff
    : 'On' | 'Off'
    ;
```


__Note.__ `Explicit` and `Off` are not reserved words.

```vb
Option Explicit Off

Module Test
    Sub Main()
        x = 5 ' Valid because Option Explicit is off.
    End Sub
End Module
```

In this example, the local variable `x` is implicitly declared by assigning to it. The type of `x` is `Object`.


### Option Strict Statement

The `Option Strict` statement determines whether conversions and operations on `Object` are governed by strict or permissive type semantics and whether types are implicitly typed as `Object` if no `As` clause is specified. The statement may be followed by the keywords `On` or `Off`; if neither is specified, the default is `On`. If no statement is specified in a file, the compilation environment determines which will be used.

```antlr
OptionStrictStatement
    : 'Option' 'Strict' OnOff? StatementTerminator
    ;
```

__Note.__ `Strict` and `Off` are not reserved words.

```vb
Option Strict On

Module Test
    Sub Main()
        Dim x ' Error, no type specified.
        Dim o As Object
        Dim b As Byte = o ' Error, narrowing conversion.

        o.F() ' Error, late binding disallowed.
        o = o + 1 ' Error, addition is not defined on Object.
    End Sub
End Module
```

Under strict semantics, the following are disallowed:

* Narrowing conversions without an explicit cast operator.

* Late binding.

* Operations on type `Object` other than `TypeOf`...`Is`, `Is`, and `IsNot`.

* Omitting the `As` clause in a declaration that does not have an inferred type.


### Option Compare Statement

The `Option Compare` statement determines the semantics of string comparisons. String comparisons are carried out either using binary comparisons (in which the binary Unicode value of each character is compared) or text comparisons (in which the lexical meaning of each character is compared using the current culture). If no statement is specified in a file, the compilation environment controls which type of comparison will be used.

```antlr
OptionCompareStatement
    : 'Option' 'Compare' CompareOption StatementTerminator
    ;

CompareOption
    : 'Binary' | 'Text'
    ;
```

__Note.__ `Compare`, `Binary`, and `Text` are not reserved words.

```vb
Option Compare Text

Module Test
    Sub Main()
        Console.WriteLine("a" = "A")    ' Prints True.
    End Sub
End Module
```

In this case, the string comparison is done using a text comparison that ignores case differences. If `Option Compare Binary` had been specified, then this would have printed `False`.


### Integer Overflow Checks

Integer operations can either be checked or not checked for overflow conditions at run time. If overflow conditions are checked and an integer operation overflows, a `System.OverflowException` exception is thrown. If overflow conditions are not checked, integer operation overflows do not throw an exception. The compilation environment determines whether this option is on or off.

### Option Infer Statement

The `Option Infer` statement determines whether local variable declarations that have no `As` clause have an inferred type or use `Object`. The statement may be followed by the keywords `On` or `Off`; if neither is specified, the default is `On`. If no statement is specified in a file, the compilation environment determines which will be used.

```antlr
OptionInferStatement
    : 'Option' 'Infer' OnOff? StatementTerminator
    ;
```

__Note.__ `Infer` and `Off` are not reserved words.

```vb
Option Infer On

Module Test
    Sub Main()
        ' The type of x is Integer
        Dim x = 10

        ' The type of y is String
        Dim y = "abc"
    End Sub
End Module
```


## Imports Statement

`Imports` statements import the names of entities into a source file, allowing the names to be referenced without qualification, or import a namespace for use in XML expressions.

```antlr
ImportsStatement
    : 'Imports' ImportsClauses StatementTerminator
    ;

ImportsClauses
    : ImportsClause ( Comma ImportsClause )*
    ;

ImportsClause
    : AliasImportsClause
    | MembersImportsClause
    | XMLNamespaceImportsClause
    ;
```

Within member declarations in a source file that contains an `Imports` statement, the types contained in the given namespace can be referenced directly, as seen in the following example:

```vb
Imports N1.N2

Namespace N1.N2
    Class A
    End Class
End Namespace 

Namespace N3
    Class B
        Inherits A
    End Class
End Namespace
```

Here, within the source file, the type members of namespace `N1.N2` are directly available, and thus class `N3.B` derives from class `N1.N2.A`.

`Imports` statements must appear after any `Option` statements but before any type declarations. The compilation environment may also define implicit `Imports` statements.

`Imports` statements make names available in a source file, but do not declare anything in the global namespace's declaration space. The scope of the names imported by an `Imports` statement extends over the namespace member declarations contained in the source file. The scope of an `Imports` statement specifically does not include other `Imports` statements, nor does it include other source files. `Imports` statements may not refer to one another.

In this example, the last `Imports` statement is in error because it is not affected by the first import alias.

```vb
Imports R1 = N1 ' OK.
Imports R2 = N1.N2 ' OK.
Imports R3 = R1.N2 ' Error: Can't refer to R1.

Namespace N1.N2
End Namespace
```

__Note.__ The namespace or type names that appear in `Imports` statements are always treated as if they are fully qualified. That is, the leftmost identifier in a namespace or type name always resolves in the global namespace and the rest of the resolution proceeds according to normal name resolution rules. This is the only place in the language that applies such a rule; the rule ensures that a name cannot be completely hidden from qualification. Without the rule, if a name in the global namespace were hidden in a particular source file, it would be impossible to specify any names from that namespace in a qualified way.

In this example, the `Imports` statement always refers to the global `System` namespace, and not the class in the source file.

```vb
Imports System   ' Imports the namespace, not the class.

Class System
End Class
```


### Import Aliases

An *import alias* defines an alias for a namespace or type.

```antlr
AliasImportsClause
    : Identifier Equals TypeName
    ;
```

```vb
Imports A = N1.N2.A

Namespace N1.N2
    Class A
    End Class
End Namespace

Namespace N3
    Class B
        Inherits A
    End Class
End Namespace
```

Here, within the source file, `A` is an alias for `N1.N2.A`, and thus class `N3.B` derives from class `N1.N2.A`. The same effect can be obtained by creating an alias `R` for `N1.N2` and then referencing `R.A`:

```vb
Imports R = N1.N2

Namespace N3
    Class B
        Inherits R.A
    End Class
End Namespace
```

The identifier of an import alias must be unique within the declaration space of the global namespace (not just the global namespace declaration in the source file in which the import alias is defined), even though it does not declare a name in the global namespace's declaration space.

__Note.__ Declarations in a module do not introduce names into the containing declaration space. Thus, it is valid for a declaration in a module to have the same name as an import alias, even though the declaration's name will be accessible in the containing declaration space.

```vb
' Error: Alias A conflicts with typename A
Imports A = N3.A

Class A
End Class

Namespace N3
    Class A
    End Class
End Namespace
```

Here, the global namespace already contains a member `A`, so it is an error for an import alias to use that identifier. It is likewise an error for two or more import aliases in the same source file to declare aliases by the same name.

An import alias can create an alias for any namespace or type. Accessing a namespace or type through an alias yields exactly the same result as accessing the namespace or type through its declared name.

```vb
Imports R1 = N1
Imports R2 = N1.N2

Namespace N1.N2
    Class A
    End Class
End Namespace

Namespace N3
    Class B
        Private a As N1.N2.A
        Private b As R1.N2.A
        Private c As R2.A
    End Class
End Namespace
```

Here, the names `N1.N2.A`, `R1.N2.A`, and `R2.A` are equivalent, and all refer to the class whose fully qualified name is `N1.N2.A`.

The import specifies the exact name of the namespace or type to which it is creating an alias. This must be the exact fully qualified name of that namespace or type: it does not use the normal rules for qualified name resolution (which for instance allow access to the members of a base class through a derived class).

If an import alias points to a type or namespace which cannot be resolved by these rules, then the import statement is ignored (and the compiler gives a warning).

Also, the reference cannot be to an open generic type -- all generic types must have valid type arguments supplied, and all type arguments must be resolvable by the rules above. Any incorrect binding of a generic type is an error.

```vb
Imports A = G              ' error: since G is an open generic type
Imports B = G(Of Integer)  ' okay
Imports C = Derived.Nested ' warning: Derived.Nested isn't itself a type
Imports D = G(Of Derived.Nested) ' error: Derived.Nested isn't found

Class G(Of T) : End Class

Class Base
    Class Nested : End Class
End Class

Class Derived : Inherits Base
End Class

Module Module1
    Sub Main()
        Dim x As C               ' error: "C" wasn't succesfully defined
        Dim y As Derived.Nested  ' okay
    End Sub
End Module
```

Declarations in the source file may shadow the import alias name.

```vb
Imports R = N1.N2

Namespace N1.N2
    Class A
    End Class
End Namespace

Namespace N3
    Class R
    End Class

    Class B
        Inherits R.A  ' Error, R has no member A
    End Class
End Namespace
```

In the preceding example the reference to `R.A` in the declaration of `B` causes an error because `R` refers to `N3.R`, not `N1.N2`.

An import alias makes an alias available within a particular source file, but it does not contribute any new members to the underlying declaration space. In other words, an import alias is not transitive, but rather affects only the source file in which it occurs.

File1.vb:

```vb
Imports R = N1.N2

Namespace N1.N2
    Class A
    End Class
End Namespace
```

File2.vb:

```vb
Class B
    Inherits R.A ' Error, R unknown.
End Class
```

In the above example, because the scope of the import alias that introduces `R` only extends to declarations in the source file in which it is contained, `R` is unknown in the second source file.


### Namespace Imports

A *namespace import* imports all of the members of a namespace or type, allowing the identifier of each member of the namespace or type to be used without qualification. In the case of types, a namespace import only allows access to the shared members of the type without requiring qualification of the class name. In particular, it allows the members of enumerated types to be used without qualification.

```antlr
MembersImportsClause
    : TypeName
    ;
```

For example:

```vb
Imports Colors

Enum Colors
    Red
    Green
    Blue
End Enum

Module M1
    Sub Main()
        Dim c As Colors = Red
    End Sub
End Module
```

Unlike an import alias, a namespace import has no restrictions on the names it imports and may import namespaces and types whose identifiers are already declared within the global namespace. The names imported by a regular import are shadowed by import aliases and declarations in the source file.

In the following example, `A` refers to `N3.A` rather than `N1.N2.A` within member declarations in the `N3` namespace.

```vb
Imports N1.N2

Namespace N1.N2
    Class A
    End Class
End Namespace

Namespace N3
    Class A
    End Class

    Class B
        Inherits A
    End Class
End Namespace
```

When more than one imported namespace contains members by the same name (and that name is not otherwise shadowed by an import alias or declaration), a reference to that name is ambiguous and causes a compile-time error.

```vb
Imports N1
Imports N2

Namespace N1
    Class A
    End Class
End Namespace 

Namespace N2
    Class A
    End Class
End Namespace 

Namespace N3
    Class B
        Inherits A ' Error, A is ambiguous.
    End Class
End Namespace
```

In the above example, both `N1` and `N2` contain a member `A`. Because `N3` imports both, referencing `A` in `N3` causes a compile-time error. In this situation, the conflict can be resolved either through qualification of references to `A`, or by introducing an import alias that picks a particular `A`, as in the following example:

```vb
Imports N1
Imports N2
Imports A = N1.A

Namespace N3
    Class B
        Inherits A ' A means N1.A.
    End Class
End Namespace
```

Only namespaces, classes, structures, enumerated types, and standard modules may be imported.


### XML Namespace Imports

An *XML namespace import* defines a namespace or the default namespace for unqualified XML expressions contained within the compilation unit.

```antlr
XMLNamespaceImportsClause
    : '<' XMLNamespaceAttributeName XMLWhitespace? Equals XMLWhitespace?
      XMLNamespaceValue '>'
    ;

XMLNamespaceValue
    : DoubleQuoteCharacter XMLAttributeDoubleQuoteValueCharacter* DoubleQuoteCharacter
    | SingleQuoteCharacter XMLAttributeSingleQuoteValueCharacter* SingleQuoteCharacter
    ;
```

For example:

```vb
Imports <xmlns:db="http://example.org/database">

Module Test
    Sub Main()
        ' db namespace is "http://example.org/database"
        Dim x = <db:customer><db:Name>Bob</></>

        Console.WriteLine(x.<db:Name>)
    End Sub
End Module
```

An XML namespace, including the default namespace, can only be defined once for a particular set of imports. For example:

```vb
Imports <xmlns:db="http://example.org/database-one">
' Error: namespace db is already defined
Imports <xmlns:db="http://example.org/database-two">
```


## Namespaces

Visual Basic programs are organized using namespaces. Namespaces both internally organize a program as well as organize the way program elements are exposed to other programs.

Unlike other entities, namespaces are open-ended, and may be declared multiple times within the same program and across many programs, with each declaration contributing members to the same namespace. In the following example, the two namespace declarations contribute to the same declaration space, declaring two classes with the fully qualified names `N1.N2.A` and `N1.N2.B`.

```vb
Namespace N1.N2
    Class A
    End Class
End Namespace

Namespace N1.N2
    Class B
    End Class
End Namespace
```

Because the two declarations contribute to the same declaration space, it would be an error if each contained a declaration of a member with the same name.

There is a global namespace that has no name and whose nested namespaces and types can always be accessed without qualification. The scope of a namespace member declared in the global namespace is the entire program text. Otherwise, the scope of a type or namespace declared in a namespace whose fully qualified name is `N` is the program text of each namespace whose corresponding namespace's fully qualified name begins with `N` or is `N` itself. (Note that a compiler can choose to put declarations in a particular namespace by default. This does not alter the fact that there is still a global, unnamed namespace.)

In this example, the class `B` can see the class `A` because `B`'s namespace `N1.N2.N3` is conceptually nested within the namespace `N1.N2`.

```vb
Namespace N1.N2
    Class A
    End Class
End Namespace

Namespace N1.N2.N3
    Class B
        Inherits A
    End Class
End Namespace
```

### Namespace Declarations

There are three forms of namespace declaration.

```antlr
NamespaceDeclaration
    : 'Namespace' NamespaceName StatementTerminator
      NamespaceMemberDeclaration*
      'End' 'Namespace' StatementTerminator
    ;

NamespaceName
    : RelativeNamespaceName
    | 'Global'
    | 'Global' '.' RelativeNamespaceName
    ;

RelativeNamespaceName
    : Identifier ( Period IdentifierOrKeyword )*
    ;
```

The first form starts with the keyword `Namespace` followed by a relative namespace name. If the relative namespace name is qualified, the namespace declaration is treated as if it is lexically nested within namespace declarations corresponding to each name in the qualified name. For example, the following two namespaces are semantically equivalent:

```vb
Namespace N1.N2
    Class A
    End Class

    Class B
    End Class
End Namespace 

Namespace N1
    Namespace N2
        Class A
        End Class

        Class B
        End Class
    End Namespace
End Namespace
```

The second form starts with the keywords `Namespace Global`. It is treated as if all its member declarations were lexically placed in the global unnamed namespace -- regardless of any defaults provided by the compilation environment. This form of namespace declaration may not be lexically nested within any other namespace declaration.

The third form starts with the keywords `Namespace Global` followed by a qualified identifier `N`. It is treated as if it were a namespace declaration of the first form "`Namespace N`" that was lexically placed in the global unnamed namespace -- regardless of any defaults provided by the compilation environment. This form of namespace declaration may not be lexically nested within any other namespace declaration.

```vb
Namespace Global       ' Puts N1.A in the global namespace
    Namespace N1
        Class A
        End Class
    End Namespace
End Namespace

Namespace Global.N1    ' Equivalent to the above
    Class A
    End Class
End Namespace

Namespace N1           ' May or may not be equivalent to the above,
    Class A            ' depending on defaults provided by the
    End Class          ' compilation environment
End Namespace
```

When dealing with the members of a namespace, it is not important where a particular member is declared. If two programs define an entity with the same name in the same namespace, attempting to resolve the name in the namespace causes an ambiguity error.

Namespaces are by definition `Public`, so a namespace declaration cannot include any access modifiers.


### Namespace Members

Namespace members can only be namespace declarations and type declarations. Type declarations may have `Public` or `Friend` access. The default access for types is `Friend` access.

```antlr
NamespaceMemberDeclaration
    : NamespaceDeclaration
    | TypeDeclaration
    ;

TypeDeclaration
    : ModuleDeclaration
    | NonModuleDeclaration
    ;

NonModuleDeclaration
    : EnumDeclaration
    | StructureDeclaration
    | InterfaceDeclaration
    | ClassDeclaration
    | DelegateDeclaration
    ;
```
