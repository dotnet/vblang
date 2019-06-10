# Continue Exit For Identifier

* [x] [Proposed](https://github.com/dotnet/vblang/issues/186) 
* [x] Prototype: [Prototype/ContinueForIdentifier](https://github.com/AdamSpeight2008/roslyn-AdamSpeight2008/tree/Prototpye/ContinueForID) 
* [ ] Implementation: [Not Started](https://github.com/dotnet/roslyn/BRANCH_NAME)
* [ ] Specification: [Not Started](pr/1)

## Summary
[summary]: #summary

Extend the capibility of `Contine For` and `Exit For` to enable continuing or exitting a loop nested higher.
Eg. `Continue For x` and `Exit For x`

-------

## Motivation
[motivation]: #motivation

* Why are we doing this?
  * Seems like natural extension to the language, given that the `Next` can also have an identifer. eg `Next x`
  * It can be difficult to implement workarounds, or require used different loop constructs at different nestings.
  * Often implemented via `label`s and `GoTo`s. eg `GoTo start_of_loop_x`.
* What use cases does it support?
  * This proposal is tightly focused on two cases;
    * `Continue For loop_identifier`
    * `Exit For loop_identifier`
  * It can difficult to implement, resorting to using `label` and `GoTo` eg `GoTo start_of_loop_x` 
* What is the expected outcome?
  * Without an identifier eg `Continue For` or `Exit For` will perform exactly as it does now.
  * With an identifier
    * `Continue For x` will continue with the next iterator of the loop named with the identifier `x`.
    * `Exit For x` will exit the loop named with the identifier `x`. Resuming execution at the next statement after it.
  
------

## Detailed design
[design]: #detailed-design

### Syntax Node Changes
  An additional optional child node `<child name="ControlVariable" optional="true" kind="@ExpressionSyntax" />` will added to
  * `ExitStatementSyntax`
  * `ContinueStatementSyntax`
  This additional node is used to identify which loop identifier to the statement refers to.
  If the identifier is absent, the statement refers to current loop.
  
<details><summary>ExitStatementSyntax change</summary><p>

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
</p></details>

<details><summary>ContinueStatementSyntax change</summary><p>

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
</p></details>

### Parsing Changes
During parsing of the `ContinueForStatement` and `ExitForStatement` an additional check is performed to see if there is an identifer present. 

Parse the optional identifier only if the statements are of the correct kind.
`ContinueForStatement` and `ExitForStatement`

### Binding Changes
During binding of the `ContinueForStatement` and `ExitForStatment`, we must validate that the identifer is a loop identifer, of a current or higher nesting level.

### Lowering Changes
The proposal achieves this via modifying how the `For` and `For Each` lowering are done. By extending the labels using during the lowering, to now include the loop's identifier. eg `StartOfForLoop_OuterLoop:` and `ExitForLoop_InnerLoop:`
So when the `Exit` or `Continue` is lowered, the corrispondig `GoTo` is to the correct location (thus correct nested loop) in the lowered code. eg `GoTo StartOfForLoop_OuterLoop` and `GoTo ExitForLoop_InnerLoop`

----------

<details><summary>Example Code:</summary><p>

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

<details><summary>Expected Output</summary><p>
 
```
Loop i Block Start (0)
Loop j Block Start (0)
Continuing
Loop i Block Start (1)
Loop j Block Start (0)
Exiting
After Loop i
```

</p></details>

</p></details>


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
  * `While .. End While`

