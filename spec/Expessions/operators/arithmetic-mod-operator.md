^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Specification]("vblang/spec/VisualBasic-Specification.md") / [Operator Expressions](Operator-Expressions.md) / **Mod Operators**


## Mod Operator

The `Mod` (modulo) operator computes the remainder of the division between two operands.

```antlr
ModuloOperatorExpression : Expression 'Mod' LineTerminator? Expression  ;
```

The `Mod` operator is defined for the following types:

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong` and `Long`. The result of `x Mod y` is the value produced by `x - (x \ y) * y`. If `y` is zero, a `System.DivideByZeroException` exception is thrown. The modulo operator never causes an overflow.

* `Single` and `Double`. The remainder is computed according to the rules of IEEE 754 arithmetic.

* `Decimal`. If the value of the right operand is zero, a `System.DivideByZeroException` exception is thrown. If the resulting value is too large to represent in the decimal format, a `System.OverflowException` exception is thrown. If the result value is too small to represent in the decimal format, the result is zero.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Sh | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __In__ |    |    |    |    |    | In | Lo | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Lo | De | De | Si | Do | Err | Err | Do  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | UL | De | Si | Do | Err | Err | Do  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | De | Si | Do | Err | Err | Do  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Si | Do | Err | Err | Do  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Do  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 

