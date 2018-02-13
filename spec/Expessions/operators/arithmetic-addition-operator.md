^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Specification]("vblang/spec/VisualBasic-Specification.md") / [Operator Expressions](Operator-Expressions.md) / [Arithmethic Operators](arithmetic-operators.md) / **Addition Operator** / 


## Addition Operator

The addition operator computes the sum of the two operands.

```antlr
AdditionOperatorExpression : Expression '+' LineTerminator? Expression ;
```

The addition operator is defined for the following types:

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, and `Long`. If integer overflow checking is on and the sum is outside the range of the result type, a `System.OverflowException` exception is thrown. Otherwise, overflows are not reported, and any significant high-order bits of the result are discarded.

* `Single` and `Double`. The sum is computed according to the rules of IEEE 754 arithmetic.

* `Decimal`. If the resulting value is too large to represent in the decimal format, a `System.OverflowException` exception is thrown. If the result value is too small to represent in the decimal format, the result is 0.

* `String`. The two `String` operands are concatenated together.

* `Date`. The `System.DateTime` type defines overloaded addition operators. Because `System.DateTime` is equivalent to the intrinsic `Date` type, these operators is also available on the `Date` type.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| __Bo__ | Sh | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __In__ |    |    |    |    |    | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 
| __Lo__ |    |    |    |    |    |    |    | Lo | De | De | Si | Do | Err | Err | Do | Ob | 
| __UL__ |    |    |    |    |    |    |    |    | UL | De | Si | Do | Err | Err | Do | Ob | 
| __De__ |    |    |    |    |    |    |    |    |    | De | Si | Do | Err | Err | Do | Ob | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Si | Do | Err | Err | Do | Ob | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do | Ob | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | St  | Err | St | Ob | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | St  | St | Ob | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | St | Ob | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |    | Ob | 

