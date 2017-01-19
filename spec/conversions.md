# Conversions

Conversion is the process of changing a value from one type to another. For example, a value of type `Integer` can be converted to a value of type `Double`, or a value of type `Derived` can be converted to a value of type `Base`, assuming that `Base` and `Derived` are both classes and `Derived` inherits from `Base`. Conversions may not require the value itself to change (as in the latter example), or they may require significant changes in the value representation (as in the former example).

Conversions may either be widening or narrowing. A *widening conversion* is a conversion from a type to another type whose value domain is at least as big, if not bigger, than the original type's value domain. Widening conversions should never fail. A *narrowing conversion* is a conversion from a type to another type whose value domain is either smaller than the original type's value domain or sufficiently unrelated that extra care must be taken when doing the conversion (for example, when converting from `Integer` to `String`). Narrowing conversions, which may entail loss of information, can fail.

The identity conversion (i.e. a conversion from a type to itself) and default value conversion (i.e. a conversion from `Nothing`) are defined for all types.

## Implicit and Explicit Conversions

Conversions can be either *implicit* or *explicit*. Implicit conversions occur without any special syntax. The following is an example of implicit conversion of an `Integer` value to a `Long` value:

```vb
Module Test
    Sub Main()
        Dim intValue As Integer = 123
        Dim longValue As Long = intValue

        Console.WriteLine(intValue & " = " & longValue)
    End Sub
End Module
```

Explicit conversions, on the other hand, require cast operators. Attempting to do an explicit conversion on a value without a cast operator causes a compile-time error. The following example uses an explicit conversion to convert a `Long` value to an `Integer` value.

```vb
Module Test
    Sub Main()
        Dim longValue As Long = 134
        Dim intValue As Integer = CInt(longValue)

        Console.WriteLine(longValue & " = " & intValue)
    End Sub
End Module
```

The set of implicit conversions depends on the compilation environment and the `Option Strict` statement. If strict semantics are being used, only widening conversions may occur implicitly. If permissive semantics are being used, all widening and narrowing conversions (in other words, all conversions) may occur implicitly.

## Boolean Conversions

Although `Boolean` is not a numeric type, it does have narrowing conversions to and from the numeric types as if it were an enumerated type. The literal `True` converts to the literal `255` for `Byte`, `65535` for `UShort`, `4294967295` for `UInteger`, `18446744073709551615` for `ULong`, and to the expression `-1` for `SByte`, `Short`, `Integer`, `Long`, `Decimal`, `Single`, and `Double`. The literal `False` converts to the literal `0`. A zero numeric value converts to the literal `False`. All other numeric values convert to the literal `True`.

There is a narrowing conversion from Boolean to String, converting to either `System.Boolean.TrueString` or `System.Boolean.FalseString`. There is also a narrowing conversion from `String` to `Boolean`: if the string was equal to `TrueString` or `FalseString` (in the current culture, case-insensitively) then it uses the appropriate value; otherwise it attempts to parse the string as a numeric type (in hex or octal if possible, otherwise as a float) and uses the above rules; otherwise it throws `System.InvalidCastException`.

## Numeric Conversions

Numeric conversions exist between the types `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single` and `Double`, and all enumerated types. When being converted, enumerated types are treated as if they were their underlying types. When converting to an enumerated type, the source value is not required to conform to the set of values defined in the enumerated type. For example:

```vb
Enum Values
    One
    Two
    Three
End Enum

Module Test
    Sub Main()
        Dim x As Integer = 5

        ' OK, even though there is no enumerated value for 5.
        Dim y As Values = CType(x, Values)
    End Sub
End Module
```

Numeric conversions are processed at run-time as follows:

* For a conversion from a numeric type to a wider numeric type, the value is simply converted to the wider type. Conversions from `UInteger`, `Integer`, `ULong`, `Long`, or `Decimal` to `Single` or `Double` are rounded to the nearest `Single` or `Double` value. While this conversion may cause a loss of precision, it will never cause a loss of magnitude.

* For a conversion from an integral type to another integral type, or from `Single`, `Double`, or `Decimal` to an integral type, the result depends on whether integer overflow checking is on:

  *If integer overflow is being checked:*

  * If the source is an integral type, the conversion succeeds if the source argument is within the range of the destination type. The conversion throws a `System.OverflowException` exception if the source argument is outside the range of the destination type.

  * If the source is `Single`, `Double`, or `Decimal`, the source value is rounded up or down to the nearest integral value, and this integral value becomes the result of the conversion. If the source value is equally close to two integral values, the value is rounded to the value that has an even number in the least significant digit position. If the resulting integral value is outside the range of the destination type, a `System.OverflowException` exception is thrown.

  *If integer overflow is not being checked:*

  * If the source is an integral type, the conversion always succeeds and simply consists of discarding the most significant bits of the source value.

  * If the source is `Single`, `Double`, or `Decimal`, the conversion always succeeds and simply consists of rounding the source value towards the nearest integral value. If the source value is equally close to two integral values, the value is always rounded to the value that has an even number in the least significant digit position.

* For a conversion from `Double` to `Single`, the `Double` value is rounded to the nearest `Single` value. If the `Double` value is too small to represent as a `Single`, the result becomes positive zero or negative zero. If the `Double` value is too large to represent as a `Single`, the result becomes positive infinity or negative infinity. If the `Double` value is `NaN`, the result is also `NaN`.

* For a conversion from `Single` or `Double` to `Decimal`, the source value is converted to `Decimal` representation and rounded to the nearest number after the 28th decimal place if required. If the source value is too small to represent as a `Decimal`, the result becomes zero. If the source value is `NaN`, infinity, or too large to represent as a `Decimal`, a `System.OverflowException` exception is thrown.

* For a conversion from `Double` to `Single`, the `Double` value is rounded to the nearest `Single` value. If the `Double` value is too small to represent as a `Single`, the result becomes positive zero or negative zero. If the `Double` value is too large to represent as a `Single`, the result becomes positive infinity or negative infinity. If the `Double` value is `NaN`, the result is also `NaN`.

## Reference Conversions

Reference types may be converted to a base type, and vice versa. Conversions from a base type to a more derived type only succeed at run time if the value being converted is a null value, the derived type itself, or a more derived type.

Class and interface types can be cast to and from any interface type. Conversions between a type and an interface type only succeed at run time if the actual types involved have an inheritance or implementation relationship. Because an interface type will always contain an instance of a type that derives from `Object`, an interface type can also always be cast to and from `Object`.

__Note.__ It is not an error to convert a `NotInheritable` classes to and from interfaces that it does not implement because classes that represent COM classes may have interface implementations that are not known until run time. 

If a reference conversion fails at run time, a `System.InvalidCastException` exception is thrown.

### Reference Variance Conversions

Generic interfaces or delegates may have variant type parameters that allow conversions between compatible variants of the type. Therefore, at runtime a conversion from a class type or an interface type to an interface type that is variant compatible with an interface type it inherits from or implements will succeed. Similarly, delegate types can be cast to and from variant compatible delegate types. For example, the delegate type

```vb
Delegate Function F(Of In A, Out R)(a As A) As R
```

would allow a conversion from `F(Of Object, Integer)` to `F(Of String, Integer)`. That is, a delegate `F` which takes `Object` may be safely used as a delegate `F` which takes `String`. When the delegate is invoked, the target method will be expecting an object, and a string is an object.

A generic delegate or interface type `S(Of S1,...,Sn)` is said to be *variant compatible* with a generic interface or delegate type `T(Of T1,...,Tn)` if:

* `S` and `T` are both constructed from the same generic type `U(Of U1,...,Un)`.

* For each type parameter `Ux`:

  * If the type parameter was declared without variance then `Sx` and `Tx` must be the same type.

  * If the type parameter was declared `In` then there must be a widening identity, default, reference, array, or type parameter conversion from `Sx` to `Tx`.

  * If the type parameter was declared `Out` then there must be a widening identity, default, reference, array, or type parameter conversion from `Tx` to `Sx`.

When converting from a class to a generic interface with variant type parameters, if the class implements more than one variant compatible interface the conversion is ambiguous if there is not a non-variant conversion. For example:

```vb
Class Base
End Class

Class Derived1
    Inherits Base
End Class

Class Derived2
    Inherits Base
End Class

Class OneAndTwo
    Implements IEnumerable(Of Derived1)
    Implements IEnumerable(Of Derived2)
End Class

Class BaseAndOneAndTwo
    Implements IEnumerable(Of Base)
    Implements IEnumerable(Of Derived1)
    Implements IEnumerable(Of Derived2)
End Class

Module Test
    Sub Main()
        ' Error: conversion is ambiguous
        Dim x As IEnumerable(Of Base) = New OneAndTwo()

        ' OK, will pick up the direct implementation of IEnumerable(Of Base)
        Dim y as IEnumerable(Of Base) = New BaseAndOneAndTwo()
    End Sub
End Module
```

### Anonymous Delegate Conversions

When an expression classified as a lambda method is reclassified as a value in a context where there is no target type (for example, `Dim x = Function(a As Integer, b As Integer) a + b`), or where the target type is not a delegate type, the type of the resulting expression is an anonymous delegate type equivalent to the signature of the lambda method. This anonymous delegate type has a conversion to any compatible delegate type: a compatible delegate type is any delegate type that can be created using a delegate creation expression with the anonymous delegate type's `Invoke` method as a parameter. For example:

```vb
' Anonymous delegate type similar to Func(Of Object, Object, Object)
Dim x = Function(x, y) x + y

' OK because delegate type is compatible
Dim y As Func(Of Integer, Integer, Integer) = x
```

Note that the types `System.Delegate` and `System.MulticastDelegate` are not themselves considered delegate types (even though all delegate types inherit from them). Also, note that the conversion from anonymous delegate type to a compatible delegate type is not a reference conversion.

## Array Conversions

Besides the conversions that are defined on arrays by virtue of the fact that they are reference types, several special conversions exist for arrays.

For any two types `A` and `B`, if they are both reference types or type parameters not known to be value types, and if `A` has a reference, array, or type parameter conversion to `B`, a conversion exists from an array of type `A` to an array of type `B` with the same rank. This relationship is known as *array covariance*. Array covariance in particular means that an element of an array whose element type is `B` may actually be an element of an array whose element type is `A`, provided that both `A` and `B` are reference types and that `B` has a reference conversion or array conversion to `A`. In the following example, the second invocation of `F` causes a `System.ArrayTypeMismatchException` exception to be thrown because the actual element type of `b` is `String`, not `Object`:

```vb
Module Test
    Sub F(ByRef x As Object)
    End Sub

    Sub Main()
        Dim a(10) As Object
        Dim b() As Object = New String(10) {}
        F(a(0)) ' OK.
        F(b(1)) ' Not allowed: System.ArrayTypeMismatchException.
   End Sub
End Module
```

Because of array covariance, assignments to elements of reference type arrays include a run-time check that ensures that the value being assigned to the array element is actually of a permitted type.

```vb
Module Test
    Sub Fill(array() As Object, index As Integer, count As Integer, _
            value As Object)
        Dim i As Integer

        For i = index To (index + count) - 1
            array(i) = value
        Next i
    End Sub

    Sub Main()
        Dim strings(100) As String

        Fill(strings, 0, 101, "Undefined")
        Fill(strings, 0, 10, Nothing)
        Fill(strings, 91, 10, 0)
    End Sub
End Module
```

In this example, the assignment to `array(i)` in method `Fill` implicitly includes a run-time check that ensures that the object referenced by the variable `value` is either `Nothing` or an instance of a type that is compatible with the actual element type of array `array`. In method `Main`, the first two invocations of method `Fill` succeed, but the third invocation causes a `System.ArrayTypeMismatchException` exception to be thrown upon executing the first assignment to `array(i)`. The exception occurs because an `Integer` cannot be stored in a `String` array.

If one of the array element types is a type parameter whose type turns out to be a value type at runtime, a `System.InvalidCastException` exception will be thrown. For example:

```vb
Module Test
    Sub F(Of T As U, U)(x() As T)
        Dim y() As U = x
    End Sub

    Sub Main()
        ' F will throw an exception because Integer() cannot be
        ' converted to Object()
        F(New Integer() { 1, 2, 3 })
    End Sub
End Module
```

Conversions also exist between an array of an enumerated type and an array of the enumerated type's underlying type or an array of another enumerated type with the same underlying type, provided the arrays have the same rank.

```vb
Enum Color As Byte
    Red
    Green
    Blue
End Enum

Module Test
    Sub Main()
        Dim a(10) As Color
        Dim b() As Integer
        Dim c() As Byte

        b = a    ' Error: Integer is not the underlying type of Color
        c = a    ' OK
        a = c    ' OK
    End Sub
End Module
```

In this example, an array of `Color` is converted to and from an array of `Byte`, `Color`'s underlying type. The conversion to an array of `Integer`, however, will be an error because `Integer` is not the underlying type of `Color`.

A rank-1 array of type `A()` also has an array conversion to the collection interface types `IList(Of B)`, `IReadOnlyList(Of B)`, `ICollection(Of B)`, `IReadOnlyCollection(Of B)` and `IEnumerable(Of B)` found in `System.Collections.Generic`, so long as one of the following is true:

- `A` and `B` are both reference types or type parameters not known to be value types; and `A` has a widening reference, array or type parameter conversion to `B`; or
- `A` and `B` are both enumerated types of the same underlying type; or
- one of `A` and `B` is an enumerated type, and the other is its underlying type.

Any array of type A with any rank also has an array conversion to the non-generic collection interface types `IList`, `ICollection` and `IEnumerable` found in `System.Collections`.

It is possible to iterate over the resulting interfaces using `For Each`, or through invoking the `GetEnumerator` methods directly. In the case of rank-1 arrays converted generic or non-generic forms of `IList` or `ICollection`, it is also possible to get elements by index. In the case of rank-1 arrays converted to generic or non-generic forms of `IList`, it is also possible to set elements by index, subject to the same runtime array covariance checks as described above. The behavior of all other interface methods is undefined by the VB language specification; it is up to the underlying runtime.

## Value Type Conversions

A value type value can be converted to one of its base reference types or an interface type that it implements through a process called *boxing*. When a value type value is boxed, the value is copied from the location where it lives onto the .NET Framework heap. A reference to this location on the heap is then returned and can be stored in a reference type variable. This reference is also referred to as a *boxed* instance of the value type. The boxed instance has the same semantics as a reference type instead of a value type.

Boxed value types can be converted back to their original value type through a process called *unboxing*. When a boxed value type is unboxed, the value is copied from the heap into a variable location. From that point on, it behaves as if it was a value type. When unboxing a value type, the value must be a null value or an instance of the value type. Otherwise a `System.InvalidCastException` exception is thrown. If the value is an instance of an enumerated type, that value can also be unboxed to the enumerated type's underlying type or another enumerated type that has the same underlying type. A null value is treated as if it were the literal `Nothing`.

To support nullable value types well, the value type `System.Nullable(Of T)` is treated specially when doing boxing and unboxing. Boxing a value of type `Nullable(Of T)` results in a boxed value of type `T` if the value's `HasValue` property is `True` or a value of `Nothing` if the value's `HasValue` property is `False`. Unboxing a value of type `T` to `Nullable(Of T)` results in an instance of `Nullable(Of T)` whose `Value` property is the boxed value and whose `HasValue` property is `True`. The value `Nothing` can be unboxed to `Nullable(Of T)` for any `T` and results in a value whose `HasValue` property is `False`. Because boxed value types behave like reference types, it is possible to create multiple references to the same value. For the primitive types and enumerated types, this is irrelevant because instances of those types are *immutable*. That is, it is not possible to modify a boxed instance of those types, so it is not possible to observe the fact that there are multiple references to the same value.

Structures, on the other hand, may be mutable if its instance variables are accessible or if its methods or properties modify its instance variables. If one reference to a boxed structure is used to modify the structure, then all references to the boxed structure will see the change. Because this result may be unexpected, when a value typed as `Object` is copied from one location to another boxed value types will automatically be cloned on the heap instead of merely having their references copied. For example:

```vb
Class Class1
    Public Value As Integer = 0
End Class

Structure Struct1
    Public Value As Integer
End Structure

Module Test
    Sub Main()
        Dim val1 As Object = New Struct1()
        Dim val2 As Object = val1

        val2.Value = 123

        Dim ref1 As Object = New Class1()
        Dim ref2 As Object = ref1

        ref2.Value = 123

        Console.WriteLine("Values: " & val1.Value & ", " & val2.Value)
        Console.WriteLine("Refs: " & ref1.Value & ", " & ref2.Value)
    End Sub
End Module
```

The output of the program is:

```
Values: 0, 123
Refs: 123, 123
```

The assignment to the field of the local variable `val2` does not impact the field of the local variable `val1` because when the boxed `Struct1` was assigned to `val2`, a copy of the value was made. In contrast, the assignment `ref2.Value = 123` affects the object that both `ref1` and `ref2` references.

__Note.__ Structure copying is not done for boxed structures typed as `System.ValueType` because it is not possible to late bind off of `System.ValueType`.

There is one exception to the rule that boxed value types will be copied on assignment. If a boxed value type reference is stored within another type, the inner reference will not be copied. For example:

```vb
Structure Struct1
    Public Value As Object
End Structure

Module Test
    Sub Main()
        Dim val1 As Struct1
        Dim val2 As Struct1

        val1.Value = New Struct1()
        val1.Value.Value = 10

        val2 = val1
        val2.Value.Value = 123
        Console.WriteLine("Values: " & val1.Value.Value & ", " & _
            val2.Value.Value)
    End Sub
End Module
```

The output of the program is:

```
Values: 123, 123
```

This is because the inner boxed value is not copied when the value is copied. Thus, both `val1.Value` and `val2.Value` have a reference to the same boxed value type.

__Note.__ The fact that inner boxed value types are not copied is a limitation of the .NET type system -- to ensure that all inner boxed value types were copied whenever a value of type `Object` was copied would be prohibitively expensive.

As previously described, boxed value types can only be unboxed to their original type. Boxed primitive types, however, are treated specially when typed as `Object`. They can be converted to any other primitive type that they have a conversion to. For example:

```vb
Module Test
    Sub Main()
        Dim o As Object = 5
        Dim b As Byte = CByte(o)  ' Legal
        Console.WriteLine(b) ' Prints 5
    End Sub
End Module
```

Normally, the boxed `Integer` value `5` could not be unboxed into a `Byte` variable. However, because `Integer` and `Byte` are primitive types and have a conversion, the conversion is allowed.

It is important to note that converting a value type to an interface is different than a generic argument constrained to an interface. When accessing interface members on a constrained type parameter (or calling methods on `Object`), boxing does not occur as it does when a value type is converted to an interface and an interface member is accessed. For example, suppose an interface `ICounter` contains a method `Increment` which can be used to modify a value. If `ICounter` is used as a constraint, the implementation of the `Increment` method is called with a reference to the variable that `Increment` was called on, not a boxed copy:

```vb
Interface ICounter
    Sub Increment()
    ReadOnly Property Value() As Integer
End Interface

Structure Counter
    Implements ICounter

    Dim _value As Integer

    Property Value() As Integer Implements ICounter.Value
        Get
            Return _value
        End Get
    End Property

    Sub Increment() Implements ICounter.Increment
       value += 1
    End Sub
End Structure

Module Test
      Sub Test(Of T As ICounter)(x As T)
         Console.WriteLine(x.value)
         x.Increment()                     ' Modify x
         Console.WriteLine(x.value)
         CType(x, ICounter).Increment()    ' Modify boxed copy of x
         Console.WriteLine(x.value)
      End Sub

      Sub Main()
         Dim x As Counter
         Test(x)
      End Sub
End Module
```

The first call to `Increment` modifies the value in the variable `x`. This is not equivalent to the second call to `Increment`, which modifies the value in a boxed copy of `x`. Thus, the output of the program is:

```
0
1
1
```

### Nullable Value Type Conversions

A value type `T` can convert to and from the nullable version of the type, `T?`. The conversion from `T?` to `T` throws a `System.InvalidOperationException` exception if the value being converted is `Nothing`. Also, `T?` has a conversion to a type `S` if `T` has an intrinsic conversion to `S`. And if `S` is a value type, then the following intrinsic conversions exist between `T?` and `S?`:

* A conversion of the same classification (narrowing or widening) from `T?` to `S?`.

* A conversion of the same classification (narrowing or widening) from `T` to `S?`.

* A narrowing conversion from `S?` to `T`.

For example, an intrinsic widening conversion exists from `Integer?` to `Long?` because an intrinsic widening conversion exists from `Integer` to `Long`:

```vb
Dim i As Integer? = 10
Dim l As Long? = i
```

When converting from `T?` to `S?`, if the value of `T?` is `Nothing`, then the value of `S?` will be `Nothing`. When converting from `S?` to `T` or `T?` to `S`, if the value of `T?` or `S?` is `Nothing`, a `System.InvalidCastException` exception will be thrown.

Because of the behavior of the underlying type `System.Nullable(Of T)`, when a nullable value type `T?` is boxed, the result is a boxed value of type `T`, not a boxed value of type `T?`. And, conversely, when unboxing to a nullable value type `T?`, the value will be wrapped by `System.Nullable(Of T)`, and `Nothing` will be unboxed to a null value of type `T?`. For example:

```vb
Dim i1? As Integer = Nothing
Dim o1 As Object = i1

Console.WriteLine(o1 Is Nothing)                    ' Will print True
o1 = 10
i1 = CType(o1, Integer?)
Console.WriteLine(i1)                               ' Will print 10
```

A side effect of this behavior is that a nullable value type `T?` appears to implement all of the interfaces of `T`, because converting a value type to an interface requires the type to be boxed. As a result, `T?` is convertible to all the interfaces that `T` is convertible to. It is important to note, however, that a nullable value type `T?` does not actually implement the interfaces of `T` for the purposes of generic constraint checking or reflection. For example:

```vb
Interface I1
End Interface

Structure T1
    Implements I1
    ...
End Structure

Module Test
    Sub M1(Of T As I1)(ByVal x As T)
    End Sub

    Sub Main()
        Dim x? As T1 = Nothing
        Dim y As I1 = x                ' Valid
        M1(x)                          ' Error: x? does not satisfy I1 constraint
    End Sub
End Module
```

## String Conversions

Converting `Char` into `String` results in a string whose first character is the character value. Converting `String` into `Char` results in a character whose value is the first character of the string. Converting an array of `Char` into `String` results in a string whose characters are the elements of the array. Converting `String` into an array of `Char` results in an array of characters whose elements are the characters of the string.

The exact conversions between `String` and `Boolean`, `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single`, `Double`, `Date`, and vice versa, are beyond the scope of this specification and are implementation dependent with the exception of one detail. String conversions always consider the current culture of the run-time environment. As such, they must be performed at run time.

## Widening Conversions

Widening conversions never overflow but may entail a loss of precision. The following conversions are widening conversions:

__Identity/Default conversions__

* From a type to itself.

* From an anonymous delegate type generated for a lambda method reclassification to any delegate type with an identical signature.

* From the literal `Nothing` to a type.

__Numeric conversions__

* From `Byte` to `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single`, or `Double`.

* From `SByte` to `Short`, `Integer`, `Long`, `Decimal`, `Single`, or `Double`.

* From `UShort` to `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single`, or `Double`.

* From `Short` to `Integer`, `Long`, `Decimal`, `Single` or `Double`.

* From `UInteger` to `ULong`, `Long`, `Decimal`, `Single`, or `Double`.

* From `Integer` to `Long`, `Decimal`, `Single` or `Double`.

* From `ULong` to `Decimal`, `Single`, or `Double`.

* From `Long` to `Decimal`, `Single` or `Double`.

* From `Decimal` to `Single` or `Double`.

* From `Single` to `Double`.

* From the literal `0` to an enumerated type. (__Note.__ The conversion from `0` to any enumerated type is widening to simplify testing flags. For example, if `Values` is an enumerated type with a value `One`, you could test a variable `v` of type `Values` by saying `(v And Values.One) = 0`.)

* From an enumerated type to its underlying numeric type, or to a numeric type that its underlying numeric type has a widening conversion to.

* From a constant expression of type `ULong`, `Long`, `UInteger`, `Integer`, `UShort`, `Short`, `Byte`, or `SByte` to a narrower type, provided the value of the constant expression is within the range of the destination type. (__Note.__ Conversions from `UInteger` or `Integer` to `Single`, `ULong` or `Long` to `Single` or `Double`, or `Decimal` to `Single` or `Double` may cause a loss of precision, but will never cause a loss of magnitude. The other widening numeric conversions never lose any information.)

__Reference conversions__

* From a reference type to a base type.

* From a reference type to an interface type, provided that the type implements the interface or a variant compatible interface.

* From an interface type to `Object`.

* From an interface type to a variant compatible interface type.

* From a delegate type to a variant compatible delegate type. (__Note.__ Many other reference conversions are implied by these rules. For example, anonymous delegates are reference types that inherit from `System.MulticastDelegate`; array types are reference types that inherit from `System.Array`; anonymous types are reference types that inherit from `System.Object`.)

__Anonymous Delegate conversions__

* From an anonymous delegate type generated for a lambda method reclassification to any wider delegate type.

__Array conversions__

* From an array type `S` with an element type `Se` to an array type `T` with an element type `Te`, provided all of the following are true:

  * `S` and `T` differ only in element type.

  * Both `Se` and `Te` are reference types or are type parameters known to be a reference type.

  * A widening reference, array, or type parameter conversion exists from `Se` to `Te`.

* From an array type `S` with an enumerated element type `Se` to an array type `T` with an element type `Te`, provided all of the following are true:

  * `S` and `T` differ only in element type.

  * `Te` is the underlying type of `Se`.

* From an array type `S` of rank 1 with an enumerated element type `Se`, to `System.Collections.Generic.IList(Of Te)`, `IReadOnlyList(Of Te)`, `ICollection(Of Te)`, `IReadOnlyCollection(Of Te)`, and `IEnumerable(Of Te)`, provided one of the following is true:

  * Both `Se` and `Te` are reference types or are type parameters known to be a reference type, and a widening reference, array, or type parameter conversion exists from `Se` to `Te`; or

  * `Te` is the underlying type of `Se`; or

  * `Te` is identical to `Se`

__Value Type conversions__

* From a value type to a base type.

* From a value type to an interface type that the type implements.

__Nullable Value Type conversions__

* From a type `T` to the type `T?`.

* From a type `T?` to a type `S?`, where there is a widening conversion from the type `T` to the type `S`.

* From a type `T` to a type `S?`, where there is a widening conversion from the type `T` to the type `S`.

* From a type `T?` to an interface type that the type `T` implements.

__String conversions__

* From `Char` to `String`.

* From `Char()` to `String`.

__Type Parameter conversions__

* From a type parameter to `Object`.

* From a type parameter to an interface type constraint or any interface variant compatible with an interface type constraint.

* From a type parameter to an interface implemented by a class constraint.

* From a type parameter to an interface variant compatible with an interface implemented by a class constraint.

* From a type parameter to a class constraint, or a base type of the class constraint.

* From a type parameter `T` to a type parameter constraint `Tx`, or anything `Tx` has a widening conversion to.

## Narrowing Conversions

Narrowing conversions are conversions that cannot be proved to always succeed, conversions that are known to possibly lose information, and conversions across domains of types sufficiently different to merit narrowing notation. The following conversions are classified as narrowing conversions:

__Boolean conversions__

* From `Boolean` to `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single`, or `Double`.

* From `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single`, or `Double` to `Boolean`.

__Numeric conversions__

* From `Byte` to `SByte`.

* From `SByte` to `Byte`, `UShort`, `UInteger`, or `ULong`.

* From `UShort` to `Byte`, `SByte`, or `Short`.

* From `Short` to `Byte`, `SByte`, `UShort`, `UInteger`, or `ULong`.

* From `UInteger` to `Byte`, `SByte`, `UShort`, `Short`, or `Integer`.

* From `Integer` to `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, or `ULong`.

* From `ULong` to `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, or `Long`.

* From `Long` to `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, or `ULong`.

* From `Decimal` to `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, or `Long`.

* From `Single` to `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, or `Decimal`.

* From `Double` to `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, or `Single`.

* From a numeric type to an enumerated type.

* From an enumerated type to a numeric type its underlying numeric type has a narrowing conversion to.

* From an enumerated type to another enumerated type.

__Reference conversions__

* From a reference type to a more derived type.

* From a class type to an interface type, provided the class type does not implement the interface type or an interface type variant compatible with it.

* From an interface type to a class type.

* From an interface type to another interface type, provided there is no inheritance relationship between the two types and provided they are not variant compatible.

__Anonymous Delegate conversions__

* From an anonymous delegate type generated for a lambda method reclassification to any narrower delegate type.

__Array conversions__

* From an array type `S` with an element type `Se`, to an array type `T` with an element type `Te`, provided that all of the following are true:

  * `S` and `T` differ only in element type.
  * Both `Se` and `Te` are reference types or are type parameters not known to be value types.
  * A narrowing reference, array, or type parameter conversion exists from `Se` to `Te`.

* From an array type `S` with an element type `Se` to an array type `T` with an enumerated element type `Te`, provided all of the following are true:

  * `S` and `T` differ only in element type.
  * `Se` is the underlying type of `Te` , or they are both different enumerated types that share the same underlying type.

* From an array type `S` of rank 1 with an enumerated element type `Se`, to `IList(Of Te)`, `IReadOnlyList(Of Te)`, `ICollection(Of Te)`, `IReadOnlyCollection(Of Te)` and `IEnumerable(Of Te)`, provided one of the following is true:

  * Both `Se` and `Te` are reference types or are type parameters known to be a reference type, and a narrowing reference, array, or type parameter conversion exists from `Se` to `Te`; or
  * `Se` is the underlying type of `Te`, or they are both different enumerated types that share the same underlying type.

__Value type conversions__

* From a reference type to a more derived value type.

* From an interface type to a value type, provided the value type implements the interface type.

__Nullable Value Type conversions__

* From a type `T?` to a type `T`.

* From a type `T?` to a type `S?`, where there is a narrowing conversion from the type `T` to the type `S`.

* From a type `T` to a type `S?`, where there is a narrowing conversion from the type `T` to the type `S`.

* From a type `S?` to a type `T`, where there is a conversion from the type `S` to the type `T`.

__String conversions__

* From `String` to `Char`.

* From `String` to `Char()`.

* From `String` to `Boolean` and from `Boolean` to `String`.

* Conversions between `String` and `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single`, or `Double`.

* From `String` to `Date` and from `Date` to `String`.

__Type Parameter conversions__

* From `Object` to a type parameter.

* From a type parameter to an interface type, provided the type parameter is not constrained to that interface or constrained to a class that implements that interface.

* From an interface type to a type parameter.

* From a type parameter to a derived type of a class constraint.

* From a type parameter `T` to anything a type parameter constraint `Tx` has a narrowing conversion to.

## Type Parameter Conversions

Type parameters' conversions are determined by the constraints, if any, put on them. A type parameter `T` can always be converted to itself, to and from `Object`, and to and from any interface type. Note that if the type `T` is a value type at run-time, converting from `T` to `Object` or an interface type will be a boxing conversion and converting from `Object` or an interface type to `T` will be an unboxing conversion. A type parameter with a class constraint `C` defines additional conversions from the type parameter to `C` and its base classes, and vice versa. A type parameter `T` with a type parameter constraint `Tx` defines a conversion to `Tx` and anything `Tx` converts to.

An array whose element type is a type parameter with an interface constraint `I` has the same covariant array conversions as an array whose element type is `I`, provided that the type parameter also has a `Class` or class constraint (since only reference array element types can be covariant). An array whose element type is a type parameter with a class constraint `C` has the same covariant array conversions as an array whose element type is `C`.

The above conversions rules do not permit conversions from unconstrained type parameters to non-interface types, which may be surprising. The reason for this is to prevent confusion about the semantics of such conversions. For example, consider the following declaration:

```vb
Class X(Of T)
    Public Shared Function F(t As T) As Long 
        Return CLng(t)    ' Error, explicit conversion not permitted
    End Function
End Class
```

If the conversion of `T` to `Integer` were permitted, one might easily expect that `X(Of Integer).F(7)` would return `7L`. However, it would not, because numeric conversions are only considered when the types are known to be numeric at compile time. In order to make the semantics clear, the above example must instead be written:

```vb
Class X(Of T)
    Public Shared Function F(t As T) As Long
        Return CLng(CObj(t))    ' OK, conversions permitted
    End Function
End Class
```

## User-Defined Conversions

*Intrinsic conversions* are conversions defined by the language (i.e. listed in this specification), while *user-defined conversions* are defined by overloading the `CType` operator. When converting between types, if no intrinsic conversions are applicable then user-defined conversions will be considered. If there is a user-defined conversion that is *most specific* for the source and target types, then the user-defined conversion will be used. Otherwise, a compile-time error results. The most specific conversion is the one whose operand is "closest" to the source type and whose result type is "closest" to the target type. When determining what user-defined conversion to use, the most specific widening conversion will be used; if no widening conversion is most specific, the most specific narrowing conversion will be used. If there is no most specific narrowing conversion, then the conversion is undefined and a compile-time error occurs.

The following sections cover how the most specific conversions are determined. They use the following terms:

If an intrinsic widening conversion exists from a type `A` to a type `B`, and if neither `A` nor `B` are interfaces, then `A` is *encompassed* by `B`, and `B` *encompasses* `A`.

The *most encompassing* type in a set of types is the one type that encompasses all other types in the set. If no single type encompasses all other types, then the set has no most encompassing type. In intuitive terms, the most encompassing type is the "largest" type in the set -- the one type to which each of the other types can be converted through a widening conversion.

The *most encompassed* type in a set of types is the one type that is encompassed by all other types in the set. If no single type is encompassed by all other types, then the set has no most encompassed type. In intuitive terms, the most encompassed type is the "smallest" type in the set -- the one type that can be converted to each of the other types through a narrowing conversion.

When collecting the candidate user-defined conversions for a type `T?`, the user-defined conversion operators defined by `T` are used instead. If the type being converted to is also a nullable value type, then any of `T`'s user-defined conversions operators that involve only non-nullable value types are lifted. A conversion operator from `T` to `S` is lifted to be a conversion from `T?` to `S?` and is evaluated by converting `T?` to `T`, if necessary, then evaluating the user-defined conversion operator from `T` to `S` and then converting `S` to `S?`, if necessary. If the value being converted is `Nothing`, however, a lifted conversion operator converts directly into a value of `Nothing` typed as `S?`. For example:

```vb
Structure S
    ...
End Structure

Structure T
    Public Shared Widening Operator CType(ByVal v As T) As S
        ...
    End Operator
End Structure

Module Test
    Sub Main()
        Dim x As T?
        Dim y As S?

        y = x                ' Legal: y is still null
        x = New T()
        y = x                ' Legal: Converts from T to S
    End Sub
End Module
```

When resolving conversions, user-defined conversions operators are always preferred over lifted conversion operators. For example:

```vb
Structure S
    ...
End Structure

Structure T
    Public Shared Widening Operator CType(ByVal v As T) As S
        ...
    End Operator

    Public Shared Widening Operator CType(ByVal v As T?) As S?
        ...
    End Operator
End Structure

Module Test
    Sub Main()
        Dim x As T?
        Dim y As S?

        y = x                ' Calls user-defined conversion, not lifted conversion
    End Sub
End Module
```

At run-time, evaluating a user-defined conversion can involve up to three steps:

1. First, the value is converted from the source type to the operand type using an intrinsic conversion, if necessary.

2. Then, the user-defined conversion is invoked.

3. Finally, the result of the user-defined conversion is converted to the target type using an intrinsic conversion, if necessary.

It is important to note that evaluation of a user-defined conversion will never involve more than one user-defined conversion operator.

### Most Specific Widening Conversion

Determining the most specific user-defined widening conversion operator between two types is accomplished using the following steps:

1. First, all of the candidate conversion operators are collected. The candidate conversion operators are all of the user-defined widening conversion operators in the source type and all of the user-defined widening conversion operators in the target type.

2. Then, all non-applicable conversion operators are removed from the set. A conversion operator is applicable to a source type and target type if there is an intrinsic widening conversion operator from the source type to the operand type and there is an intrinsic widening conversion operator from the result of the operator to the target type. If there are no applicable conversion operators, then there is no most specific widening conversion.

3. Then, the most specific source type of the applicable conversion operators is determined:

   * If any of the conversion operators convert directly from the source type, then the source type is the most specific source type.

   * Otherwise, the most specific source type is the most encompassed type in the combined set of source types of the conversion operators. If no most encompassed type can be found, then there is no most specific widening conversion.

4. Then, the most specific target type of the applicable conversion operators is determined:

   * If any of the conversion operators convert directly to the target type, then the target type is the most specific target type.

   * Otherwise, the most specific target type is the most encompassing type in the combined set of target types of the conversion operators. If no most encompassing type can be found, then there is no most specific widening conversion.

5. Then, if exactly one conversion operator converts from the most specific source type to the most specific target type, then this is the most specific conversion operator. If more than one such operator exists, then there is no most specific widening conversion.

### Most Specific Narrowing Conversion

Determining the most specific user-defined narrowing conversion operator between two types is accomplished using the following steps:

1. First, all of the candidate conversion operators are collected. The candidate conversion operators are all of the user-defined conversion operators in the source type and all of the user-defined conversion operators in the target type.

2. Then, all non-applicable conversion operators are removed from the set. A conversion operator is applicable to a source type and target type if there is an intrinsic conversion operator from the source type to the operand type and there is an intrinsic conversion operator from the result of the operator to the target type. If there are no applicable conversion operators, then there is no most specific narrowing conversion.

3. Then, the most specific source type of the applicable conversion operators is determined:

   * If any of the conversion operators convert directly from the source type, then the source type is the most specific source type.

   * Otherwise, if any of the conversion operators convert from types that encompass the source type, then the most specific source type is the most encompassed type in the combined set of source types of those conversion operators. If no most encompassed type can be found, then there is no most specific narrowing conversion.

   * Otherwise, the most specific source type is the most encompassing type in the combined set of source types of the conversion operators. If no most encompassing type can be found, then there is no most specific narrowing conversion.

4. Then, the most specific target type of the applicable conversion operators is determined:

   * If any of the conversion operators convert directly to the target type, then the target type is the most specific target type.

   * Otherwise, if any of the conversion operators convert to types that are encompassed by the target type, then the most specific target type is the most encompassing type in the combined set of source types of those conversion operators. If no most encompassing type can be found, then there is no most specific narrowing conversion.

   * Otherwise, the most specific target type is the most encompassed type in the combined set of target types of the conversion operators. If no most encompassed type can be found, then there is no most specific narrowing conversion.

5. Then, if exactly one conversion operator converts from the most specific source type to the most specific target type, then this is the most specific conversion operator. If more than one such operator exists, then there is no most specific narrowing conversion.

## Native Conversions

Several of the conversions are classified as *native conversions* because they are supported natively by the .NET Framework. These conversions are ones that can be optimized through the use of the `DirectCast` and `TryCast` conversion operators, as well as other special behaviors. The conversions classified as native conversions are: identity conversions, default conversions, reference conversions, array conversions, value type conversions, and type parameter conversions.

## Dominant Type

Given a set of types, it is often necessary in situations such as type inference to determine the *dominant type* of the set. The dominant type of a set of types is determined by first removing any types that one or more other types do not have an implicit conversion to. If there are no types left at this point, there is no dominant type. The dominant type is then the most encompassed of the remaining types. If there is more than one type that is most encompassed, then there is no dominant type.
