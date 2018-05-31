^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Specification]("vblang/spec/VisualBasic-Specification.md") / [Operator Expressions](Operator-Expressions.md) / [Arithmethic Operators](arithmetic-operators.md) / **Exponentiation Operator** 


## Exponentiation Operator

The exponentiation operator computes the first operand raised to the power of the second operand.

```antlr
ExponentOperatorExpression : Expression '^' LineTerminator? Expression  ;
```

The exponentiation operator is defined for type `Double`. The value is computed according to the rules of IEEE 754 arithmetic.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __SB__ |    | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __By__ |    |    | Do | Do | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __Sh__ |    |    |    | Do | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __US__ |    |    |    |    | Do | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __In__ |    |    |    |    |    | Do | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __UI__ |    |    |    |    |    |    | Do | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Do | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | Do | Do | Do | Do | Err | Err | Do  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | Do | Do | Do | Err | Err | Do  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Do | Do | Err | Err | Do  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Do | Err | Err | Do  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Do  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 
