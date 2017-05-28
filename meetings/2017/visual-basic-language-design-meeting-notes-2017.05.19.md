# Visual Basic Language Design Meeting Notes - 2017.05.19

## Agenda
* Default Interface Implementations

## Discussion

### Default Interface Implementations

#### Q: Are there any syntactic ambiguities around default interface implementations in VB? In C# a definition with a body or without one can be determined by the presence or absence of a semicolon.

**Follow-up: Need to look at parser and come back.**

#### Modifiers

| Modifier                                     | C#  | VB   | Notes |
| -------------------------------------------- | --- | ---- | ----- |
| Overridable (virtual) | Allowed, implied if there is a body | Same |  |
| MustOverride (abstract) | Implied if (not static or private) and there is no body | Same |  |
| NotOverridable (sealed) | Implied for static and private | Same | It's not permitted to make a NotOverridable Overrides. Means "non virtual" |
| Public/Private/Friend | Allowed with normal meaning | Same |  |
| Protected/Friend Protected/Private Protected | Allowed, but meaning is implementation specific |  |  |

#### Allowed members in interfaces with bodies

| Member | C#  | VB  | Notes |
| ------ | --- | --- | ----- |
| Instance methods | Yes |  |  |
| Shared methods | Yes |  |  |
| Instance fields | **No** |  |  |
| Shared fields | Yes |  |  |
| Instance constructors | No |  |  |
| Shared constructors | TBD |  |  |
| Instance finalizers | No |  |  |
| Instance properties | Yes, but not auto |  |  |
| Shared properties | Yes, auto is TBD |  |  |
| Default properties (indexers) | N/A | Same as other properties |  |
| Instance events | Yes, but must be Custom |  | Should have same restrictions as in classes (private `RaiseEvent`) |
| Shared events | Yes, auto is TBD |  |  |
| Nested types | Yes | Yes |  |
| Operators | TBD |  |  |
| Conversion operators | No | Same |  |

**Follow-up: Verify event behavior makes sense with regard to WinRT events.**

#### Q: Do we allow `Implements` clause?

``` VB.NET
Interface IBase

    Sub M()

End Interface 

Interface IDerived
    Inherits IBase

    ~~Overridable Sub N() Implements IBase.M~~

    ~~End Sub~~

    NotOverridable Sub N() Implements IBase.M

    End Sub
End Interface
```

**Yes**. `NotOverridable` allowed and implied.

#### What does `MyClass` do in an interface?
Nothing special; it allows non-virtual calls.

#### Q: Should we allow or forbid implicit interface implementations (like C#) now?
Because of the way the CLR looks up interface implementations (by name), if an interface declares an overridable member _and_ an implementor declares or inherits a public `Overridable`/`Overrides` member of the same name it will implicitly be picked up as the implementation/override of that interface member.

**Options**
* Make it an error
* Recognize behavior (because we don't have a choice)
* Allow `Overridable` or `MustOverride` methods to implement (interface members with bodies).

**Decision Let's make it an error for now and evaluate implicit interface implementations separately.**
