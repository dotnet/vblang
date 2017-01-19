# Documentation Comments

Documentation comments are specially formatted comments in the source that can be analyzed to produce documentation about the code they are attached to. The basic format for documentation comments is XML. When the compiling code with documentation comments, the compiler may optionally emit an XML file that represents the sum total of the documentation comments in the source. This XML file can then be used by other tools to produce printed or online documentation.

This chapter describes document comments and recommended XML tags to use with document comments.

## Documentation Comment Format

Document comments are special comments that begin with `'''`, three single quote marks. They must immediately precede the type (such as a class, delegate, or interface) or type member (such as a field, event, property, or method) that they document. A document comment on a partial method declaration will be replaced by the document comment on the method that supplies its body, if there is one. All adjacent document comments are appended together to produce a single document comment. If there is a whitespace character following the `'''` characters, then that whitespace character is not included in the concatenation. For example:

```vb
''' <remarks>
''' Class <c>Point</c> models a point in a two-dimensional plane.
''' </remarks>
Public Class Point 
   ''' <remarks>
   ''' Method <c>Draw</c> renders the point.
   ''' </remarks>
   Sub Draw()
   End Sub
End Class
```

Documentation comments must be well formed XML according to http://www.w3.org/TR/REC-xml. If the XML is not well formed, a warning is generated and the documentation file will contain a comment saying that an error was encountered.

Although developers are free to create their own set of tags, a recommended set is defined in the next section. Some of the recommended tags have special meanings:

* The `<param>` tag is used to describe parameters. The parameter specified by a `<param>` tag must exist and all parameters of the type member must be described in the documentation comment. If either condition is not true, the compiler issues a warning.

* The `cref` attribute can be attached to any tag to provide a reference to a code element. The code element must exist; at compile-time the compiler replaces the name with the ID string representing the member. If the code element does not exist, the compiler issues a warning. When looking for a name described in a `cref` attribute, the compiler respects `Imports` statements that appear within the containing source file.

* The `<summary>` tag is intended to be used by a documentation viewer to display additional information about a type or member.

Note that the documentation file does not provide full information about a type and members, only what is contained in the document comments. To get more information about a type or member, the documentation file must be used in conjunction with reflection on the actual type or member.

## Recommended tags

The documentation generator must accept and process any tag that is valid according to the rules of XML. The following tags provide commonly used functionality in user documentation:

`<c>` Sets text in a code-like font

`<code>` Sets one or more lines of source code or program output in a code-like font

`<example>` Indicates an example

`<exception>` Identifies the exceptions a method can throw

`<include>` Includes an external XML document

`<list>` Creates a list or table

`<para>` Permits structure to be added to text

`<param>` Describes a parameter for a method or constructor

`<paramref>` Identifies that a word is a parameter name

`<permission>` Documents the security accessibility of a member

`<remarks>` Describes a type

`<returns>` Describes the return value of a method

`<see>` Specifies a link

`<seealso>` Generates a See Also entry

`<summary>` Describes a member of a type

`<typeparam>` Describes a type parameter

`<value>` Describes a property

### &lt;c&gt;

This tag specifies that a fragment of text within a description should use a font like that used for a block of code. (For lines of actual code, use `<code>`.)

__Syntax:__

```xml
<c>text to be set like code</c>
```

__Example:__

```vb
''' <remarks>
''' Class <c>Point</c> models a point in a two-dimensional plane.
''' </remarks>
Public Class Point 
End Class
```

### &lt;code&gt;

This tag specifies that one or more lines of source code or program output should use a fixed-width font. (For small code fragments, use `<c>`.)

__Syntax:__

```xml
<code>source code or program output</code>
```

__Example:__

```vb
''' <summary>
''' This method changes the point's location by the given x- and 
''' y-offsets.
''' <example>
''' For example:
''' <code>
'''    Dim p As Point = New Point(3,5)
'''    p.Translate(-1,3)
''' </code>
''' results in <c>p</c>'s having the value (2,8).
''' </example>
''' </summary>
Public Sub Translate(x As Integer, y As Integer)
    Me.x += x
    Me.y += y
End Sub
```

### &lt;example&gt;

This tag allows example code within a comment to show how an element can be used. Ordinarily, this will involve use of the tag `<code>` as well.

__Syntax:__

```xml
<example>description</example>
```

__Example:__

See `<code>` for an example.

### &lt;exception&gt;

This tag provides a way to document the exceptions a method can throw.

__Syntax:__

```xml
<exception cref="member">description</exception>
```

__Example:__

```vb
Public Module DataBaseOperations
    ''' <exception cref="MasterFileFormatCorruptException"></exception>
    ''' <exception cref="MasterFileLockedOpenException"></exception>
    Public Sub ReadRecord(flag As Integer)
        If Flag = 1 Then
            Throw New MasterFileFormatCorruptException()
        ElseIf Flag = 2 Then
            Throw New MasterFileLockedOpenException()
        End If
        ' ...
    End Sub
End Module
```

### &lt;include&gt;

This tag is used to include information from an external well-formed XML document. An XPath expression is applied to the XML document to specify what XML should be included from the document. The `<include>` tag is then replaced with the selected XML from the external document.

__Syntax:__

```xml
<include file="filename" path="xpath">
```

__Example:__

If the source code contained a declaration like the following:

```vb
''' <include file="docs.xml" path="extra/class[@name="IntList"]/*" />
```

and the external file docs.xml had the following contents

```xml
<?xml version="1.0"?>
<extra>
    <class name="IntList">
        <summary>
            Contains a list of integers.
        </summary>
    </class>
    <class name="StringList">
        <summary>
            Contains a list of strings.
        </summary>
    </class>
</extra>
```

then the same documentation is output as if the source code contained:

```xml
''' <summary>
''' Contains a list of integers.
''' </summary>
```

### &lt;list&gt;

This tag is used to create a list or table of items. It may contain a `<listheader>` block to define the heading row of either a table or definition list. (When defining a table, only an entry for term in the heading need be supplied.)

Each item in the list is specified with an `<item>` block. When creating a definition list, both term and description must be specified. However, for a table, bulleted list, or numbered list, only description need be specified.

__Syntax:__

```xml
<list type="bullet" | "number" | "table">
    <listheader>
        <term>term</term>
        <description>description</description>
    </listheader>
    <item>
        <term>term</term>
        <description>description</description>
    </item>
    ...
    <item>
        <term>term</term>
        <description>description</description>
    </item>
</list>
```

__Example:__

```vb
Public Class TestClass
    ''' <remarks>
    ''' Here is an example of a bulleted list:
    ''' <list type="bullet">
    '''     <item>
    '''        <description>Item 1.</description>
    '''     </item>
    '''     <item>
    '''         <description>Item 2.</description>
    '''     </item>
    ''' </list>
    ''' </remarks>
    Public Shared Sub Main()
    End Sub
End Class
```

### &lt;para&gt;

This tag is for use inside other tags, such as `<remarks>` or `<returns>`, and permits structure to be added to text.

__Syntax:__

```xml
<para>content</para>
```

__Example:__

```vb
''' <summary>
''' This is the entry point of the Point class testing program.
''' <para>This program tests each method and operator, and
''' is intended to be run after any non-trvial maintenance has
''' been performed on the Point class.</para>
''' </summary>
Public Shared Sub Main()
End Sub
```

### &lt;param&gt;

This tag describes a parameter for a method, constructor, or indexed property.

__Syntax:__

```xml
<param name="name">description</param>
```

__Example:__

```vb
''' <summary>
''' This method changes the point's location to the given
''' coordinates.
''' </summary>
''' <param name="x"><c>x</c> is the new x-coordinate.</param>
''' <param name="y"><c>y</c> is the new y-coordinate.</param>
Public Sub Move(x As Integer, y As Integer)
    Me.x = x
    Me.y = y
End Sub
```

### &lt;paramref&gt;

This tag indicates that a word is a parameter. The documentation file can be processed to format this parameter in some distinct way.

__Syntax:__

```xml
<paramref name="name"/>
```

__Example:__

```vb
''' <summary>
''' This constructor initializes the new Point to
''' (<paramref name="x"/>,<paramref name="y"/>).
''' </summary>
''' <param name="x"><c>x</c> is the new Point's x-coordinate.</param>
''' <param name="y"><c>y</c> is the new Point's y-coordinate.</param>
Public Sub New(x As Integer, y As Integer)
    Me.x = x
    Me.y = y
End Sub
```

### &lt;permission&gt;

This tag documents the security accessibility of a member

__Syntax:__

```xml
<permission cref="member">description</permission>
```

__Example:__

```vb
''' <permission cref="System.Security.PermissionSet">Everyone can
''' access this method.</permission>
Public Shared Sub Test()
End Sub
```

### &lt;remarks&gt;

This tag specifies overview information about a type. (Use `<summary>` to describe the members of a type.)

__Syntax:__

```xml
<remarks>description</remarks>
```

__Example:__

```vb
''' <remarks>
''' Class <c>Point</c> models a point in a two-dimensional plane.
''' </remarks>
Public Class Point 
End Class
```

### &lt;returns&gt;

This tag describes the return value of a method.

__Syntax:__

```xml
<returns>description</returns>
```

__Example:__

```vb
''' <summary>
''' Report a point's location as a string.
''' </summary>
''' <returns>
''' A string representing a point's location, in the form (x,y), without
''' any leading, training, or embedded whitespace.
''' </returns>
Public Overrides Function ToString() As String
    Return "(" & x & "," & y & ")"
End Sub
```

### &lt;see&gt;

This tag allows a link to be specified within text. (Use `<seealso>` to indicate text that is to appear in a See Also section.)

__Syntax:__

```xml
<see cref="member"/>
```

__Example:__

```vb
''' <summary>
''' This method changes the point's location to the given
''' coordinates.
''' </summary>
''' <see cref="Translate"/>
Public Sub Move(x As Integer, y As Integer)
    Me.x = x
    Me.y = y
End Sub

''' <summary>
''' This method changes the point's location by the given x- and
''' y-offsets.
''' </summary>
''' <see cref="Move"/>
Public Sub Translate(x As Integer, y As Integer)
    Me.x += x
    Me.y += y
End Sub
```

### &lt;seealso&gt;

This tag generates an entry for the See Also section. (Use `<see>` to specify a link from within text.)

__Syntax:__

```xml
<seealso cref="member"/>
```

__Example:__

```vb
''' <summary>
''' This method determines whether two Points have the same location.
''' </summary>
''' <seealso cref="operator=="/>
''' <seealso cref="operator!="/>
Public Overrides Function Equals(o As Object) As Boolean
    ' ...
End Function
```

### &lt;summary&gt;

This tag describes a type member. (Use `<remarks>` to describe a type itself.)

__Syntax:__

```xml
<summary>description</summary>
```

__Example:__

```vb
''' <summary>
''' This constructor initializes the new Point to (0,0).
''' </summary>
Public Sub New()
    Me.New(0,0)
End Sub
```

### &lt;typeparam&gt;

This tag describes a type parameter.

__Syntax:__

```xml
<typeparam name="name">description</typeparam>
```

__Example:__

```vb
''' <typeparam name="T">
''' The base item type. Must implement IComparable.
''' </typeparam>
Public Class ItemManager(Of T As IComparable)
End Class
```

### &lt;value&gt;

This tag describes a property.

__Syntax:__

```xml
<value>property description</value>
```

__Example:__

```vb
''' <value>
''' Property <c>X</c> represents the point's x-coordinate.
''' </value>
Public Property X() As Integer
    Get
        Return _x
    End Get
    Set (Value As Integer)
        _x = Value
    End Set
End Property
```

## ID Strings

When generating the documentation file, the compiler generates an ID string for each element in the source code that is tagged with a documentation comment that uniquely identifies it. This ID string can be used by external tools to identify which element in a compiled assembly corresponds to the document comment.

ID strings are generated as follows:

No white space is placed in the string.

The first part of the string identifies the kind of member being documented, via a single character followed by a colon. The following kinds of members are defined, with the corresponding character in parenthesis after it: events (E), fields (F), methods including constructors and operators (M), namespaces (N), properties (P) and types (T). An exclamation point (!) indicates an error occurred while generating the ID string, and the rest of the string provides information about the error.

The second part of the string is the fully qualified name of the element, starting at the global namespace. The name of the element, its enclosing type(s), and namespace are separated by periods. If the name of the item itself has periods, they are replaced by the pound sign (#). (It is assumed that no element has this character in its name.) The name of a type with type parameters ends with a backquote (`) followed by a number that represents the number of type parameters on the type. It is important to remember that because nested types have access to the type parameters of the types containing them, nested types implicitly contain the type parameters of their containing types, and those types are counted in their type parameter totals in this case.

For methods and properties with arguments, the argument list follows, enclosed in parentheses. For those without arguments, the parentheses are omitted. The arguments are separated by commas. The encoding of each argument is the same as a CLI signature, as follows: Arguments are represented by their fully qualified name. For example, `Integer` becomes `System.Int32`, `String` becomes `System.String`, `Object` becomes `System.Object`, and so on. Arguments having the `ByRef` modifier have a '@' following their type name. Arguments having the `ByVal`, `Optional` or `ParamArray` modifier have no special notation. Arguments that are arrays are represented as `[lowerbound:size, ..., lowerbound:size]` where the number of commas is the rank - 1, and the lower bounds and size of each dimension, if known, are represented in decimal. If a lower bound or size is not specified, it is omitted. If the lower bound and size for a particular dimension are omitted, the ':' is omitted as well. Arrays of arrays are represented by one "`[]`" per level.

### ID string examples

The following examples each show a fragment of VB code, along with the ID string produced from each source element capable of having a documentation comment:

Types are represented using their fully qualified name.

```vb
Enum Color
    Red
    Blue
    Green
End Enum

Namespace Acme
    Interface IProcess
    End Interface

    Structure ValueType
        ...
    End Structure

    Class Widget
        Public Class NestedClass
        End Class

        Public Interface IMenuItem
        End Interface

        Public Delegate Sub Del(i As Integer)

        Public Enum Direction
            North
            South
            East
            West
        End Enum
    End Class
End Namespace

"T:Color"
"T:Acme.IProcess"
"T:Acme.ValueType"
"T:Acme.Widget"
"T:Acme.Widget.NestedClass"
"T:Acme.Widget.IMenuItem"
"T:Acme.Widget.Del"
"T:Acme.Widget.Direction"
```

Fields are represented by their fully qualified name.

```vb
Namespace Acme
    Structure ValueType
        Private total As Integer
    End Structure

    Class Widget
        Public Class NestedClass
            Private value As Integer
        End Class

        Private message As String
        Private Shared defaultColor As Color
        Private Const PI As Double = 3.14159
        Protected ReadOnly monthlyAverage As Double
        Private array1() As Long
        Private array2(,) As Widget
    End Class
End Namespace

"F:Acme.ValueType.total"
"F:Acme.Widget.NestedClass.value"
"F:Acme.Widget.message"
"F:Acme.Widget.defaultColor"
"F:Acme.Widget.PI"
"F:Acme.Widget.monthlyAverage"
"F:Acme.Widget.array1"
"F:Acme.Widget.array2"
```

Constructors.

```vb
Namespace Acme
    Class Widget
        Shared Sub New()
        End Sub

        Public Sub New()
        End Sub

        Public Sub New(s As String)
        End Sub
    End Class
End Namespace

"M:Acme.Widget.#cctor"
"M:Acme.Widget.#ctor"
"M:Acme.Widget.#ctor(System.String)"
```

Methods.

```vb
Namespace Acme
    Structure ValueType
        Public Sub M(i As Integer)
        End Sub
    End Structure

    Class Widget
        Public Class NestedClass
            Public Sub M(i As Integer)
            End Sub
        End Class

        Public Shared Sub M0()
        End Sub

        Public Sub M1(c As Char, ByRef f As Float, _
            ByRef v As ValueType)
        End Sub

        Public Sub M2(x1() As Short, x2(,) As Integer, _
            x3()() As Long)
        End Sub

        Public Sub M3(x3()() As Long, x4()(,,) As Widget)
        End Sub

        Public Sub M4(Optional i As Integer = 1)

        Public Sub M5(ParamArray args() As Object)
        End Sub
    End Class
End Namespace

"M:Acme.ValueType.M(System.Int32)"
"M:Acme.Widget.NestedClass.M(System.Int32)"
"M:Acme.Widget.M0"
"M:Acme.Widget.M1(System.Char,System.Single@,Acme.ValueType@)"
"M:Acme.Widget.M2(System.Int16[],System.Int32[0:,0:],System.Int64[][])"
"M:Acme.Widget.M3(System.Int64[][],Acme.Widget[0:,0:,0:][])"
"M:Acme.Widget.M4(System.Int32)"
"M:Acme.Widget.M5(System.Object[])"
```

Properties.

```vb
Namespace Acme
    Class Widget
        Public Property Width() As Integer
            Get
            End Get
            Set (Value As Integer)
            End Set
        End Property

        Public Default Property Item(i As Integer) As Integer
            Get
            End Get
            Set (Value As Integer)
            End Set
        End Property

        Public Default Property Item(s As String, _
            i As Integer) As Integer
            Get
            End Get
            Set (Value As Integer)
            End Set
        End Property
    End Class
End Namespace

"P:Acme.Widget.Width"
"P:Acme.Widget.Item(System.Int32)"
"P:Acme.Widget.Item(System.String,System.Int32)"
```

Events   

```vb
Namespace Acme
    Class Widget
        Public Event AnEvent As EventHandler
        Public Event AnotherEvent()
    End Class
End Namespace

"E:Acme.Widget.AnEvent"
"E:Acme.Widget.AnotherEvent"
```

Operators.

```vb
Namespace Acme
    Class Widget
        Public Shared Operator +(x As Widget) As Widget
        End Operator

        Public Shared Operator +(x1 As Widget, x2 As Widget) As Widget
        End Operator
    End Class
End Namespace

"M:Acme.Widget.op_UnaryPlus(Acme.Widget)"
"M:Acme.Widget.op_Addition(Acme.Widget,Acme.Widget)"
```

Conversion operators have a trailing `~` followed by the return type.

```vb
Namespace Acme
    Class Widget
        Public Shared Narrowing Operator CType(x As Widget) As _
            Integer
        End Operator

        Public Shared Widening Operator CType(x As Widget) As Long
        End Operator
    End Class
End Namespace

"M:Acme.Widget.op_Explicit(Acme.Widget)~System.Int32"
"M:Acme.Widget.op_Implicit(Acme.Widget)~System.Int64"
```

## Documentation comments example

The following example shows the source code of a `Point` class:

```vb
Namespace Graphics
    ''' <remarks>
    ''' Class <c>Point</c> models a point in a two-dimensional
    ''' plane.
    ''' </remarks>
    Public Class Point
        ''' <summary>
        ''' Instance variable <c>x</c> represents the point's x-coordinate.
        ''' </summary>
        Private _x As Integer

        ''' <summary>
        ''' Instance variable <c>y</c> represents the point's y-coordinate.
        ''' </summary>
        Private _y As Integer

        ''' <value>
        ''' Property <c>X</c> represents the point's x-coordinate.
        ''' </value>
        Public Property X() As Integer
            Get
                Return _x
            End Get
            Set(Value As Integer)
                _x = Value
            End Set
        End Property

        ''' <value>
        ''' Property <c>Y</c> represents the point's y-coordinate.
        ''' </value>
        Public Property Y() As Integer
            Get
                Return _y
            End Get
            Set(Value As Integer)
                _y = Value
            End Set
        End Property

        ''' <summary>
        ''' This constructor initializes the new Point to (0,0).
        ''' </summary>
        Public Sub New()
            Me.New(0, 0)
        End Sub

        ''' <summary>
        ''' This constructor initializes the new Point to
        ''' (<paramref name="x"/>,<paramref name="y"/>).
        ''' </summary>
        ''' <param name="x"><c>x</c> is the new Point's
        ''' x-coordinate.</param>
        ''' <param name="y"><c>y</c> is the new Point's
        ''' y-coordinate.</param>
        Public Sub New(x As Integer, y As Integer)
            Me.X = x
            Me.Y = y
        End Sub

        ''' <summary>
        ''' This method changes the point's location to the given
        ''' coordinates.
        ''' </summary>
        ''' <param name="x"><c>x</c> is the new x-coordinate.</param>
        ''' <param name="y"><c>y</c> is the new y-coordinate.</param>
        ''' <see cref="Translate"/>
        Public Sub Move(x As Integer, y As Integer)
            Me.X = x
            Me.Y = y
        End Sub

        ''' <summary>
        ''' This method changes the point's location by the given x- and
        ''' y-offsets.
        ''' <example>
        ''' For example:
        ''' <code>
        '''    Dim p As Point = New Point(3, 5)
        '''    p.Translate(-1, 3)
        ''' </code>
        ''' results in <c>p</c>'s having the value (2,8).
        ''' </example>
        ''' </summary>
        ''' <param name="x"><c>x</c> is the relative x-offset.</param>
        ''' <param name="y"><c>y</c> is the relative y-offset.</param>
        ''' <see cref="Move"/>
        Public Sub Translate(x As Integer, y As Integer)
            Me.X += x
            Me.Y += y
        End Sub

        ''' <summary>
        ''' This method determines whether two Points have the same
        ''' location.
        ''' </summary>
        ''' <param name="o"><c>o</c> is the object to be compared to the
        ''' current object.</param>
        ''' <returns>
        ''' True if the Points have the same location and they have the
        ''' exact same type; otherwise, false.
        ''' </returns>
        ''' <seealso cref="Operator op_Equality"/>
        ''' <seealso cref="Operator op_Inequality"/>
        Public Overrides Function Equals(o As Object) As Boolean
            If o Is Nothing Then
                Return False
            End If
            If o Is Me Then
                Return True
            End If
            If Me.GetType() Is o.GetType() Then
                Dim p As Point = CType(o, Point)
                Return (X = p.X) AndAlso (Y = p.Y)
            End If
            Return False
        End Function

        ''' <summary>
        ''' Report a point's location as a string.
        ''' </summary>
        ''' <returns>
        ''' A string representing a point's location, in the form
        ''' (x,y), without any leading, training, or embedded whitespace.
        ''' </returns>
        Public Overrides Function ToString() As String
            Return "(" & X & "," & Y & ")"
        End Function

        ''' <summary>
        ''' This operator determines whether two Points have the
        ''' same location.
        ''' </summary>
        ''' <param name="p1"><c>p1</c> is the first Point to be compared.
        ''' </param>
        ''' <param name="p2"><c>p2</c> is the second Point to be compared.
        ''' </param>
        ''' <returns>
        ''' True if the Points have the same location and they 
        ''' have the exact same type; otherwise, false.
        ''' </returns>
        ''' <seealso cref="Equals"/>
        ''' <seealso cref="op_Inequality"/>
        Public Shared Operator =(p1 As Point, p2 As Point) As Boolean
            If p1 Is Nothing OrElse p2 Is Nothing Then
                Return False
            End If
            If p1.GetType() Is p2.GetType() Then
                Return (p1.X = p2.X) AndAlso (p1.Y = p2.Y)
            End If
            Return False
        End Operator

        ''' <summary>
        ''' This operator determines whether two Points have the
        ''' same location.
        ''' </summary>
        ''' <param name="p1"><c>p1</c> is the first Point to be comapred.
        ''' </param>
        ''' <param name="p2"><c>p2</c> is the second Point to be compared.
        ''' </param>
        ''' <returns>
        ''' True if the Points do not have the same location and
        ''' the exact same type; otherwise, false.
        ''' </returns>
        ''' <seealso cref="Equals"/>
        ''' <seealso cref="op_Equality"/>
        Public Shared Operator <>(p1 As Point, p2 As Point) As Boolean
            Return Not p1 = p2
        End Operator

        ''' <summary>
        ''' This is the entry point of the Point class testing program.
        ''' <para>This program tests each method and operator, and
        ''' is intended to be run after any non-trvial maintenance has
        ''' been performed on the Point class.</para>
        ''' </summary>
        Public Shared Sub Main()
            ' class test code goes here
        End Sub
    End Class
End Namespace
```

Here is the output produced when given the source code for class `Point`, shown above:

```xml
<?xml version="1.0"?>
<doc>
    <assembly>
        <name>Point</name>
    </assembly>
    <members>
        <member name="T:Graphics.Point">
            <remarks>Class <c>Point</c> models a point in a
            two-dimensional plane. </remarks>
        </member>
        <member name="F:Graphics.Point.x">
            <summary>Instance variable <c>x</c> represents the point's
            x-coordinate.</summary>
        </member>
        <member name="F:Graphics.Point.y">
            <summary>Instance variable <c>y</c> represents the point's
            y-coordinate.</summary>
        </member>
        <member name="M:Graphics.Point.#ctor">
            <summary>This constructor initializes the new Point to
            (0,0).</summary>
        </member>
        <member name="M:Graphics.Point.#ctor(System.Int32,System.Int32)">
            <summary>This constructor initializes the new Point to
            (<paramref name="x"/>,<paramref name="y"/>).</summary>
            <param><c>x</c> is the new Point's x-coordinate.</param>
            <param><c>y</c> is the new Point's y-coordinate.</param>
        </member>
        <member name="M:Graphics.Point.Move(System.Int32,System.Int32)">
            <summary>This method changes the point's location to
            the given coordinates.</summary>
            <param><c>x</c> is the new x-coordinate.</param>
            <param><c>y</c> is the new y-coordinate.</param>
            <see cref=
            "M:Graphics.Point.Translate(System.Int32,System.Int32)"/>
        </member>
        <member name=
        "M:Graphics.Point.Translate(System.Int32,System.Int32)">
            <summary>This method changes the point's location by the given
            x- and y-offsets.
            <example>For example:
            <code>
            Point p = new Point(3,5);
            p.Translate(-1,3);
            </code>
            results in <c>p</c>'s having the value (2,8).
            </example>
            </summary>
            <param><c>x</c> is the relative x-offset.</param>
            <param><c>y</c> is the relative y-offset.</param>
            <see cref="M:Graphics.Point.Move(System.Int32,System.Int32)"/>
        </member>
        <member name="M:Graphics.Point.Equals(System.Object)">
            <summary>This method determines whether two Points have the
            same location.</summary>
            <param><c>o</c> is the object to be compared to the current
            object.</param>
            <returns>True if the Points have the same location and they
            have the exact same type; otherwise, false.</returns>
            <seealso cref=
            "M:Graphics.Point.op_Equality(Graphics.Point,Graphics.Point)"
            />
            <seealso cref=
           "M:Graphics.Point.op_Inequality(Graphics.Point,Graphics.Point)"
            />
        </member>
        <member name="M:Graphics.Point.ToString">
            <summary>Report a point's location as a string.</summary>
            <returns>A string representing a point's location, in the form
            (x,y), without any leading, training, or embedded
            whitespace.</returns>
        </member>
        <member name=
        "M:Graphics.Point.op_Equality(Graphics.Point,Graphics.Point)">
            <summary>This operator determines whether two Points have the
            same location.</summary>
            <param><c>p1</c> is the first Point to be compared.</param>
            <param><c>p2</c> is the second Point to be compared.</param>
            <returns>True if the Points have the same location and they
            have the exact same type; otherwise, false.</returns>
            <seealso cref="M:Graphics.Point.Equals(System.Object)"/>
            <seealso cref=
           "M:Graphics.Point.op_Inequality(Graphics.Point,Graphics.Point)"
            />
        </member>
        <member name=
        "M:Graphics.Point.op_Inequality(Graphics.Point,Graphics.Point)">
            <summary>This operator determines whether two Points have the
            same location.</summary>
            <param><c>p1</c> is the first Point to be compared.</param>
            <param><c>p2</c> is the second Point to be compared.</param>
            <returns>True if the Points do not have the same location and
            the exact same type; otherwise, false.</returns>
            <seealso cref="M:Graphics.Point.Equals(System.Object)"/>
            <seealso cref=
            "M:Graphics.Point.op_Equality(Graphics.Point,Graphics.Point)"
            />
        </member>
        <member name="M:Graphics.Point.Main">
            <summary>This is the entry point of the Point class testing
            program.
            <para>This program tests each method and operator, and
            is intended to be run after any non-trvial maintenance has
            been performed on the Point class.</para>
            </summary>
        </member>
        <member name="P:Graphics.Point.X">
            <value>Property <c>X</c> represents the point's
            x-coordinate.</value>
        </member>
        <member name="P:Graphics.Point.Y">
            <value>Property <c>Y</c> represents the point's
            y-coordinate.</value>
        </member>
    </members>
</doc>
```
