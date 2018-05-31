^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Specification]("vblang/spec/VisualBasic-Specification.md") / [Expressions](Expressions.md) / **Constant Expressions**


## Constant Expressions

A *constant expression* is an expression whose value can be fully evaluated at compile time.

```antlr
ConstantExpression
    : Expression
    ;
```

The type of a constant expression can be `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Char`, `Single`, `Double`, `Decimal`, `Date`, `Boolean`, `String`, `Object`, or any enumeration type. The following constructs are permitted in constant expressions:

* Literals (including `Nothing`).

* References to constant type members or constant locals.

* References to members of enumeration types.

* Parenthesized subexpressions.

* Coercion expressions, provided the target type is one of the types listed above. Coercions to and from `String` are an exception to this rule and are only allowed on null values because `String` conversions are always done in the current culture of the execution environment at run time. Note that constant coercion expressions can only ever use intrinsic conversions.

* The `+`, `-` and `Not` unary operators, provided the operand and result is of a type listed above.

* The `+`, `-`, `*`, `^`, `Mod`, `/`, `\`, `<<`, `>>`, `&`, `And`, `Or`, `Xor`, `AndAlso`, `OrElse`, `=`, `<`, `>`, `<>`, `<=`, and `=>` binary operators, provided each operand and result is of a type listed above.

* The conditional operator If, provided each operand and result is of a type listed above.

* The following run-time functions: `Microsoft.VisualBasic.Strings.ChrW`; `Microsoft.VisualBasic.Strings.Chr` if the constant value is between 0 and 128; `Microsoft.VisualBasic.Strings.AscW` if the constant string is not empty; `Microsoft.VisualBasic.Strings.Asc` if the constant string is not empty.

The following constructs are *not* permitted in constant expressions:

* Implicit binding through a `With` context.

Constant expressions of an integral type (`ULong`, `Long`, `UInteger`, `Integer`, `UShort`, `Short`, `SByte`, or `Byte`) can be implicitly converted to a narrower integral type, and constant expressions of type `Double` can be implicitly converted to `Single`, provided the value of the constant expression is within the range of the destination type. These narrowing conversions are allowed regardless of whether permissive or strict semantics are being used.
