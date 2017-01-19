# Preprocessing Directives

Once a file has been lexically analyzed, several kinds of source preprocessing occur. The most important, conditional compilation, determines which source is processed by the syntactic grammar; two other types of directives -- external source directives and region directives -- provide meta-information about the source but have no effect on compilation.

## Conditional Compilation

Conditional compilation controls whether sequences of logical lines are translated into actual code. At the beginning of conditional compilation, all logical lines are enabled; however, enclosing lines in conditional compilation statements may selectively disable those lines within the file, causing them to be ignored during the rest of the compilation process.

```antlr
CCStart
    : CCStatement*
    ;

CCStatement
    : CCConstantDeclaration
    | CCIfGroup
    | LogicalLine
    ;

CCExpression
    : LiteralExpression
    | CCParenthesizedExpression
    | CCSimpleNameExpression
    | CCCastExpression
    | CCOperatorExpression
    | CCConditionalExpression
    ;

CCParenthesizedExpression
    : '(' CCExpression ')'
    ;

CCSimpleNameExpression
    : Identifier
    ;

CCCastExpression
    : 'DirectCast' '(' CCExpression ',' TypeName ')'
    | 'TryCast' '(' CCExpression ',' TypeName ')'
    | 'CType' '(' CCExpression ',' TypeName ')'
    | CastTarget '(' CCExpression ')'
    ;

CCOperatorExpression
    : CCUnaryOperator CCExpression
    | CCExpression CCBinaryOperator CCExpression
    ;

CCUnaryOperator
    : '+' | '-' | 'Not'
    ;

CCBinaryOperator
    : '+'     | '-'     | '*'   | '/'       | '\\'     | 'Mod' | '^' | '='
    | '<' '>' | '<'     | '>'   | '<' '='   | '>' '='  | '&'
    | 'And'   | 'Or'    | 'Xor' | 'AndAlso' | 'OrElse'
    | '<' '<' | '>' '>'
    ;

CCConditionalExpression
    : 'If' '(' CCExpression ',' CCExpression ',' CCExpression ')'
    | 'If' '(' CCExpression ',' CCExpression ')'
    ;
```


For example, the program

```vb
#Const A = True
#Const B = False

Class C

#If A Then
    Sub F()
    End Sub
#Else
    Sub G()
    End Sub
#End If

#If B Then
    Sub H()
    End Sub
#Else
    Sub I()
    End Sub
#End If

End Class
```

produces the exact same sequence of tokens as the program

```vb
Class C
    Sub F()
    End Sub

    Sub I()
    End Sub
End Class
```

The constant expressions allowed in conditional compilation directives are a subset of general constant expressions.

The preprocessor allows whitespace and explicit line continuations before and after every token.


### Conditional Constant Directives

Conditional constant statements define constants that exist in a separate conditional compilation declaration space scoped to the source file.

```antlr
CCConstantDeclaration
    : '#' 'Const' Identifier '=' CCExpression LineTerminator
    ;
```

The declaration space is special in that no explicit declaration of conditional compilation constants is necessary -- conditional constants can be implicitly defined in a conditional compilation directive.

Prior to being assigned a value, a conditional compilation constant has the value `Nothing`. When a conditional compilation constant is assigned a value, which must be a constant expression, the type of the constant becomes the type of the value being assigned to it. A conditional compilation constant may be redefined multiple times throughout a source file.

For example, the following code prints only the string `about to print value` and the value of `Test`.

```vb
Module M1
    Sub PrintValue(Test As Integer)

#Const DebugCode = True

#If DebugCode Then
        Console.WriteLine("about to print value")
#End If

#Const DebugCode = False

        Console.WriteLine(Test)

#If DebugCode Then
        Console.WriteLine("printed value")
#End If

    End Sub
End Module
```

The compilation environment may also define conditional constants in a conditional compilation declaration space.


### Conditional Compilation Directives

Conditional compilation directives control conditional compilation.

```antlr
CCIfGroup
    : '#' 'If' CCExpression 'Then'? LineTerminator CCStatement*
    CCElseIfGroup* CCElseGroup? '#' 'End' 'If' LineTerminator
    ;

CCElseIfGroup
    : '#' ElseIf CCExpression 'Then'? LineTerminator CCStatement*
    ;

CCElseGroup
    : '#' 'Else' LineTerminator CCStatement*
    ;
```

Conditional constants can only reference constant expressions and conditional compilation constants. Each of the constant expressions within a single conditional compilation group is evaluated and converted to the `Boolean` type in textual order from first to last until one of the conditional expressions evaluates to `True`. If an expression is not convertible to `Boolean`, a compile-time error results. Permissive semantics and binary string comparisons are always used when evaluating conditional compilation constant expressions, regardless of any `Option` directives or compilation environment settings.

All lines enclosed by the group, including nested conditional compilation directives, are disabled except for lines between the statement containing the `True` expression and the next conditional statement of the group, or lines between the `Else` statement and the `End If` statement if an `Else` appears in the group and all of the expressions evaluate to `False`.

In this example, the call to `WriteToLog` in the `Trace` conditional compilation directive is not processed because the surrounding `Debug` conditional compilation directive evaluates to `False`.

```vb
#Const Debug = False   ' Debugging off
#Const Trace = True    ' Tracing on

Class PurchaseTransaction
    Sub Commit()

#If Debug Then
        CheckConsistency()
#If Trace Then
        WriteToLog(Me.ToString())
#End If
#End If
        ...
    End Sub
End Class
```


## External Source Directives

A source file may include external source directives that indicate a mapping between source lines and text external to the source.

```antlr
ESDStart
    : ExternalSourceStatement*
    ;

ExternalSourceStatement
    : ExternalSourceGroup
    | LogicalLine
    ;

ExternalSourceGroup
    : '#' 'ExternalSource' '(' StringLiteral ',' IntLiteral ')' LineTerminator
      LogicalLine* '#' 'End' 'ExternalSource' LineTerminator
    ;
```

External source directives have no effect on compilation and may not be nested. For example:

```vb
Module Test
    Sub Main()

#ExternalSource("c:\wwwroot\inetpub\test.aspx", 30)
        Console.WriteLine("In test.aspx")
#End ExternalSource

    End Sub
End Module
```


## Region Directives

Region directives group lines of source code but have no other effect on compilation. The entire group can be collapsed and hidden, or expanded and viewed, in the integrated development environment (IDE).

```antlr
RegionStart
    : RegionStatement*
    ;

RegionStatement
    : RegionGroup
    | LogicalLine
    ;

RegionGroup
    : '#' 'Region' StringLiteral LineTerminator
      RegionStatement*
      '#' 'End' 'Region' LineTerminator
    ;
```

Regions may be nested. Region directives are special in that they can neither start nor terminate within a method body, and they must respect the block structure of the program. For example:

```vb
Module Test
#Region "Startup code - do not edit"
    Sub Main()
    End Sub
#End Region

End Module


' Error due to Region directives breaking the block structure
Class C
#Region "Fred"
End Class
#End Region
```


## External Checksum Directives

A source file may include an external checksum directive that indicates what checksum should be emitted for a file referenced in an external source directive. In all other respects external source directives have no effect on compilation.

```antlr
ExternalChecksumStart
    : ExternalChecksumStatement*
    ;

ExternalChecksumStatement
    : '#' 'ExternalChecksum' '('
      StringLiteral ',' StringLiteral ',' StringLiteral
      ')' LineTerminator
    ;
```

An external checksum directive contains the filename of the external file, a globally unique identifier (GUID) associated with the file and the checksum for the file. The GUID is specified as a string constant of the form "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}", where x is a hexadecimal digit. The checksum is specified as a string constant of the form "xxxx...", where x is a hexadecimal digit. The number of digits in a checksum must be an even number.

An external file may have multiple external checksum directives associated with it provided that all of the GUID and checksum values match exactly. If the name of the external file matches the name of a file being compiled, the checksum is ignored in favor of the compiler's checksum calculation.

For example:

```vb
#ExternalChecksum("c:\wwwroot\inetpub\test.aspx", _
    "{12345678-1234-1234-1234-123456789abc}", _
    "1a2b3c4e5f617239a49b9a9c0391849d34950f923fab9484")

Module Test
    Sub Main()

#ExternalSource("c:\wwwroot\inetpub\test.aspx", 30)
        Console.WriteLine("In test.aspx")
#End ExternalSource

    End Sub
End Module
```

