# Visual Basic Language Design Meeting - 2017.08.09

### Agenda
* Await in Catch/Finally
* Non-trailing named arguments

## Await in Catch/Finally

**Differences from C#**

VB allows jumping (`goto`) from the `Catch` block.

``` VB.NET
Dim retryCount = 0
Try
    retry:
Catch ex As Exception When retryCount < 3
    retryCount += 1
    GoTo retry
End Try
```

Decision: The feature is approved in principle but needs its priority driven by other platform changes such as `IAsyncDisposable/Async Using`.

## Non-Trailing Named Arguments

### Attributes
Because in VB named argument syntax does not correspond to constructor arguments but field/property initialization this is not permitted in attribute.

``` VB.NET
<FooAttribute(0, Name:="A")>
```

### Late-Binding
Late-bound method invocations which have non-trailing named arguments raise a compile-time error.

Action: Make sure we produce an error _after_ overload-resolution, not making method groups inapplicable.