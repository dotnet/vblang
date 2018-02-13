^*Copyright (c) Microsoft. All Rights Reserved. Licensed under the Apache License, Version 2.0.  See [License.txt](https://github.com/dotnet/roslyn/blob/master/License.txt) for license information.*^    
[Specification]("vblang/spec/VisualBasic-Specification.md") / [Expressions]("vblang/spec/Expressions/expressions.md") / **Operator Expressions**    
^**Topics:**    
*[Operator Precedence and Associativity]("vblang/spec/Expressions/operators/operator-expressions.md#operator-precedence-and-associativity") | [Object Operands]("vblang/spec/Expressions/operators/operator-expressions.md##object-operands") | [Operator Resolution](#operator-resolution) | [Arithmetic Operators](arithmetic-operators.md) | [Relational Operators](#relational-operators) |    
 [Like Operator](#like-operator) | [Concatenation Operator](#concatenation-operator) | [Logical Operators](#Logical-operators) | [Shift Operators](#shift-operators) |*^

## Operator Expressions

There are two kinds of operators. *Unary operators* take one operand and use prefix notation (for example, `-x`). *Binary operators* take two operands and use infix notation (for example, `x + y`). With the exception of the relational operators, which always result in `Boolean`, an operator defined for a particular type results in that type. The operands to an operator must always be classified as a value; the result of an operator expression is classified as a value.

```antlr
OperatorExpression : ArithmeticOperatorExpression
                   | RelationalOperatorExpression
                   | LikeOperatorExpression
                   | ConcatenationOperatorExpression
                   | ShortCircuitLogicalOperatorExpression
                   | LogicalOperatorExpression
                   | ShiftOperatorExpression
                   | AwaitOperatorExpression
                   ;
```

### Operator Precedence and Associativity

When an expression contains multiple binary operators, the *precedence* of the operators controls the order in which the individual binary operators are evaluated. For example, the expression `x + y * z` is evaluated as `x + (y * z)` because the `*` operator has higher precedence than the `+` operator. The following table lists the binary operators in descending order of precedence:


| __Category__     | __Operators__                                          | 
|------------------|--------------------------------------------------------|
| Primary          | All non-operator expressions                           |
| Await            | `Await`                                                |
| Exponentiation   | `^`                                                    |
| Unary negation   | `+`, `-`                                               |
| Multiplicative   | `*`, `/`                                               |
| Integer division | `\`                                                    |
| Modulus          | `Mod`                                                  |
| Additive         | `+`, `-`                                               |
| Concatenation    | `&`                                                    |
| Shift            | `<<`, `>>`                                             |
| Relational       | `=`, `<>`, `<`, `>`, `<=`, `>=`, `Like`, `Is`, `IsNot` |
| Logical NOT      | `Not`                                                  |
| Logical AND      | `And`, `AndAlso`                                       |
| Logical OR       | `Or`, `OrElse`                                         |
| Logical XOR      | `Xor`                                                  |

When an expression contains two operators with the same precedence, the *associativity* of the operators controls the order in which the operations are performed. All binary operators are left-associative, meaning that operations are performed from left to right. Precedence and associativity can be controlled using parenthetical expressions.

### Object Operands

In addition to the regular types supported by each operator, all operators support operands of type `Object`. Operators applied to `Object` operands are handled similarly to method calls made on `Object` values: a late-bound method call might be chosen, in which case the run-time type of the operands, rather than the compile-time type, determines the validity and type of the operation. If strict semantics are specified by the compilation environment or by `Option Strict`, any operators with operands of type `Object` cause a compile-time error, except for the `TypeOf...Is`, `Is` and `IsNot` operators.

When operator resolution determines that an operation should be performed late-bound, the outcome of the operation is the result of applying the operator to the operand types if the run-time types of the operands are types that are supported by the operator. The value `Nothing` is treated as the default value of the type of the other operand in a binary operator expression. In a unary operator expression, or if both operands are `Nothing` in a binary operator expression, the type of the operation is `Integer` or the only result type of the operator, if the operator does not result in `Integer`. The result of the operation is always then cast back to `Object`. If the operand types have no valid operator, a `System.InvalidCastException` exception is thrown. Conversions at run time are done without regard to whether they are implicit or explicit.

If the result of a numeric binary operation would produce an overflow exception (regardless of whether integer overflow checking is on or off), then the result type is promoted to the next wider numeric type, if possible. For example, consider the following code:

```vb
Module Test
    Sub Main()
        Dim o As Object = CObj(CByte(2)) * CObj(CByte(255))

        Console.WriteLine(o.GetType().ToString() & " = " & o)
    End Sub
End Module
```

It prints the following result:

```
System.Int16 = 512
```

If no wider numeric type is available to hold the number, a `System.OverflowException` exception is thrown.

### Operator Resolution

Given an operator type and a set of operands, operator resolution determines which operator to use for the operands. When resolving operators, user-defined operators will be considered first, using the following steps:

1. First, all of the candidate operators are collected. The candidate operators are all of the user-defined operators of the particular operator type in the source type and all of the user-defined operators of the particular type in the target type. If the source type and destination type are related, common operators are only considered once.

2. Then, overload resolution is applied to the operators and operands to select the most specific operator. In the case of binary operators, this may result in a late-bound call.

When collecting the candidate operators for a type `T?`, the operators of type `T` are used instead. Any of `T`'s user-defined operators that involve only non-nullable value types are also lifted. A lifted operator uses the nullable version of any value types, with the exception the return types of `IsTrue` and `IsFalse` (which must be `Boolean`). Lifted operators are evaluated by converting the operands to their non-nullable version, then evaluating the user-defined operator and then converting the result type to its nullable version. If ether operand is `Nothing`, the result of the expression is a value of `Nothing` typed as the nullable version of the result type. For example:

```vb
Structure T
    ...
End Structure

Structure S
    Public Shared Operator +(ByVal op1 As S, ByVal op2 As T) As T
        ...
    End Operator
End Structure

Module Test
    Sub Main()
        Dim x As S?
        Dim y, z As T?

        ' Valid, as S + T = T is lifted to S? + T? = T?
        z = x + y 
    End Sub
End Module
```

If the operator is a binary operator and one of the operands is reference type, the operator is also lifted, but any binding to the operator produces an error. For example:

```vb
Structure S1
    Public F1 As Integer

    Public Shared Operator +(left As S1, right As String) As S1
       ...
    End Operator
End Structure

Module Test
    Sub Main()
        Dim a? As S1
        Dim s As String
        
        ' Error: '+' is not defined for S1? and String
        a = a + s
    End Sub
End Module
```

__Note.__ This rule exists because there has been consideration whether we wish to add null-propagating reference types in a future version, in which case the behavior in the case of binary operators between the two types would change.

As with conversions, user-defined operators are always preferred over lifted operators.

When resolving overloaded operators, there may be differences between classes defined in Visual Basic and those defined in other languages:

* In other languages, `Not`, `And`, and `Or` may be overloaded both as logical operators and bitwise operators. Upon import from an external assembly, either form is accepted as a valid overload for these operators. However, for a type which defines both logical and bitwise operators, only the bitwise implementation will be considered.

* In other languages, `>>` and `<<` may be overloaded both as signed operators and unsigned operators. Upon import from an external assembly, either form is accepted as a valid overload. However, for a type which defines both signed and unsigned operators, only the signed implementation will be considered.

* If no user-defined operator is most specific to the operands, then intrinsic operators will be considered. If no intrinsic operator is defined for the operands and either operand has type Object then the operator will be resolved late-bound; otherwise,  a compile-time error results.

In prior versions of Visual Basic, if there was exactly one operand of type Object, and no applicable user-defined operators, and no applicable intrinsic operators, then it was an error. As of Visual Basic 11, it is now resolved late-bound. For example:

```vb
Module Module1
  Sub Main()
      Dim p As Object = Nothing
      Dim U As New Uri("http://www.microsoft.com")
      Dim j = U * p  ' is now resolved late-bound
   End Sub
End Module
```

A type `T` that has an intrinsic operator also defines that same operator for `T?`. The result of the operator on `T?` will be the same as for `T`, except that if either operand is `Nothing`, the result of the operator will be `Nothing` (i.e. the null value is propagated). For the purposes of resolving the type of an operation, the `?` is removed from any operands that have them, the type of the operation is determined, and a `?` is added to the type of the operation if any of the operands were nullable value types. For example:

```vb
Dim v1? As Integer = 10
Dim v2 As Long = 20

' Type of operation will be Long?
Console.WriteLine(v1 + v2)
```

Each operator lists the intrinsic types it is defined for and the type of the operation performed given the operand types. The result of type of a intrinsic operation follows these general rules:

* If all operands are of the same type, and the operator is defined for the type, then no conversion occurs and the operator for that type is used.

* Any operand whose type is not defined for the operator is converted using the following steps and the operator is resolved against the new types:

  * The operand is converted to the next widest type that is defined for both the operator and the operand and to which it is implicitly convertible.

  * If there is no such type, then the operand is converted to the next narrowest type that is defined for both the operator and the operand and to which it is implicitly convertible.

  * If there is no such type or the conversion cannot occur, a compile-time error occurs.

* Otherwise, the operands are converted to the wider of the operand types and the operator for that type is used. If the narrower operand type cannot be implicitly converted to the wider operator type, a compile-time error occurs.

Despite these general rules, however, there are a number of special cases called out in the operator results tables.

__Note.__ For formatting reasons, the operator type tables abbreviate the predefined names to their first two characters. So "By" is `Byte`, "UI" is `UInteger`, "St" is `String`, etc. "Err" means that there is no operation defined for the given operand types.



## Relational Operators

The *relational operators* compare values to one other. The comparison operators are `=`, `<>`, `<`, `>`, `<=`, and `>=`.

```antlr
RelationalOperatorExpression
    : Expression '=' LineTerminator? Expression
    | Expression '<' '>' LineTerminator? Expression
    | Expression '<' LineTerminator? Expression
    | Expression '>' LineTerminator? Expression
    | Expression '<' '=' LineTerminator? Expression
    | Expression '>' '=' LineTerminator? Expression
    ;
```

All of the relational operators result in a `Boolean` value.

The relational operators have the following general meaning:

* The `=` operator tests whether the two operands are equal.

* The `<>` operator tests whether the two operands are not equal.

* The `<` operator tests whether the first operand is less than the second operand.

* The `>` operator tests whether the first operand is greater than the second operand.

* The `<=` operator tests whether the first operand is less than or equal to the second operand.

* The `>=` operator tests whether the first operand is greater than or equal to the second operand.

The relational operators are defined for the following types:

* `Boolean`. The operators compare the truth values of the two operands. `True` is considered to be less than `False`, which matches with their numeric values.

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, and `Long`. The operators compare the numeric values of the two integral operands.

* `Single` and `Double`. The operators compare the operands according to the rules of the IEEE 754 standard.

* `Decimal`. The operators compare the numeric values of the two decimal operands.

* `Date`. The operators return the result of comparing the two date/time values.

* `Char`. The operators return the result of comparing the two Unicode values.

* `String`. The operators return the result of comparing the two values using either a binary comparison or a text comparison. The comparison used is determined by the compilation environment and the `Option Compare` statement. A binary comparison determines whether the numeric Unicode value of each character in each string is the same. A text comparison does a Unicode text comparison based on the current culture in use on the .NET Framework. When doing a string comparison, a null value is equivalent to the string literal `""`.

__Operation Type:__
        
|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| __Bo__ | Bo | SB | Sh | Sh | In | In | Lo | Lo | De | De | Si | Do | Err | Err | Bo | Ob | 
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
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Da  | Err | Da | Ob | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Ch  | St | Ob | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | St | Ob | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |    | Ob | 


## Like Operator

The `Like` operator determines whether a string matches a given pattern.

```antlr
LikeOperatorExpression
    : Expression 'Like' LineTerminator? Expression
    ;
```

The `Like` operator is defined for the `String` type. The first operand is the string being matched, and the second operand is the pattern to match against. The pattern is made up of Unicode characters. The following character sequences have special meanings:

* The character `?` matches any single character.

* The character `*` matches zero or more characters.

* The character `#` matches any single digit (0-9).

* A list of characters surrounded by brackets (`[ab...]`) matches any single character in the list.

* A list of characters surrounded by brackets and prefixed by an exclamation point (`[!ab...]`) matches any single character not in the character list.

* Two characters in a character list separated by a hyphen (`-`) specify a range of Unicode characters starting with the first character and ending with the second character. If the second character is not later in the sort order than the first character, a run-time exception occurs. A hyphen that appears at the beginning or end of a character list specifies itself.

To match the special characters left bracket (`[`), question mark (`?`), number sign (`#`), and asterisk (`*`), brackets must enclose them. The right bracket (`]`) cannot be used within a group to match itself, but it can be used outside a group as an individual character. The character sequence `[]` is considered to be the string literal `""`. 

Note that character comparisons and ordering for character lists are dependent on the type of comparisons being used. If binary comparisons are being used, character comparisons and ordering are based on the numeric Unicode values. If text comparisons are being used, character comparisons and ordering are based on the current locale being used on the .NET Framework.

In some languages, special characters in the alphabet represent two separate characters and vice versa. For example, several languages use the character `æ` to represent the characters `a` and `e` when they appear together, while the characters `^` and `O` can be used to represent the character `Ô`. When using text comparisons, the `Like` operator recognizes such cultural equivalences. In that case, an occurrence of the single special character in either pattern or string matches the equivalent two-character sequence in the other string. Similarly, a single special character in pattern enclosed in brackets (by itself, in a list, or in a range) matches the equivalent two-character sequence in the string and vice versa.

In a `Like` expression where both operands are `Nothing` or one operand has an intrinsic conversion to `String` and the other operand is `Nothing`, `Nothing` is treated as if it were the empty string literal `""`.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|--------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| __Bo__ | St | St | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __SB__ |    | St | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __By__ |    |    | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __Sh__ |    |    |    | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __US__ |    |    |    |    | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __In__ |    |    |    |    |    | St | St | St | St | St | St | St | St | St | St | Ob | 
| __UI__ |    |    |    |    |    |    | St | St | St | St | St | St | St | St | St | Ob | 
| __Lo__ |    |    |    |    |    |    |    | St | St | St | St | St | St | St | St | Ob | 
| __UL__ |    |    |    |    |    |    |    |    | St | St | St | St | St | St | St | Ob | 
| __De__ |    |    |    |    |    |    |    |    |    | St | St | St | St | St | St | Ob | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | St | St | St | St | St | Ob | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | St | St | St | St | Ob | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | St | St | St | Ob | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |    | St | St | Ob | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |    |    | St | Ob | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    | Ob | 


## Concatenation Operator

```antlr
ConcatenationOperatorExpression
    : Expression '&' LineTerminator? Expression
    ;
```

The *concatenation operator* is defined for all of the intrinsic types, including the nullable versions of the intrinsic value types. It is also defined for concatenation between the types mentioned above and `System.DBNull`, which is treated as a `Nothing` string. The concatenation operator converts all of its operands to `String`; in the expression, all conversions to `String` are considered to be widening, regardless of whether strict semantics are used. A `System.DBNull` value is converted to the literal `Nothing` typed as `String`. A nullable value type whose value is `Nothing` is also converted to the literal `Nothing` typed as `String`, rather than throwing a run-time error.

A concatenation operation results in a string that is the concatenation of the two operands in order from left to right. The value `Nothing` is treated as if it were the empty string literal `""`.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|--------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| __Bo__ | St | St | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __SB__ |    | St | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __By__ |    |    | St | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __Sh__ |    |    |    | St | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __US__ |    |    |    |    | St | St | St | St | St | St | St | St | St | St | St | Ob | 
| __In__ |    |    |    |    |    | St | St | St | St | St | St | St | St | St | St | Ob | 
| __UI__ |    |    |    |    |    |    | St | St | St | St | St | St | St | St | St | Ob | 
| __Lo__ |    |    |    |    |    |    |    | St | St | St | St | St | St | St | St | Ob | 
| __UL__ |    |    |    |    |    |    |    |    | St | St | St | St | St | St | St | Ob | 
| __De__ |    |    |    |    |    |    |    |    |    | St | St | St | St | St | St | Ob | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | St | St | St | St | St | Ob | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | St | St | St | St | Ob | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | St | St | St | Ob | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |    | St | St | Ob | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |    |    | St | Ob | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    | Ob | 


## Logical Operators

The `And`, `Not`, `Or`, and `Xor` operators are called the logical operators.

```antlr
LogicalOperatorExpression
    : 'Not' Expression
    | Expression 'And' LineTerminator? Expression
    | Expression 'Or' LineTerminator? Expression
    | Expression 'Xor' LineTerminator? Expression
    ;
```

The logical operators are evaluated as follows:

* For the `Boolean` type:

  * A logical `And` operation is performed on its two operands.

  * A logical `Not` operation is performed on its operand.

  * A logical `Or` operation is performed on its two operands.

  * A logical exclusive-`Or` operation is performed on its two operands.

* For `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, and all enumerated types, the specified operation is performed on each bit of the binary representation of the two operand(s):

  * `And`: The result bit is 1 if both bits are 1; otherwise the result bit is 0.

  * `Not`: The result bit is 1 if the bit is 0; otherwise the result bit is 1.

  * `Or`: The result bit is 1 if either bit is 1; otherwise the result bit is 0.

  * `Xor`: The result bit is 1 if either bit is 1 but not both bits; otherwise the result bit is 0 (that is, 1 `Xor` 0 = 1, 1 `Xor` 1 = 0).

* When the logical operators `And` and `Or` are lifted for the type `Boolean?`, they are extended to encompass three-valued Boolean logic as such:

  * `And` evaluates to true if both operands are true; false if one of the operands is false; `Nothing` otherwise.

  * `Or` evaluates to true if either operand is true; false is both operands are false; `Nothing` otherwise.

For example:

```vb
Module Test
    Sub Main()
        Dim x?, y? As Boolean

        x = Nothing
        y = True 

        If x Or y Then
            ' Will execute
        End If
    End Sub
End Module
```

__Note.__ Ideally, the logical operators `And` and `Or` would be lifted using three-valued logic for any type that can be used in a Boolean expression (i.e. a type that implements `IsTrue` and `IsFalse`), in the same way that `AndAlso` and `OrElse` short circuit across any type that can be used in a Boolean expression. Unfortunately, three-valued lifting is only applied to `Boolean?`, so user-defined types that desire three-valued logic must do so manually by defining `And` and `Or` operators for their nullable version.

No overflows are possible from these operations. The enumerated type operators do the bitwise operation on the underlying type of the enumerated type, but the return value is the enumerated type.

__Not Operation Type:__

| __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| Bo | SB | By | Sh | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo | Ob | 

__And, Or, Xor Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Bo | SB | Sh | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Bo  | Ob  | 
| __SB__ |    | SB | Sh | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __By__ |    |    | By | Sh | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Sh__ |    |    |    | Sh | In | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __US__ |    |    |    |    | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __In__ |    |    |    |    |    | In | Lo | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __UI__ |    |    |    |    |    |    | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Lo | Lo | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | UL | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | Lo | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Lo | Lo | Err | Err | Lo  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Lo | Err | Err | Lo  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Lo  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


### Short-circuiting Logical Operators

The `AndAlso` and `OrElse` operators are the short-circuiting versions of the `And` and `Or` logical operators.

```antlr
ShortCircuitLogicalOperatorExpression
    : Expression 'AndAlso' LineTerminator? Expression
    | Expression 'OrElse' LineTerminator? Expression
    ;
```

Because of their short circuiting behavior, the second operand is not evaluated at run time if the operator result is known after evaluating the first operand.

The short-circuiting logical operators are evaluated as follows:

* If the first operand in an `AndAlso` operation evaluates to `False` or returns True from its `IsFalse` operator, the expression returns its first operand. Otherwise, the second operand is evaluated and a logical `And` operation is performed on the two results.

* If the first operand in an `OrElse` operation evaluates to `True` or returns True from its `IsTrue` operator, the expression returns its first operand. Otherwise, the second operand is evaluated and a logical `Or` operation is performed on its two results.

The `AndAlso` and `OrElse` operators are defined for the type `Boolean`, or for any type `T` that overloads the following operators:

```vb
Public Shared Operator IsTrue(op As T) As Boolean
Public Shared Operator IsFalse(op As T) As Boolean
```

as well as overloading the corresponding `And` or `Or` operator:

```vb
Public Shared Operator And(op1 As T, op2 As T) As T
Public Shared Operator Or(op1 As T, op2 As T) As T
```

When evaluating the `AndAlso` or `OrElse` operators, the first operand is evaluated only once, and the second operand is either not evaluated or evaluated exactly once. For example, consider the following code:

```vb
Module Test
    Function TrueValue() As Boolean
        Console.Write(" True")
        Return True
    End Function

    Function FalseValue() As Boolean
        Console.Write(" False")
        Return False
    End Function

    Sub Main()
        Console.Write("And:")
        If FalseValue() And TrueValue() Then
        End If
        Console.WriteLine()

        Console.Write("Or:")
        If TrueValue() Or FalseValue() Then
        End If
        Console.WriteLine()

        Console.Write("AndAlso:")
        If FalseValue() AndAlso TrueValue() Then
        End If
        Console.WriteLine()

        Console.Write("OrElse:")
        If TrueValue() OrElse FalseValue() Then
        End If
        Console.WriteLine()
    End Sub
End Module
```

It prints the following result:

```
And: False True
Or: True False
AndAlso: False
OrElse: True
```

In the lifted form of the `AndAlso` and `OrElse` operators, if the first operand was a null `Boolean?`, then the second operand is evaluated but the result is always a null `Boolean?`.

__Operation Type:__

|        | __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ |
|--------|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| __Bo__ | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __SB__ |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __By__ |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __Sh__ |    |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __US__ |    |    |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __In__ |    |    |    |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __UI__ |    |    |    |    |    |    | Bo | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __Lo__ |    |    |    |    |    |    |    | Bo | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __UL__ |    |    |    |    |    |    |    |    | Bo | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __De__ |    |    |    |    |    |    |    |    |    | Bo | Bo | Bo | Err | Err | Bo  | Ob  | 
| __Si__ |    |    |    |    |    |    |    |    |    |    | Bo | Bo | Err | Err | Bo  | Ob  | 
| __Do__ |    |    |    |    |    |    |    |    |    |    |    | Bo | Err | Err | Bo  | Ob  | 
| __Da__ |    |    |    |    |    |    |    |    |    |    |    |    | Err | Err | Err | Err | 
| __Ch__ |    |    |    |    |    |    |    |    |    |    |    |    |     | Err | Err | Err | 
| __St__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     | Bo  | Ob  | 
| __Ob__ |    |    |    |    |    |    |    |    |    |    |    |    |     |     |     | Ob  | 


## Shift Operators

The binary operators `<<` and `>>` perform bit shifting operations.

```antlr
ShiftOperatorExpression
    : Expression '<' '<' LineTerminator? Expression
    | Expression '>' '>' LineTerminator? Expression
    ;
```

The operators are defined for the `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong` and `Long` types. Unlike the other binary operators, the result type of a shift operation is determined as if the operator was a unary operator with just the left operand. The type of the right operand must be implicitly convertible to `Integer` and is not used in determining the result type of the operation.

The `<<` operator causes the bits in the first operand to be shifted left the number of places specified by the shift amount. The high-order bits outside the range of the result type are discarded and the low-order vacated bit positions are zero-filled.

The `>>` operator causes the bits in the first operand to be shifted right the number of places specified by the shift amount. The low-order bits are discarded and the high-order vacated bit positions are set to zero if the left operand is positive or to one if negative. If the left operand is of type `Byte`, `UShort`, `UInteger`, or `ULong` the vacant high-order bits are zero-filled.

The shift operators shift the bits of the underlying representation of the first operand by the amount of the second operand. If the value of the second operand is greater than the number of bits in the first operand, or is negative, then the shift amount is computed as `RightOperand And SizeMask` where `SizeMask` is:

| __LeftOperand Type__  | __SizeMask__ | 
|-----------------------|--------------|
| `Byte`, `SByte`       | 7 (`&H7`)    | 
| `UShort`, `Short`     | 15 (`&HF`)   | 
| `UInteger`, `Integer` | 31 (`&H1F`)  | 
| `ULong`, `Long`       | 63 (`&H3F`)  | 

If the shift amount is zero, the result of the operation is identical to the value of the first operand. No overflows are possible from these operations.

__Operation Type:__


| __Bo__ | __SB__ | __By__ | __Sh__ | __US__ | __In__ | __UI__ | __Lo__ | __UL__ | __De__ | __Si__ | __Do__ | __Da__  | __Ch__  | __St__ | __Ob__ | 
|----|----|----|----|----|----|----|----|----|----|----|----|-----|-----|----|----|
| Sh | SB | By | Sh | US | In | UI | Lo | UL | Lo | Lo | Lo | Err | Err | Lo | Ob | 