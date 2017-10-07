[Lexical Grammar](Lexical-Grammar) / [Literals](Literals) / Floating-Point Literal    
**Related**: [Nothing Literal](Literals#Nothing-Literal), [Boolean Literal](Literals#Boolean-Literal), **[Integer](Literals-Integer#Integer-Literal)**, [Floating Point](Literals-FloatingPoint#Integer-Literal), [String](Literals-String#String-Literal), [Character](Literals-String#Character-Literal), [Date](Literals-Date#Date-Literal)   
**Sub-Topics**: [Binary](#Binary), [Octal](#Octal), [Decimal](#Decimal), [Hexadecimal](#HexaDecimal), [Digit Separator](#Digit-Separator), [Type Character](#Type-Character)

----

## Integer Literal
```antlr
IntegerLiteral        : IntegralLiteralValue IntegralTypeCharacter?  ;

IntegralLiteralValue  :   IntLiteral 
                        | HexLiteral
                        | OctalLiteral
                        | BinaryLiteral  ;

IntegralTypeCharacter :   ShortCharacter
                        | UnsignedShortCharacter
                        | IntegerCharacter
                        | UnsignedIntegerCharacter
                        | LongCharacter
                        | UnsignedLongCharacter
                        | IntegerTypeCharacter
                        | LongTypeCharacter    ;
```
Integer literals can be;-
+ [Binary](#Binary)    
+ [Octal](#Octal)
+ [Decimal](#Decimal)
+ [Hexadecimal](#Hexadecimal)
###### Binary (base 2)
```antlr
BinaryDigit   : '0' | '1'  ;
BinaryLiteral : '&' 'B' ( DigitSeparator* BinaryDigit+ )+  ;
```
  * Represe+nt the binary value of the integral literal. eg `&B1111 ' = 15`
  * A binary literal is `&B` follewed by a string of binary digits. (0-1). 
---
###### Octal (base 8)
```antlr
OctalDigit   : '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7'  ;
OctalLiteral : '&' 'O' ( DigitSeparator* OctalDigit+ )+  ;
```
   * Represe+nt the binary value of the integral literal. eg `&O77 ' = 63`
   * An octal literal is `&O` followed by a string of octal digits (0-7).
---
###### Decimal (base 10)
```antlr
     Digit : '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'  ;
IntLiteral : Digit (DigitSeparator* Digit+ )+  ;
``` 
   * Represents the decimal value of the integral literal.
   * A decimal integer literal is a string of decimal digits (0-9).

--- 
    
###### Hexadecimal (base 16)
```antrl
HexDigit    : '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' | 'A' | 'B' | 'C' | 'D' | 'E' | 'F'  ;
HexLiteral  : '&' 'H' ( DigitSeparator* HexDigit+ )+  ;
```
  * Represe+nt the binary value of the integral literal. eg `&H99 ' = 153`
  * A hexadecimal literal is `&H` followed by a string of hexadecimal digits (0-9, A-F).

-----

##### Digit Separator
```antlr
DigitSeparator : '_'  ;
``` 
   * Any number of digit separator `_` are permited before any digit, or block of digits with a integer literal.
   . eg. `&B_00000000_0000__0000___0000___000000000000`.

 * An integer literal can not end with a digit separator. eg `&B_00001_`

-----

### Type Character
```antlr
          ShortCharacter : 'S'   ;
  UnsignedShortCharacter : 'US'  ;
        IntegerCharacter : 'I'   ;
UnsignedIntegerCharacter : 'UI'  ;
           LongCharacter : 'L'   ;
   UnsignedLongCharacter : 'UL'  ;
```
The type of a literal is determined by its value or by the following type character.    
If no type character is specified,
  * values in the range of the `Integer` type are typed as `Integer`
  * values outside the range for `Integer` are typed as `Long`.
  * If an integer literal's type is of insufficient size to hold the integer literal, a compile-time error results.
  
| Character | Type | MinValue | MaxValue |
| --------- | ---- | -------- | -------- |
| `S` | Short | -32768 | +32767 | 
| `US` | Unsigned Short | 0 | +65535 |
| `I` | Integer | -2147483648 | +2147483647 |
| `UI` | Unsigned Integer | 0 | +4294967295 |
| `L` | Long | 	-9223372036854775808 | +9223372036854775807 |
| `UL` | Unsigned Long | 0 | +18446744073709551615 |

**Note.** There isn't a type character for `Byte` because the most natural character would be `B`, which is a legal character in a hexadecimal literal.

**Prev:** [Literals](Literals) **Next:** [Floating-Point Literals](Literals-FloatingPoint)