[Lexical Grammar](Lexical-Grammar) / [Literals](Literals) / String Literal    
**Related**: [Nothing Literal](Literals#Nothing-Literal), [Boolean Literal](Literals#Boolean-Literal), [Integer](Literals-Integer#Integer-Literal), [Floating Point](Literals-FloatingPoint#Integer-Literal), **[String Literal](#String=Literal)**, **[Character Literal](#Character-Literal)**    
**Sub-Topics**: [String Literal](#String=Literal), [Character Literal](#Character-Literal)

----

## String Literal
```antlr
       StringLiteral :   DoubleQuoteCharacter StringCharacter* DoubleQuoteCharacter  ;
DoubleQuoteCharacter :   '"'
                       | '<unicode left double-quote 0x201c>'
                       | '<unicode right double-quote 0x201D>'  ;

     StringCharacter :   '<Any character except DoubleQuoteCharacter>'
                       | DoubleQuoteCharacter DoubleQuoteCharacter  ;
```
A string literal is a sequence of zero or more Unicode characters beginning and ending with an ASCII double-quote character, a Unicode left double-quote character, or a Unicode right double-quote character. Within a string, a sequence of two double-quote characters is an escape sequence representing a double quote in the string.


A string constant is of the `System.String` type.

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

The compiler is allowed to replace a constant string expression with a string literal.

Each string literal does not necessarily result in a new string instance. When two or more string literals that are equivalent according to the string equality operator using binary comparison semantics appear in the same program, these string literals may refer to the same string instance.   

For instance, the output of the following program may return `True` because the two literals may refer to the same string instance.

**Example**
```vb
Module Test
    Sub Main()
        Dim a As Object = "he" & "llo"
        Dim b As Object = "hello"
        Console.WriteLine(a Is b)
    End Sub
End Module
```

-----

## Character Literal
```antlr
CharacterLiteral : DoubleQuoteCharacter StringCharacter DoubleQuoteCharacter 'C'  ;
```
A character literal represents a single Unicode character of the `System.Char` type.
 * Two double-quote characters is an escape sequence representing the double-quote character.


**Example**

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

