[Lexical Grammar](Lexical-Grammar) / [Literals](Literals) / Floating-Point Literal    
Related: [Nothing Literal](Literals#Nothing-Literal), [Boolean Literal](Literals#Boolean-Literal), [Integer](Literals-Integer#Integer-Literal), **[Floating Point](Literals-FloatingPoint#Integer-Literal)**, [String](Literals-String#String-Literal), [Character](Literals-String#Character-Literal), [Date](Literals-Date#Date-Literal)

----
  
## Floating-Point Literal
```antlr
FloatingPointLiteral       :   FloatingPointLiteralValue FloatingPointTypeCharacter?
                             | IntLiteral FloatingPointTypeCharacter  ;

FloatingPointTypeCharacter :   SingleCharacter
                             | DoubleCharacter
                             | DecimalCharacter
                             | SingleTypeCharacter
                             | DoubleTypeCharacter
                             | DecimalTypeCharacter  ;

SingleCharacter            : 'F'  ;
DoubleCharacter            : 'R'  ;
DecimalCharacter           : 'D'  ;

FloatingPointLiteralValue :    IntLiteral '.' IntLiteral Exponent?
                             | '.' IntLiteral Exponent?
                             | IntLiteral Exponent  ;

Exponent                  : 'E' Sign? IntLiteral  ;

Sign : '+' | '-'  ;
```

A floating-point literal is an integer literal followed by an optional decimal point (the ASCII period character) and mantissa, and an optional base 10 exponent.
* By default, a floating-point literal is of type `System.Double`.
* If the `Single`, `Double`, or `Decimal` type character is specified, the literal is of that type.
* If a floating-point literal's type is of insufficient size to hold the floating-point literal, a compile-time error results.

__Note.__ It is worth noting that the `System.Decimal` data type can encode trailing zeros in a value.    
The specification currently makes no comment about whether trailing zeros in a `System.Decimal` literal should be honored by a compiler.

----

**Prev:** [Integer Literals](Literals-Integer) **Next:** [String Literals](Literals-String)