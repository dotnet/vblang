^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Specification]("vblang/spec/VisualBasic-Specification.md") / [Operator Expressions](Operator-Expressions.md) / **Arthimetic Operators**


## Arithmetic Operators

  * Unary Operators
    * [Unary Plus Operator](#Unary-Plus-Operator)  
    * [Unary Minus Operator](#Unary-Minus-Operator)
  * Binary Operators
    * [Addition Operator](arithmetic-addition-operator.md)
    * [Subtraction Operator](arithmetic-subtraction-operator.md)
    * [Multiplication Operator](arithmetic-multiplication-operator.md)
    * [Division Operator](arithmetic-division-operator.md)
    * [Mod Operator](arithmetic-mod-operator.md)
    * [Exponentiation Operator](arithmetic-exponentiation-operator.md)

The `*`, `/`, `\`, `^`, `Mod`, `+`, and `-` operators are the *arithmetic operators*.

```antlr
ArithmeticOperatorExpression : UnaryPlusExpression
                             | UnaryMinusExpression
                             | AdditionOperatorExpression
                             | SubtractionOperatorExpression
                             | MultiplicationOperatorExpression
                             | DivisionOperatorExpression
                             | ModuloOperatorExpression
                             | ExponentOperatorExpression
                             ;
```

Floating-point arithmetic operations may be performed with higher precision than the result type of the operation. For example, some hardware architectures support an "extended" or "long double" floating-point type with greater range and precision than the `Double` type, and implicitly perform all floating-point operations using this higher-precision type. Hardware architectures can be made to perform floating-point operations with less precision only at excessive cost in performance; rather than require an implementation to forfeit both performance and precision, Visual Basic allows the higher-precision type to be used for all floating-point operations. Other than delivering more precise results, this rarely has any measurable effects. However, in expressions of the form `x * y / z`, where the multiplication produces a result that is outside the `Double` range, but the subsequent division brings the temporary result back into the `Double` range, the fact that the expression is evaluated in a higher-range format may cause a finite result to be produced instead of infinity.


### Unary Plus Operator

```antlr
UnaryPlusExpression : '+' Expression ;
```

The unary plus operator is defined for the `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Single`, `Double`, and `Decimal` types.

__Operation Type:__


| __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| Sh | SB | By | Sh | US | In | UI | Lo | UL | De | Si | Do | Err | Err | Do | Ob | 


### Unary Minus Operator

```antlr
UnaryMinusExpression : '-' Expression ;
```

The unary minus operator is defined for the following types:

`SByte`, `Short`, `Integer`, and `Long`. The result is computed by subtracting the operand from zero. If integer overflow checking is on and the value of the operand is the maximum negative `SByte`, `Short`, `Integer`, or `Long`, a `System.OverflowException` exception is thrown. Otherwise, if the value of the operand is the maximum negative `SByte`, `Short`, `Integer`, or `Long`, the result is that same value, and the overflow is not reported.

`Single` and `Double`. The result is the value of the operand with its sign inverted, including the values 0 and Infinity. If the operand is NaN, the result is also NaN.

`Decimal`. The result is computed by subtracting the operand from zero.

__Operation Type:__

| __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| Sh | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Do | Ob | 

