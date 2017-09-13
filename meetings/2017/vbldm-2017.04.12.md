Agenda
======

-   New conversion operator/syntax

-   GitHub repo review

Guest attendee: Klaus Spoonman

New conversion operator/syntax
==============================

Scenario
--------

Today VB has several casting syntaxes, each with subtle differences and performance implications. Nonetheless there are still some CLR conversions which are either inexpressible in VB or are only emitted with specific compilation options.

### CType Conversions

CType is the all-around conversion operator supporting the widest set of conversions in the language. CType has several short-hand conversion operators in the language, e.g. CStr, CInt, that are just aliases for CType with intrinsic destination types, e.g. CType(…, String). Depending on the situation CType may emit a single IL instruction, a method call, or a combination.

CType supports the following conversion types:

-   Runtime conversions

    -   Boolean -&gt; Integer (True = -1)

-   Unboxing conversion

-   Reference conversions (upcasts/downcasts)

-   Numeric conversions

    -   Integral &lt;-&gt; Integral (opcode)

    -   Floating point -&gt; Integral (Math.Round)

-   Nullable conversions

-   String conversions (Char &lt;-&gt; String, Char() &lt;-&gt; String)

-   User-defined conversion operators

-   Other language conversions

CType does not support the following conversion types:

-   Integer &lt;-&gt; Char (AscW, ChrW)

-   Truncation

-   Unchecked integral conversion (chopping)

Additionally CType will always behave like DirectCast for type parameter conversions.

### DirectCast Conversions

DirectCast supports native reference conversions only.

Proposal 1
----------

<https://github.com/dotnet/vblang/issues/59>

As Type
-------

-   Runtime conversions

    -   Boolean -&gt; Integer (True = -1) (error)

-   Unboxing conversion

-   Reference conversions (upcasts/downcasts)

-   Numeric conversions

    -   Integral &lt;-&gt; Integral (opcode)

    -   Floating point -&gt; Integral (Math.Round)

-   Nullable conversions

-   String conversions (Char &lt;-&gt; String, Char() &lt;-&gt; String)

-   User-defined conversion operators

-   Other language conversions

TryCast like behavior – return null? Odd to throw sometimes and return null sometimes.

(T)obj -&gt; Cast obj

obj as T -&gt; TryCast(obj, T)

Proposal 2
----------

Magic method fix – emit special code for CInt(Math.Round(*floating-point*)) and CInt(Math.Truncate(*floating-point*)) and CInt(Int/Fix(*floating-point*))

Proposal 3
----------

Expand DirectCast to support native numeric conversions *but not* user-defined conversions.

Proposal 4
==========

Checked/Unchecked is orthogonal

Update TryCast to convert to nullable value types.

CVal(value, T) only for numeric types and enumerations and char.

GitHub repo review
==================
