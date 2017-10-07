^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Specification]("vblang/spec/VisualBasic-Specification.md") / [Lexical Grammar](Lexical-Grammar) / **Literals**

---

## Literals
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
A *literal* is a textual representation of a particular value of a type.
* Literals types include
  * **[Nothing](#Nothing-Literal)**
  * **[Boolean](#Boolean-Literal)**
  * [Integer](Literals-Integer#Integer-Literal)    
  * [Floating Point](Literals-FloatingPoint#Integer-Literal)
  * [String](Literals-String#String-Literal)
  * [Character](Literals-String#Character-Literal)
  * [Date](Literals-Date#Date-Literal)
---

#### Nothing Literal

`Nothing` is a special literal; it does not have a type and is convertible to all types in the type system, including type parameters. When converted to a particular type, it is the equivalent of the default value of that type.

```antlr
Nothing :   'Nothing'  ;
```
**Examples**
```vbnet
Dim x As Integer = Nothing ' x = 0
Dim s As String  = Nothing ' s = Nothing / No Reference to a string
Dim n As Integer? = Nothing ' n = New Integer?()
```

-------------------

#### Boolean Literal

`True` and `False` are literals of the `Boolean` type that map to the true and false state, respectively.

```antlr
BooleanLiteral : 'True' | 'False'  ;
```

----

**Next:** [Integer Literals](Literals-Integer)