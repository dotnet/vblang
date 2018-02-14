# Additional type comparison syntax: ```Is A``` and ```Is An```

* [x] Proposed
* [ ] Prototype: [Complete](https://github.com/PROTOTYPE_OWNER/roslyn/BRANCH_NAME)
* [ ] Implementation: [In Progress](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

## Summary
[summary]: #summary

Adding two new contextual keywords (```A``` and ```An```) to be used for type comparison.

## Motivation
[motivation]: #motivation

The ```TypeOf(obj)``` and ```GetType(TypeName)``` operators provide nothing but grief until you've learned 
the difference between the two. In the spirit of VB's English-like syntax, I suggest ```Is A``` (and ```Is An``` for grammatical
completeness) as alternatives to ```TypeOf(obj) Is```

## Detailed design
[design]: #detailed-design

Example code:

```
Function IsStringOrInteger(param As Object) As Boolean
    Return ((param Is A String) OrElse (param Is An Integer))
End Function
```

## Drawbacks
[drawbacks]: #drawbacks

The ```TypeOf(obj) Is TypeName``` syntax already exists, making this redundant.

## Alternatives
[alternatives]: #alternatives

N/A

## Unresolved questions
[unresolved]: #unresolved-questions

N/A
