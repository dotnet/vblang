# Continue Exit For Identifier

* [x] [Proposed](https://github.com/dotnet/vblang/issues/186) 
* [*] Prototype: [Prototype/ContinueForIdentifier](https://github.com/AdamSpeight2008/roslyn-AdamSpeight2008/tree/Prototpye/ContinueForID)
* [ ] Implementation: [Not Started](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

## Summary
[summary]: #summary

Extend the capibility of `Contine For` and `Exit For` to enable continuing or exitting a loop nested higher.
Eg. `Continue For x` and `Continue For x`

-------

## Motivation
[motivation]: #motivation

* Why are we doing this?
  * Seems like natural extension to the language, give the `Next` can also have an identifer. eg `Next x`
* What use cases does it support?
  * It can difficult to implement, resorting to using `label` and `GoTo` eg `GoTo start_of_loop_x` 
* What is the expected outcome?
  * Without an identifier eg `Continue For` or `Exit For` will perform exactly as it does now.
  * With an identifier
    * `Continue For x` will continue with the next iterator of the loop named with the identifier `x`.
    * `Exit For x` will exit the loop named with the identifier `x`. Resuming execution at the next statement after it.
  
------

## Detailed design
[design]: #detailed-design

Add an optional child node to `ExitStatementSyntax`
```xml
    <!--****************
      -  Exit
      ******************-->
    <node-structure name="ExitStatementSyntax" parent="ExecutableStatementSyntax">
      <description>An exit statement. The kind of block being exited can be found by examining the Kind.</description>
      <lm-equiv name="ExitNode"></lm-equiv>
      <native-equiv name=""></native-equiv>
      <spec-section>10.11</spec-section>
      <grammar>ExitStatement</grammar>
      <node-kind name="ExitDoStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitDo" />
      </node-kind>
      <node-kind name="ExitForStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitFor" />
      </node-kind>
      <node-kind name="ExitSubStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitSub" />
      </node-kind>
      <node-kind name="ExitFunctionStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitFunction" />
      </node-kind>
      <node-kind name="ExitOperatorStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitOperator" />
      </node-kind>
      <node-kind name="ExitPropertyStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitProperty" />
      </node-kind>
      <node-kind name="ExitTryStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitTry" />
      </node-kind>
      <node-kind name="ExitSelectStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitSelect" />
      </node-kind>
      <node-kind name="ExitWhileStatement">
        <lm-equiv name="Exit" />
        <native-equiv name="Statement.Opcodes.ExitWhile" />
      </node-kind>
      <child name="ExitKeyword" kind="ExitKeyword">
        <description>The "Exit" keyword.</description>
        <lm-equiv name="ExitKeyword"></lm-equiv>
        <native-equiv name=""></native-equiv>
      </child>
      <child name="BlockKeyword" >
        <description>The keyword describing the block to exit.</description>
        <lm-equiv name="Block"></lm-equiv>
        <native-equiv name=""></native-equiv>
        <kind name="DoKeyword" node-kind="ExitDoStatement"/>
        <kind name="ForKeyword" node-kind="ExitForStatement"/>
        <kind name="SubKeyword" node-kind="ExitSubStatement"/>
        <kind name="FunctionKeyword" node-kind="ExitFunctionStatement"/>
        <kind name="OperatorKeyword" node-kind="ExitOperatorStatement"/>
        <kind name="PropertyKeyword" node-kind="ExitPropertyStatement"/>
        <kind name="TryKeyword" node-kind="ExitTryStatement"/>
        <kind name="SelectKeyword" node-kind="ExitSelectStatement"/>
        <kind name="WhileKeyword" node-kind="ExitWhileStatement"/>
      </child>
      <child name="ControlVariable" optional="true" kind="@ExpressionSyntax" />
    </node-structure>
```

Add an optional child node to `ContinueStatementSyntax`
```xml
<!--****************
      -  Continue
      ******************-->
    <node-structure name="ContinueStatementSyntax" parent="ExecutableStatementSyntax">
      <description>Represents a "Continue (block)" statement. THe kind of block referenced can be determined by examining the Kind.</description>
      <lm-equiv name="ContinueNode"></lm-equiv>
      <native-equiv name="Statement"></native-equiv>
      <spec-section>10.11</spec-section>
      <grammar>ContinueStatement</grammar>
      <!-- REVIEW: What kind if just "Continue" is typed? Should there be different kinds? -->
      <node-kind name="ContinueWhileStatement">
        <lm-equiv name="Continue" />
        <native-equiv name="Statement.Opcodes.ContinueWhile" />
      </node-kind>
      <node-kind name="ContinueDoStatement">
        <lm-equiv name="Continue" />
        <native-equiv name="Statement.Opcodes.ContinueDo" />
      </node-kind>
      <node-kind name="ContinueForStatement">
        <lm-equiv name="Continue" />
        <native-equiv name="Statement.Opcodes.ContinueFor" />
      </node-kind>
      <child name="ContinueKeyword" kind="ContinueKeyword">
        <description>The "Continue" keyword.</description>
        <lm-equiv name="ContinueKeyword"></lm-equiv>
        <native-equiv name=""></native-equiv>
      </child>
      <child name="BlockKeyword">
        <description>The "Do", "For" or "While" keyword that identifies the kind of loop being continued.</description>
        <lm-equiv name="Block"></lm-equiv>
        <native-equiv name=""></native-equiv>
        <kind name="DoKeyword" node-kind="ContinueDoStatement"/>
        <kind name="ForKeyword" node-kind="ContinueForStatement"/>
        <kind name="WhileKeyword" node-kind="ContinueWhileStatement"/>
      </child>
      <child name="ControlVariable" optional="true" kind="@ExpressionSyntax" />
    </node-structure>
```

Parse the optional identifier only if the statements are of the correct kind.
`ContinueForStatement` and `ExitForStatement`

Validate that the identifer is a loop identifer, within this method. 
And only if the identifier is a nesting higher or equal to the current nesting level. IE You can continue or exit an loop that is lower eg an inner loop.

This is the bulk of the proposal. Explain the design in enough detail for somebody familiar
with the language to understand, and for somebody familiar with the compiler to implement,  and include examples of how the feature is used. This section can start out light before the prototyping phase but should get into specifics and corner-cases as the feature is iteratively designed and implemented.

Example code:

```vbnet
Imports System        
Module M1
    Sub Main()
        dim continueLoop as Boolean = true
        For i = 0 To 2
            Console.WriteLine($"Loop i Block Start ({i})")
            For j = 0 To 2
              Console.WriteLine($"Loop j Block Start ({i})")
              If continueLoop Then
                Console.WriteLine("Continuing")
                continueLoop = false
                Continue For i
              End If
              Console.WriteLine("Exiting")
              Exit For i
              Console.WriteLine($"Loop j Block End ({i})")
            Next j
            Console.WriteLine("After Loop j")
            Console.WriteLine($"Loop i Block End ({i})")
        Next   
        Console.WriteLine("After Loop i") 
    End Sub
End Module
```
Output
```
Loop i Block Start (0)
Loop j Block Start (0)
Continuing
Loop i Block Start (1)
Loop j Block Start (0)
Exiting
After Loop i
```

```vbnet
Imports System        
Module M1
    Sub Main()
        dim continueLoop as Boolean = true
        Dim xs = {0, 1, 2}
        Dim ys = {0, 1, 2}
        For Each x In xs
            Console.WriteLine($"Loop x Block Start ({x})")
            For Each y In ys
              Console.WriteLine($"Loop y Block Start ({x})")
              If continueLoop Then
                Console.WriteLine("Continuing")
                continueLoop = false
                Continue For i
              End If
              Console.WriteLine("Exiting")
              Exit For i
              Console.WriteLine($"Loop y Block End ({y})")
            Next j
            Console.WriteLine("After Loop y")
            Console.WriteLine($"Loop x Block End ({x})")
        Next   
        Console.WriteLine("After Loop x") 
    End Sub
End Module
```
Output
```
Loop x Block Start (0)
Loop y Block Start (0)
Continuing
Loop x Block Start (1)
Loop y Block Start (0)
Exiting
After Loop x
```

This is implement in this proposal by modifying how the lowering is done, especially the labels use.
The labels used will now include the loop's identifier, which allows it to jump to correct location in the lowered code.

-----

## Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

------

## Alternatives
[alternatives]: #alternatives

What other designs have been considered? What is the impact of not doing this?

----

## Unresolved questions
[unresolved]: #unresolved-questions
What parts of the design are still TBD?

* **TIED TO VB15** need to be updated, if accepted.
* Checking of the validity of the identifer used in the `Continue` and `Exit`
* Additional error messages
* Additional use cases. *Not covered in this proposal, has it require provide a name for the loop constuct.*
  * `Do ... Loop`
  * 'While .. End While`

