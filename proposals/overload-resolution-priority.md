# Overload Resolution Priority

## Summary
[summary]: #summary

We introduced a new attribute, `System.Runtime.CompilerServices.OverloadResolutionPriority`, that can be used by API authors to adjust the relative priority of
overloads within a single type as a means of steering API consumers to use specific APIs, even if those APIs would normally be considered ambiguous or otherwise
not be chosen by overload resolution rules. See [Overload Resolution Priority in C#](https://github.com/dotnet/csharplang/blob/main/proposals/csharp-13.0/overload-resolution-priority.md).

## Motivation
[motivation]: #motivation

API authors often run into an issue of what to do with a member after it has been obsoleted. For backwards compatibility purposes, many will keep the existing member around
with `ObsoleteAttribute` set to error in perpetuity, in order to avoid breaking consumers who upgrade binaries at runtime. This particularly hits plugin systems, where the
author of a plugin does not control the environment in which the plugin runs. The creator of the environment may want to keep an older method present, but block access to it
for any newly developed code. However, `ObsoleteAttribute` by itself is not enough. The type or member is still visible in overload resolution, and may cause unwanted overload
resolution failures when there is a perfectly good alternative, but that alternative is either ambiguous with the obsoleted member, or the presence of the obsoleted member causes
overload resolution to end early without ever considering the good member. For this purpose, we want to have a way for API authors to guide overload resolution on resolving the
ambiguity, so that they can evolve their API surface areas and steer users towards performant APIs without having to compromise the user experience.

## Detailed Design
[detailed-design]: #detailed-design

### Overload resolution priority

We define a new concept, ***overload_resolution_priority***, which is used during the process of overload resolution. ***overload_resolution_priority*** is a 32-bit integer
value. All methods have an ***overload_resolution_priority*** of 0 by default, and this can be changed by applying
[`OverloadResolutionPriorityAttribute`](#systemruntimecompilerservicesoverloadresolutionpriorityattribute) to a method. We update section 
[Overloaded Method Resolution](https://github.com/dotnet/vblang/blob/main/spec/overload-resolution.md#overloaded-method-resolution) of the VB specification as
follows (change in **bold**):

> 2.  Next, eliminate all members from the set that are inaccessible or not applicable (Section [Applicability To Argument List](overload-resolution.md#applicability-to-argument-list)) to the argument list

> **3. Then, the reduced set of candidate members is grouped by declaring type.
>      If the member is an override, the declaring type and the ***overload_resolution_priority*** come from the least-derived declaration of that member.
>      Within each group:**
 
> - **A maximum ***overload_resolution_priority*** among candidates that do not utilize narrowing conversions or narrowing delegate relaxation is determined.**
> - **All members that have a lower ***overload_resolution_priority*** than the maximum found during the previous step, if any, within its declaring type group are removed.**
 
> **The reduced groups are then recombined into the set of candidates.**

>  **4**.  Next, if one or more arguments are `AddressOf` or lambda expressions, then calculate the *delegate relaxation levels* for each such argument as below. If the worst (lowest) delegate relaxation level in `N` is worse than the lowest delegate relaxation level in `M`, then eliminate `N` from the set. The delegate relaxation levels are as follows:

As an example, this feature would cause the following code snippet to print "I1", rather than failing compilation due to an ambiguity:

```vb
Public Interface I1
End Interface
Public Interface I2
End Interface
Public Interface I3
    Inherits I1, I2
End Interface

Public Class C
    <OverloadResolutionPriority(1)>
    Public Shared Sub M(x As I1)
        System.Console.WriteLine("I1")
    End Sub
    Public Shared Sub M(x As I2)
        System.Console.WriteLine("I2")
    End Sub
End Class

Public Class Program
    Shared Sub Main()
        Dim i3 As I3 = Nothing
        C.M(i3)
    End Sub
End Class
```

Negative numbers are allowed to be used, and can be used to mark a specific overload as worse than all other default overloads.

The **overload_resolution_priority** of a member comes from the least-derived declaration of that member. **overload_resolution_priority** is not
inherited or inferred from any interface members a type member may implement, and given a member `Mx` that implements an interface member `Mi`, no
warning is issued if `Mx` and `Mi` have different **overload_resolution_priorities**.

The candidates that utilize a narrowing conversion or a narrowing delegate relaxation are excluded from determining the maximum **overload_resolution_priority**
because there is no guarantee that those candidates can be successfully invoked with the supplied arguments.
For example, consider the following scenario:
``` vb
Module Module1

    Sub Main()
        M1(New C0())
    End Sub

    <System.Runtime.CompilerServices.OverloadResolutionPriority(1)>
    Sub M1(x As C1)
    End Sub

    Sub M1(x As C2)
        System.Console.Write(2)
    End Sub
End Module

Class C0
    Public Shared Narrowing Operator CType(x As C0) As C1
        Throw New System.InvalidCastException()
    End Operator
    Public Shared Widening Operator CType(x As C0) As C2
        Return New C2()
    End Operator
End Class

Class C1
End Class

Class C2
End Class
```

When it is compiled with `Option Strict On`, `Sub M1(x As C1)` is not applicable because it cannot be called without a
narrowing conversion. The only applicable candidate is `M1(x As C2)` and it is called.

When the code is compiled with `Option Strict Off`, both methods are applicable. If the priority was given to `Sub M1(x As C1)`,
a candidate that uses only widening conversions would be filtered out, resulting in an `InvalidCastException` thrown during execution.

Here is another example:
``` vb
Option Strict Off

Module Module1

    Sub Main()
        M1(CObj(New C2()))
    End Sub

    <System.Runtime.CompilerServices.OverloadResolutionPriority(1)>
    Sub M1(x As I1)
        System.Console.Write(1)
    End Sub

    Sub M1(x As I2)
        System.Console.Write(2)
    End Sub
End Module

Interface I1
End Interface

Interface I2
End Interface

Class C2
    Implements I2
End Class
```

In this case both candidates require narrowing conversion. Since filtering doesn't happen, the result of overload resolution
is a late-bound invocation, which determines that `Sub M1(x As I1)` is inapplicable and invokes `Sub M1(x As I2)`.

If the priority was given to `Sub M1(x As I1)` at compile time, it would remain the only applicable candidate and would be
invoked early-bound, resulting in an `InvalidCastException` thrown during execution.

### `System.Runtime.CompilerServices.OverloadResolutionPriorityAttribute`

There is the following attribute in the BCL:

```cs
namespace System.Runtime.CompilerServices;

[AttributeUsage(AttributeTargets.Method | AttributeTargets.Constructor | AttributeTargets.Property, AllowMultiple = false, Inherited = false)]
public sealed class OverloadResolutionPriorityAttribute(int priority) : Attribute
{
    public int Priority => priority;
}
```

All methods and properties in VB have a default ***overload_resolution_priority*** of 0, unless they are attributed with `OverloadResolutionPriorityAttribute`.
If they are attributed with that attribute, then their ***overload_resolution_priority*** is the integer value provided to the first argument of the attribute.

By analogy with C#, it is an error to apply `OverloadResolutionPriorityAttribute` to the following locations:

* Property, or event accessors
* Conversion operators
* Finalizers
* Shared constructors
* Overriding properties
* Overriding methods 

Attributes encountered on these locations in metadata effectively will have no impact in VB code.

### Langversion Behavior

Overload Resolution process does not perform filtering by ***overload_resolution_priority*** in VB < 17.13.
No errors or warnings issued by the Overload Resolution process due to the fact that ***overload_resolution_priority***
was ignored in this case.

By analogy with C#, a langversion errors is issued when `OverloadResolutionPriorityAttribute` is applied in VB < 17.13.
