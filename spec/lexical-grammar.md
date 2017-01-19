# Lexical Grammar

Compilation of a Visual Basic program first involves translating the raw stream of Unicode characters into an ordered set of lexical tokens. Because the Visual Basic language is not free-format, the set of tokens is then further divided into a series of logical lines. A *logical line* spans from either the start of the stream or a line terminator through to the next line terminator that is not preceded by a line continuation or through to the end of the stream.

__Note.__ With the introduction of XML literal expressions in version 9.0 of the language, Visual Basic no longer has a distinct lexical grammar in the sense that Visual Basic code can be tokenized without regard to the syntactic context. This is due to the fact that XML and Visual Basic have different lexical rules and the set of lexical rules in use at any particular time depends on what syntactic construct is being processed at that moment. This specification retains this lexical grammar section as a guide to the lexical rules of regular Visual Basic code.

```antlr
LogicalLineStart
    : LogicalLine*
    ;

LogicalLine
    : LogicalLineElement* Comment? LineTerminator
    ;

LogicalLineElement
    : WhiteSpace
    | LineContinuation
    | Token
    ;

Token
    : Identifier
    | Keyword
    | Literal
    | Separator
    | Operator
    ;
```

## Characters and Lines

Visual Basic programs are composed of characters from the Unicode character set.

```antlr
Character:
    '<Any Unicode character except a LineTerminator>'
    ;
```

### Line Terminators

Unicode line break characters separate logical lines.

```antlr
LineTerminator
    : '<Unicode 0x00D>'
    | '<Unicode 0x00A>'
    | '<CR>'
    | '<LF>'
    | '<Unicode 0x2028>'
    | '<Unicode 0x2029>'
    ;
```

### Line Continuation

A *line continuation* consists of at least one white-space character that immediately precedes a single underscore character as the last character (other than white space) in a text line. A line continuation allows a logical line to span more than one physical line. Line continuations are treated as if they were white space, even though they are not.

```antlr
LineContinuation
    : WhiteSpace '_' WhiteSpace* LineTerminator
    ;
```

The following program shows some line continuations:

```vb
Module Test
    Sub Print( _
        Param1 As Integer, _
        Param2 As Integer )

        If (Param1 < Param2) Or _
            (Param1 > Param2) Then
            Console.WriteLine("Not equal")
        End If
    End Function
End Module
```

Some places in the syntactic grammar allow for *implicit line continuations*. When a line terminator is encountered

* after a comma (`,`), open parenthesis (`(`), open curly brace (`{`), or open embedded expression (`<%=`)

* after a member qualifier (`.` or `.@` or `...`), provided that something is being qualified (i.e. is not using an implicit `With` context)

* before a close parenthesis (`)`), close curly brace (`}`), or close embedded expression (`%>`)

* after a less-than (`<`) in an attribute context

* before a greater-than (`>`) in an attribute context

* after a greater-than (`>`) in a non-file-level attribute context

* before and after query operators (`Where`, `Order`, `Select`, etc.)

* after binary operators (`+`, `-`, `/`, `*`, etc.) in an expression context

* after assignment operators (`=`, `:=`, `+=`, `-=`, etc.) in any context.

the line terminator is treated as if it was a line continuation.

```antlr
Comma
    : ',' LineTerminator?
    ;

Period
    : '.' LineTerminator?
    ;

OpenParenthesis
    : '(' LineTerminator?
    ;

CloseParenthesis
    : LineTerminator? ')'
    ;

OpenCurlyBrace
    : '{' LineTerminator?
    ;

CloseCurlyBrace
    : LineTerminator? '}'
    ;

Equals
    : '=' LineTerminator?
    ;

ColonEquals
    : ':' '=' LineTerminator?
    ;
```

For example, the previous example could also be written as:

```vb
Module Test
    Sub Print(
        Param1 As Integer,
        Param2 As Integer)

        If (Param1 < Param2) Or
            (Param1 > Param2) Then
            Console.WriteLine("Not equal")
        End If
    End Function
End Module
```

Implicit line continuations will only ever be inferred directly before or after the specified token. They will not be inferred before or after a line continuation. For example:

```vb
Dim y = 10
' Error: Expression expected for assignment to x
Dim x = _

y
```

Line continuations will not be inferred in conditional compilation contexts. (__Note.__ This last restriction is required because text in conditional compilation blocks that are not compiled do not have to be syntactically valid. Thus, text in the block might accidentally get "picked up" by the conditional compilation statement, especially as the language gets extended in the future.)


### White Space

*White space* serves only to separate tokens and is otherwise ignored. Logical lines containing only white space are ignored. (__Note.__
Line terminators are not considered white space.)

```antlr
WhiteSpace
    : '<Unicode class Zs>'
    | '<Unicode Tab 0x0009>'
    ;
```

### Comments

A *comment* begins with a single-quote character or the keyword `REM`. A single-quote character is either an ASCII single-quote character, a Unicode left single-quote character, or a Unicode right single-quote character. Comments can begin anywhere on a source line, and the end of the physical line ends the comment. The compiler ignores the characters between the beginning of the comment and the line terminator. Consequently, comments cannot extend across multiple lines by using line continuations.

```antlr
Comment
    : CommentMarker Character*
    ;

CommentMarker
    : SingleQuoteCharacter
    | 'REM'
    ;

SingleQuoteCharacter
    : '\''
    | '<Unicode 0x2018>'
    | '<Unicode 0x2019>'
    ;
```

## Identifiers

An *identifier* is a name. Visual Basic identifiers conform to the Unicode Standard Annex 15 with one exception: identifiers may begin with an underscore (connector) character. If an identifier begins with an underscore, it must contain at least one other valid identifier character to disambiguate it from a line continuation.

```antlr
Identifier
    : NonEscapedIdentifier TypeCharacter?
    | Keyword TypeCharacter
    | EscapedIdentifier
    ;

NonEscapedIdentifier
    : '<Any IdentifierName but not Keyword>'
    ;

EscapedIdentifier
    : '[' IdentifierName ']'
    ;

IdentifierName
    : IdentifierStart IdentifierCharacter*
    ;

IdentifierStart
    : AlphaCharacter
    | UnderscoreCharacter IdentifierCharacter
    ;

IdentifierCharacter
    : UnderscoreCharacter
    | AlphaCharacter
    | NumericCharacter
    | CombiningCharacter
    | FormattingCharacter
    ;

AlphaCharacter
    : '<Unicode classes Lu,Ll,Lt,Lm,Lo,Nl>'
    ;

NumericCharacter
    : '<Unicode decimal digit class Nd>'
    ;

CombiningCharacter
    : '<Unicode combining character classes Mn, Mc>'
    ;

FormattingCharacter
    : '<Unicode formatting character class Cf>'
    ;

UnderscoreCharacter
    : '<Unicode connection character class Pc>'
    ;

IdentifierOrKeyword
    : Identifier
    | Keyword
    ;
```

Regular identifiers may not match keywords, but escaped identifiers or identifiers with a type character can. An *escaped identifier* is an identifier delimited by square brackets. Escaped identifiers follow the same rules as regular identifiers except that they may match keywords and may not have type characters.

This example defines a class named `class` with a shared method named `shared` that takes a parameter named `boolean` and then calls the method.

```vb
Class [class]
    Shared Sub [shared]([boolean] As Boolean)
        If [boolean] Then
            Console.WriteLine("true")
        Else
            Console.WriteLine("false")
        End If
    End Sub
End Class

Module [module]
    Sub Main()
        [class].[shared](True)
    End Sub
End Module
```

Identifiers are case insensitive, so two identifiers are considered to be the same identifier if they differ only in case. (__Note.__ The Unicode Standard one-to-one case mappings are used when comparing identifiers and any locale-specific case mappings are ignored.)


### Type Characters

A *type character* denotes the type of the preceding identifier. The type character is not considered part of the identifier.

```antlr
TypeCharacter
    : IntegerTypeCharacter
    | LongTypeCharacter
    | DecimalTypeCharacter
    | SingleTypeCharacter
    | DoubleTypeCharacter
    | StringTypeCharacter
    ;

IntegerTypeCharacter
    : '%'
    ;

LongTypeCharacter
    : '&'
    ;

DecimalTypeCharacter
    : '@'
    ;

SingleTypeCharacter
    : '!'
    ;

DoubleTypeCharacter
    : '#'
    ;

StringTypeCharacter
    : '$'
    ;
```

If a declaration includes a type character, the type character must agree with the type specified in the declaration itself; otherwise, a compile-time error occurs. If the declaration omits the type (for example, if it does not specify an `As` clause), the type character is implicitly substituted as the type of the declaration.

No white space may come between an identifier and its type character. There are no type characters for `Byte`, `SByte`, `UShort`, `Short`, `UInteger` or `ULong`, due to a lack of suitable characters.

Appending a type character to an identifier that conceptually does not have a type (for example, a namespace name) or to an identifier whose type disagrees with the type of the type character causes a compile-time error.

The following example shows the use of type characters:

```vb
' The follow line will cause an error: standard modules have no type.
Module Test1#
End Module

Module Test2

    ' This function takes a Long parameter and returns a String.
    Function Func$(Param&)

        ' The following line causes an error because the type character
        ' conflicts with the declared type of Func and Param.
        Func# = CStr(Param@)

        ' The following line is valid.
        Func$ = CStr(Param&)
    End Function
End Module
```

The type character `!` presents a special problem in that it can be used both as a type character and as a separator in the language. To remove ambiguity, a `!` character is a type character as long as the character that follows it cannot start an identifier. If it can, then the `!` character is a separator, not a type character.


## Keywords

A *keyword* is a word that has special meaning in a
language construct. All keywords are reserved by the language and may not be used as identifiers unless the identifiers are escaped. (__Note.__ `EndIf`, `GoSub`, `Let`, `Variant`, and `Wend` are retained as keywords, although they are no longer used in Visual Basic.)

```antlr
Keyword
    : 'AddHandler'      | 'AddressOf'      | 'Alias'       | 'And'
    | 'AndAlso'         | 'As'             | 'Boolean'     | 'ByRef'
	| 'Byte'            | 'ByVal'          | 'Call'        | 'Case'        
	| 'Catch'           | 'CBool'          | 'CByte'       | 'CChar'       
	| 'CDate'           | 'CDbl'           | 'CDec'        | 'Char'        
	| 'CInt'            | 'Class'          | 'CLng'        | 'CObj'        
	| 'Const'           | 'Continue'       | 'CSByte'      | 'CShort'      
	| 'CSng'            | 'CStr'           | 'CType'       | 'CUInt'       
	| 'CULng'           | 'CUShort'        | 'Date'        | 'Decimal'     
	| 'Declare'         | 'Default'        | 'Delegate'    | 'Dim'         
	| 'DirectCast'      | 'Do'             | 'Double'      | 'Each'        
	| 'Else'            | 'ElseIf'         | 'End'         | 'EndIf'       
	| 'Enum'            | 'Erase'          | 'Error'       | 'Event'       
	| 'Exit'            | 'False'          | 'Finally'     | 'For'         
	| 'Friend'          | 'Function'       | 'Get'         | 'GetType'     
	| 'GetXmlNamespace' | 'Global'         | 'GoSub'       | 'GoTo'        
	| 'Handles'         | 'If'             | 'Implements'  | 'Imports'     
	| 'In'              | 'Inherits'       | 'Integer'     | 'Interface'   
	| 'Is'              | 'IsNot'          | 'Let'         | 'Lib'         
	| 'Like'            | 'Long'           | 'Loop'        | 'Me'          
	| 'Mod'             | 'Module'         | 'MustInherit' | 'MustOverride'
	| 'MyBase'          | 'MyClass'        | 'Namespace'   | 'Narrowing'   
	| 'New'             | 'Next'           | 'Not'         | 'Nothing'     
	| 'NotInheritable'  | 'NotOverridable' | 'Object'      | 'Of'          
	| 'On'              | 'Operator'       | 'Option'      | 'Optional'    
	| 'Or'              | 'OrElse'         | 'Overloads'   | 'Overridable' 
	| 'Overrides'       | 'ParamArray'     | 'Partial'     | 'Private'     
	| 'Property'        | 'Protected'      | 'Public'      | 'RaiseEvent'  
	| 'ReadOnly'        | 'ReDim'          | 'REM'         | 'RemoveHandler'
	| 'Resume'          | 'Return'         | 'SByte'       | 'Select'      
	| 'Set'             | 'Shadows'        | 'Shared'      | 'Short'       
	| 'Single'          | 'Static'         | 'Step'        | 'Stop'        
	| 'String'          | 'Structure'      | 'Sub'         | 'SyncLock'    
	| 'Then'            | 'Throw'          | 'To'          | 'True'        
	| 'Try'             | 'TryCast'        | 'TypeOf'      | 'UInteger'    
	| 'ULong'           | 'UShort'         | 'Using'       | 'Variant'     
	| 'Wend'            | 'When'           | 'While'       | 'Widening'    
	| 'With'            | 'WithEvents'     | 'WriteOnly'   | 'Xor'         
    ;
```

## Literals

A *literal* is a textual representation of a particular value of a type. Literal types include Boolean, integer, floating point, string, character, and date.

```antlr
Literal
    : BooleanLiteral
    | IntegerLiteral
    | FloatingPointLiteral
    | StringLiteral
    | CharacterLiteral
    | DateLiteral
    | Nothing
    ;
```

### Boolean Literals

`True` and `False` are literals of the `Boolean` type that map to the true and false state, respectively.

```antlr
BooleanLiteral
    : 'True' | 'False'
    ;
```

### Integer Literals

Integer literals can be decimal (base 10), hexadecimal (base 16), or octal (base 8). A decimal integer literal is a string of decimal digits (0-9). A hexadecimal literal is `&H` followed by a string of hexadecimal digits (0-9, A-F). An octal literal is `&O` followed by a string of octal digits (0-7). Decimal literals directly represent the decimal value of the integral literal, whereas octal and hexadecimal literals represent the binary value of the integer literal (thus, `&H8000S` is -32768, not an overflow error).

```antlr
IntegerLiteral
    : IntegralLiteralValue IntegralTypeCharacter?
    ;

IntegralLiteralValue
    : IntLiteral
    | HexLiteral
    | OctalLiteral
    ;

IntegralTypeCharacter
    : ShortCharacter
    | UnsignedShortCharacter
    | IntegerCharacter
    | UnsignedIntegerCharacter
    | LongCharacter
    | UnsignedLongCharacter
    | IntegerTypeCharacter
    | LongTypeCharacter
    ;

ShortCharacter
    : 'S'
    ;

UnsignedShortCharacter
    : 'US'
    ;

IntegerCharacter
    : 'I'
    ;

UnsignedIntegerCharacter
    : 'UI'
    ;

LongCharacter
    : 'L'
    ;

UnsignedLongCharacter
    : 'UL'
    ;

IntLiteral
    : Digit+
    ;

HexLiteral
    : '&' 'H' HexDigit+
    ;

OctalLiteral
    : '&' 'O' OctalDigit+
    ;

Digit
    : '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
    ;

HexDigit
    : '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
    | 'A' | 'B' | 'C' | 'D' | 'E' | 'F'
    ;

OctalDigit
    : '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7'
    ;
```

The type of a literal is determined by its value or by the following type character. If no type character is specified, values in the range of the `Integer` type are typed as `Integer`; values outside the range for `Integer` are typed as `Long`. If an integer literal's type is of insufficient size to hold the integer literal, a compile-time error results. (__Note.__ There isn't a type character for `Byte` because the most natural character would be `B`, which is a legal character in a hexadecimal literal.)


### Floating-Point Literals

A floating-point literal is an integer literal followed by an optional decimal point (the ASCII period character) and mantissa, and an optional base 10 exponent. By default, a floating-point literal is of type `Double`. If the `Single`, `Double`, or `Decimal` type character is specified, the literal is of that type. If a floating-point literal's type is of insufficient size to hold the floating-point literal, a compile-time error results.

__Note.__ It is worth noting that the `Decimal` data type can encode trailing zeros in a value. The specification currently makes no comment about whether trailing zeros in a `Decimal` literal should be honored by a compiler.

```antlr
FloatingPointLiteral
    : FloatingPointLiteralValue FloatingPointTypeCharacter?
    | IntLiteral FloatingPointTypeCharacter
    ;

FloatingPointTypeCharacter
    : SingleCharacter
    | DoubleCharacter
    | DecimalCharacter
    | SingleTypeCharacter
    | DoubleTypeCharacter
    | DecimalTypeCharacter
    ;

SingleCharacter
    : 'F'
    ;

DoubleCharacter
    : 'R'
    ;

DecimalCharacter
    : 'D'
    ;

FloatingPointLiteralValue
    : IntLiteral '.' IntLiteral Exponent?
    | '.' IntLiteral Exponent?
    | IntLiteral Exponent
    ;

Exponent
    : 'E' Sign? IntLiteral
    ;

Sign
    : '+'
    | '-'
    ;
```

### String Literals

A string literal is a sequence of zero or more Unicode characters beginning and ending with an ASCII double-quote character, a Unicode left double-quote character, or a Unicode right double-quote character. Within a string, a sequence of two double-quote characters is an escape sequence representing a double quote in the string.

```antlr
StringLiteral
    : DoubleQuoteCharacter StringCharacter* DoubleQuoteCharacter
    ;

DoubleQuoteCharacter
    : '"'
    | '<unicode left double-quote 0x201c>'
    | '<unicode right double-quote 0x201D>'
    ;

StringCharacter
    : '<Any character except DoubleQuoteCharacter>'
    | DoubleQuoteCharacter DoubleQuoteCharacter
    ;
```

A string constant is of the `String` type.

```vb
Module Test
    Sub Main()

        ' This prints out: ".
        Console.WriteLine("""")

        ' This prints out: a"b.
        Console.WriteLine("a""b")

        ' This causes a compile error due to mismatched double-quotes.
        Console.WriteLine("a"b")
    End Sub
End Module
```

The compiler is allowed to replace a constant string expression with a string literal. Each string literal does not necessarily result in a new string instance. When two or more string literals that are equivalent according to the string equality operator using binary comparison semantics appear in the same program, these string literals may refer to the same string instance. For instance, the output of the following program may return `True` because the two literals may refer to the same string instance.

```vb
Module Test
    Sub Main()
        Dim a As Object = "he" & "llo"
        Dim b As Object = "hello"
        Console.WriteLine(a Is b)
    End Sub
End Module
```


### Character Literals

A character literal represents a single Unicode character of the `Char` type. Two double-quote characters is an escape sequence representing the double-quote character.

```antlr
CharacterLiteral
    : DoubleQuoteCharacter StringCharacter DoubleQuoteCharacter 'C'
    ;
```


```vb
Module Test
    Sub Main()

        ' This prints out: a.
        Console.WriteLine("a"c)

        ' This prints out: ".
        Console.WriteLine(""""c)
    End Sub
End Module
```


### Date Literals

A date literal represents a particular moment in time expressed as a value of the `Date` type.

```antlr
DateLiteral
    : '#' WhiteSpace* DateOrTime WhiteSpace* '#'
    ;

DateOrTime
    : DateValue WhiteSpace+ TimeValue
    | DateValue
    | TimeValue
    ;

DateValue
    : MonthValue '/' DayValue '/' YearValue
    | MonthValue '-' DayValue '-' YearValue
    ;

TimeValue
    : HourValue ':' MinuteValue ( ':' SecondValue )? WhiteSpace* AMPM?
    | HourValue WhiteSpace* AMPM
    ;

MonthValue
    : IntLiteral
    ;

DayValue
    : IntLiteral
    ;

YearValue
    : IntLiteral
    ;

HourValue
    : IntLiteral
    ;

MinuteValue
    : IntLiteral
    ;

SecondValue
    : IntLiteral
    ;

AMPM
    : 'AM' | 'PM'
    ;

ElseIf
    : 'ElseIf'
    | 'Else' 'If'
    ;
```

The literal may specify both a date and a time, just a date, or just a time. If the date value is omitted, then January 1 of the year 1 in the Gregorian calendar is assumed. If the time value is omitted, then 12:00:00 AM is assumed.

To avoid problems with interpreting the year value in a date value, the year value cannot be two digits. When expressing a date in the first century AD/CE, leading zeros must be specified.

A time value may be specified either using a 24-hour value or a 12-hour value; time values that omit an `AM` or `PM` are assumed to be 24-hour values. If a time value omits the minutes, the literal `0` is used by default. If a time value omits the seconds, the literal `0` is used by default. If both minutes and second are omitted, then `AM` or `PM` must be specified. If the date value specified is outside the range of the `Date` type, a compile-time error occurs.

The following example contains several date literals.

```vb
Dim d As Date

d = # 8/23/1970 3:45:39AM #
d = # 8/23/1970 #              ' Date value: 8/23/1970 12:00:00AM.
d = # 3:45:39AM #              ' Date value: 1/1/1 3:45:39AM.
d = # 3:45:39 #                ' Date value: 1/1/1 3:45:39AM.
d = # 13:45:39 #               ' Date value: 1/1/1 1:45:39PM.
d = # 1AM #                    ' Date value: 1/1/1 1:00:00AM.
d = # 13:45:39PM #             ' This date value is not valid.
```


### Nothing

`Nothing` is a special literal; it does not have a type and is convertible to all types in the type system, including type parameters. When converted to a particular type, it is the equivalent of the default value of that type.

```antlr
Nothing
    : 'Nothing'
    ;
```

## Separators

The following ASCII characters are separators:

```antlr
Separator
    : '(' | ')' | '{' | '}' | '!' | '#' | ',' | '.' | ':' | '?'
    ;
```

## Operator Characters

The following ASCII characters or character sequences denote operators:

```antlr
Operator
    : '&' | '*' | '+' | '-' | '/' | '\\' | '^' | '<' | '=' | '>'
    ;
```

