# Statements

Statements represent executable code.

```antlr
Statement
    : LabelDeclarationStatement
    | LocalDeclarationStatement
    | WithStatement
    | SyncLockStatement
    | EventStatement
    | AssignmentStatement
    | InvocationStatement
    | ConditionalStatement
    | LoopStatement
    | ErrorHandlingStatement
    | BranchStatement
    | ArrayHandlingStatement
    | UsingStatement
	| AwaitStatement
	| YieldStatement
    ;
```

__Note.__ The Microsoft Visual Basic Compiler only allows statements which start with a keyword or an identifier. Thus, for instance, the invocation statement "`Call (Console).WriteLine`" is allowed, but the invocation statement "`(Console).WriteLine`" is not.

## Control Flow

*Control flow* is the sequence in which statements and expressions are executed. The order of execution depends on the particular statement or expression.

For example, when evaluating an addition operator (Section [Addition Operator](expressions.md#addition-operator)), first the left operand is evaluated, then the right operand, and then the operator itself. Blocks are executed (Section [Blocks and Labels](statements.md#blocks-and-labels)) by first executing their first substatement, and then proceeding one by one through the statements of the block.

Implicit in this ordering is the concept of a *control point*, which is the next operation to be executed. When a method is invoked (or "called"), we say it creates an *instance* of the method. A method instance consists of its own copy of the method's parameters and local variables, and its own control point.

### Regular Methods

Here is an example of a regular method

```vb
Function Test() As Integer
    Console.WriteLine("hello")
    Return 1
End Sub

Dim x = Test()    ' invokes the function, prints "hello", assigns 1 to x
```

When a regular method is invoked,

1. First an instance of the method is created specific to that invocation. This instance includes a copy of all parameters and local variables of the method.
2. Then all of its parameters are initialized to the supplied values, and all of its local variables to the default values of their types.
3. In the case of a `Function`, an implicit local variable is also initialized called the *function return variable* whose name is the function's name, whose type is the return type of the function and whose initial value is the default of its type.
4. The method instance's control point is then set at the first statement of the method body, and the method body immediately starts to execute from there (Section [Blocks and Labels](statements.md#blocks-and-labels)).

When control flow exits the method body normally - through reaching the `End Function` or `End Sub` that mark its end, or through an explicit `Return` or `Exit` statement - control flow returns to the caller of the method instance. If there is a function return variable, the result of the invocation is the final value of this variable.

When control flow exits the method body through an unhandled exception, that exception is propagated to the caller.

After control flow has exited, there are no longer any live references to the method instance. If the method instance held the only references to its copy of local variables or parameters, then they may be garbage collected.

### Iterator Methods

Iterator methods are used as a convenient way to generate a sequence, one which can be consumed by the `For Each` statement. Iterator methods use the `Yield` statement (Section [Yield Statement](statements.md#yield-statement)) to provide elements of the sequence. (An iterator method with no `Yield` statements will produce an empty sequence). Here is an example of an iterator method:

```vb
Iterator Function Test() As IEnumerable(Of Integer)
    Console.WriteLine("hello")
    Yield 1
    Yield 2
End Function

Dim en = Test()
For Each x In en          ' prints "hello" before the first x
    Console.WriteLine(x)  ' prints "1" and then "2"
Next
```

When an iterator method is invoked whose return type is `IEnumerator(Of T)`,

1. First an instance of the iterator method is created specific to that invocation. This instance includes a copy of all parameters and local variables of the method.
2. Then all of its parameters are initialized to the supplied values, and all of its local variables to the default values of their types.
3. An implicit local variable is also initialized called the *iterator current variable*, whose type is `T` and whose initial value is the default of its type.
4. The method instance's control point is then set at the first statement of the method body.
5. An *iterator object* is then created, associated with this method instance. The iterator object implements the declared return type and has behavior as described below.
6. Control flow is then resumed *immediately* in the caller, and the result of the invocation is the iterator object. Note that this transfer is done without exiting the iterator method instance, and does not cause finally handlers to execute. The method instance is still referenced by the iterator object, and will not be garbage collected so long as there exists a live reference to the iterator object.

When the iterator object's `Current` property is accessed, the *current variable* of the invocation is returned.

When the iterator object's `MoveNext` method is invoked, the invocation does not create a new method instance. Instead the existing method instance is used (and its control point and local variables and parameters) - the instance that was created when the iterator method was first invoked. Control flow resumes execution at the control point of that method instance, and proceeds through the body of the iterator method as normal.

When the iterator object's `Dispose` method is invoked, again the existing method instance is used. Control flow resumes at the control point of that method instance, but then immediately behaves as if an `Exit Function` statement were the next operation.

The above descriptions of behavior for invocation of `MoveNext` or `Dispose` on an iterator object only apply if all previous invocations of `MoveNext` or `Dispose` on that iterator object have already returned to their callers. If they haven't, then the behavior is undefined.

When control flow exits the iterator method body normally -- through reaching the `End Function` that mark its end, or through an explicit `Return` or `Exit` statement -- it must have done so in the context of an invocation of `MoveNext` or `Dispose` function on an iterator object to resume the iterator method instance, and it will have been using the method instance that was created when the iterator method was first invoked. The control point of that instance is left at the `End Function` statement, and control flow resumes in the caller; and if it had been resumed by a call to `MoveNext` then the value `False` is returned to the caller.

When control flow exits the iterator method body through an unhandled exception, then the exception is propagated to the caller, which again will be either an invocation of `MoveNext` or of `Dispose`.

As for the other possible return types of an iterator function,

* When an iterator method is invoked whose return type is `IEnumerable(Of T)` for some `T`, an instance is first created -- specific to that invocation of the iterator method -- of all parameters in the method, and they are initialized with the supplied values. The result of the invocation is an an object which implements the return type. Should this object's `GetEnumerator` method be called, it creates an instance -- specific to that invocation of `GetEnumerator` -- of all parameters and local variables in the method. It initializes the parameters to the values already saved, and proceeds as for the iterator method above.
* When an iterator method is invoked whose return type is the non-generic interface `IEnumerator`, the behavior is exactly as for `IEnumerator(Of Object)`.
* When an iterator method is invoked whose return type is the non-generic interface `IEnumerable`, the behavior is exactly as for `IEnumerable(Of Object)`.

### Async Methods

Async methods are a convenient way to do long-running work without for example blocking the UI of an application. Async is short for *Asynchronous* - it means that the caller of the async method will resume its execution promptly, but the eventual completion of the async method's instance may happen at some later time in the future. By convention async methods are named with the suffix "Async".

```vb
Async Function TestAsync() As Task(Of String)
    Console.WriteLine("hello")
    Await Task.Delay(100)
    Return "world"
End Function

Dim t = TestAsync()         ' prints "hello"
Console.WriteLine(Await t)  ' prints "world"
```

__Note.__ Async methods are *not* run on a background thread. Instead they allow a method to suspend itself through the `Await` operator, and schedule itself to be resumed in response to some event.

When an async method is invoked

1. First an instance of the async method is created specific to that invocation. This instance includes a copy of all parameters and local variables of the method.
2. Then all of its parameters are initialized to the supplied values, and all of its local variables to the default values of their types.
3. In the case of an async method with return type `Task(Of T)` for some `T`, an implicit local variable is also initialized called the *task return variable*, whose type is `T` and whose initial value is the default of `T`.
4. If the async method is a `Function` with return type `Task` or `Task(Of T)` for some `T`, then an object of that type implicitly created, associated with the current invocation. This is called an *async object* and represents the future work that will be done by executing the instance of the async method. When control resumes in the caller of this async method instance, the caller will receive this async object as the result of its invocation.
5. The instance's control point is then set at the first statement of the async method body, and immediately starts to execute the method body from there (Section [Blocks and Labels](statements.md#blocks-and-labels)).

__Resumption Delegate and Current Caller__

As detailed in Section [Await Operator](expressions.md#await-operator), execution of an `Await` expression has the ability to suspend the method instance's control point leaving control flow to go elsewhere. Control flow can later resume at the same instance's control point through invocation of a *resumption delegate*. Note that this suspension is done without exiting the async method, and does not cause finally handlers to execute. The method instance is still referenced by both the resumption delegate and the `Task` or `Task(Of T)` result (if any), and will not be garbage collected so long as there exists a live reference to either delegate or result.

It is helpful to imagine the statement `Dim x = Await WorkAsync()` approximately as syntactic shorthand for the following:

```vb
Dim temp = WorkAsync().GetAwaiter()
If Not temp.IsCompleted Then
       temp.OnCompleted(resumptionDelegate)
       Return [task]
       CONT:   ' invocation of 'resumptionDelegate' will resume here
End If
Dim x = temp.GetResult()
```

In the following, the *current caller* of the method instance is defined as either the original caller, or the caller of the resumption delegate, whichever is more recent.

When control flow exits the async method body -- through reaching the `End Sub` or `End Function` that mark its end, or through an explicit `Return` or `Exit` statement, or through an unhandled exception -- the instance's control point is set to the end of the method. Behavior then depends on the return type of the async method.

* In the case of an `Async Function` with return type `Task`:
  1. If control flow exits through an unhandled exception, then the async object's status is set to `TaskStatus.Faulted` and its `Exception.InnerException` property is set to the exception (except: certain implementation-defined exceptions such as `OperationCanceledException` change it to `TaskStatus.Canceled`). Control flow resumes in the current caller.
  2. Otherwise, the async object's status is set to `TaskStatus.Completed`. Control flow resumes in the current caller.

     (__Note.__ The whole point of Task, and what makes async methods interesting, is that when a task becomes Completed then any methods that were awaiting it will presently have their resumption delegates executed, i.e. they will become unblocked.)

* In the case of an `Async Function` with return type `Task(Of T)` for some `T`: the behavior is as above, except that in non-exception cases the async object's `Result` property is also set to the final value of the task return variable.

* In the case of an `Async Sub`:
  1. If control flow exits through an unhandled exception, then that exception is propagated to the environment in some implementation-specific manner. Control flow resumes in the current caller.
  2. Otherwise, control flow simply resumes in the current caller.

#### Async Sub

There is some Microsoft-specific behavior of an `Async Sub`.

If `SynchronizationContext.Current` is `Nothing` at the start of the invocation, then any unhandled exceptions from an Async Sub will be posted to the Threadpool.

If `SynchronizationContext.Current` is not `Nothing` at the start of the invocation, then `OperationStarted()` is invoked on that context before the start of the method and `OperationCompleted()` after the end. Additionally, any unhandled exceptions will be posted to be rethrown on the synchronization context.

This means that in UI applications, for an `Async Sub` that is invoked on the UI thread, any exceptions it fails to handle will be reposted the UI thread.

#### Mutable structures in async and iterator methods

Mutable structures in general are considered bad practice, and they are not supported by async or iterator methods. In particular, each invocation of an async or iterator method in a structure will implicitly operate on a *copy* of that structure that is copied at its moment of invocation. Thus, for example,

```vb
Structure S
       Dim x As Integer
       Async Sub Mutate()
           x = 2
       End Sub
End Structure

Dim s As New S With {.x = 1}
s.Mutate()
Console.WriteLine(s.x)   ' prints "1"
```

### Blocks and Labels

A group of executable statements is called a statement block. Execution of a statement block begins with the first statement in the block. Once a statement has been executed, the next statement in lexical order is executed, unless a statement transfers execution elsewhere or an exception occurs.

Within a statement block, the division of statements on logical lines is not significant with the exception of label declaration statements. A label is an identifier that identifies a particular position within the statement block that can be used as the target of a branch statement such as `GoTo`.

```antlr
Block
    : Statements*
    ;

LabelDeclarationStatement
    : LabelName ':'
    ;

LabelName
    : Identifier
    | IntLiteral
    ;

Statements
    : Statement? ( ':' Statement? )*
    ;
```


Label declaration statements must appear at the beginning of a logical line and labels may be either an identifier or an integer literal. Because both label declaration statements and invocation statements can consist of a single identifier, a single identifier at the beginning of a local line is always considered a label declaration statement. Label declaration statements must always be followed by a colon, even if no statements follow on the same logical line.

Labels have their own declaration space and do not interfere with other identifiers. The following example is valid and uses the name variable `x` both as a parameter and as a label.

```vb
Function F(x As Integer) As Integer
    If x >= 0 Then
        GoTo x
    End If
    x = -x
x: 
    Return x
End Function
```

The scope of a label is the body of the method containing it.

For the sake of readability, statement productions that involve multiple substatements are treated as a single production in this specification, even though the substatements may each be by themselves on a labeled line.


### Local Variables and Parameters

The preceding sections detail how and when method instances are created, and with them the copies of a method's local variables and parameters. In addition, each time the body of a loop is entered, a new copy is made of each local variable declared inside that loop as described in Section [Loop Statements](statements.md#loop-statements), and the method instance now contains this copy of its local variable rather than the previous copy.

All locals are initialized to their type's default value. Local variables and parameters are always publicly accessible. It is an error to refer to a local variable in a textual position that precedes its declaration, as the following example illustrates:

```vb
Class A
    Private i As Integer = 0

    Sub F()
        i = 1
        Dim i As Integer       ' Error, use precedes declaration.
        i = 2
    End Sub

    Sub G()
        Dim a As Integer = 1
        Dim b As Integer = a   ' This is valid.
    End Sub
End Class
```

In the `F` method above, the first assignment to `i` specifically does not refer to the field declared in the outer scope. Rather, it refers to the local variable, and it is in error because it textually precedes the declaration of the variable. In the `G` method, a subsequent variable declaration refers to a local variable declared in an earlier variable declaration within the same local variable declaration.

Each block in a method creates a declaration space for local variables. Names are introduced into this declaration space through local variable declarations in the method body and through the parameter list of the method, which introduces names into the outermost block's declaration space. Blocks do not allow shadowing of names through nesting: once a name has been declared in a block, the name may not be redeclared in any nested blocks.

Thus, in the following example, the `F` and `G` methods are in error because the name `i` is declared in the outer block and cannot be redeclared in the inner block. However, the `H` and `I` methods are valid because the two `i`'s are declared in separate non-nested blocks.

```vb
Class A
    Sub F()
        Dim i As Integer = 0
        If True Then
               Dim i As Integer = 1
        End If
    End Sub

    Sub G()
        If True Then
            Dim i As Integer = 0
        End If
        Dim i As Integer = 1
    End Sub 

    Sub H()
        If True Then
            Dim i As Integer = 0
        End If
        If True Then
            Dim i As Integer = 1
        End If
    End Sub

    Sub I() 
        For i As Integer = 0 To 9
            H()
        Next i

        For i As Integer = 0 To 9
            H()
        Next i
    End Sub 
End Class
```

When the method is a function, a special local variable is implicitly declared in the method body's declaration space with the same name as the method representing the return value of the function. The local variable has special name resolution semantics when used in expressions. If the local variable is used in a context that expects an expression classified as a method group, such as an invocation expression, then the name resolves to the function rather than to the local variable. For example:

```vb
Function F(i As Integer) As Integer
    If i = 0 Then
        F = 1        ' Sets the return value.
    Else
        F = F(i - 1) ' Recursive call.
    End If
End Function
```

The use of parentheses can cause ambiguous situations (such as `F(1)`, where `F` is a function whose return type is a one-dimensional array); in all ambiguous situations, the name resolves to the function rather than the local variable. For example:

```vb
Function F(i As Integer) As Integer()
    If i = 0 Then
        F = new Integer(2) { 1, 2, 3 }
    Else
        F = F(i - 1) ' Recursive call, not an index.
    End If
End Function
```

When control flow leaves the method body, the value of the local variable is passed back to the invocation expression. If the method is a subroutine, there is no such implicit local variable, and control simply returns to the invocation expression.

## Local Declaration Statements

A local declaration statement declares a new local variable, local constant, or static variable. *Local variables* and *local constants* are equivalent to instance variables and constants scoped to the method and are declared in the same way. *Static variables* are similar to `Shared` variables and are declared using the `Static` modifier.

```antlr
LocalDeclarationStatement
    : LocalModifier VariableDeclarators StatementTerminator
    ;

LocalModifier
    : 'Static' | 'Dim' | 'Const'
    ;
```

Static variables are locals that retain their value across invocations of the method. Static variables declared within non-shared methods are per instance: each instance of the type that contains the method has its own copy of the static variable. Static variables declared within `Shared` methods are per type; there is only one copy of the static variable for all instances. While local variables are initialized to their type's default value upon each entry into the method, static variables are only initialized to their type's default value when the type or type instance is initialized. Static variables may not be declared in structures or generic methods.

Local variables, local constants, and static variables always have public accessibility and may not specify accessibility modifiers. If no type is specified on a local declaration statement, then the following steps determine the type of the local declaration:

1. If the declaration has a type character, the type of the type character is the type of the local declaration.

2. If the local declaration is a local constant, or if the local declaration is a local variable with an initializer and local variable type inference is being used, the type of the local declaration is inferred from the type of the initializer. If the initializer refers to the local declaration, a compile-time error occurs. (Local constants are required to have initializers.)

3. If strict semantics are not being used, the type of the local declaration statement is implicitly `Object`.

4. Otherwise, a compile-time error occurs.

If no type is specified on a local declaration statement that has an array size or array type modifier, then the type of the local declaration is an array with the specified rank and the previous steps are used to determine the element type of the array. If local variable type inference is used, the type of the initializer must be an array type with the same array shape (i.e. array type modifiers) as the local declaration statement. Note that it is possible that the inferred element type may still be an array type. For example:

```vb
Option Infer On

Module Test
    Sub Main()
        ' Error: initializer is not an array type
        Dim x() = 1

        ' Type is Integer()
        Dim y() = New Integer() {}

        ' Type is Integer()()
        Dim z() = New Integer()() {}

        ' Type is Integer()()()

        Dim a()() = New Integer()()() {}

        ' Error: initializer does not have same array shape
        Dim b()() = New Integer(,)() {}
    End Sub
End Module
```

If no type is specified on a local declaration statement that has a nullable type modifier, then the type of the local declaration is the nullable version of the inferred type or the inferred type itself if it a nullable value type already.  If the inferred type is not a value type that can be made nullable, a compile-time error occurs. If both a nullable type modifier and an array size or array type modifier are placed on a local declaration statement with no type, then the nullable type modifier is considered to apply to the element type of the array and the previous steps are used to determine the element type.

Variable initializers on local declaration statements are equivalent to assignment statements placed at the textual location of the declaration. Thus, if execution branches over the local declaration statement, the variable initializer is not executed. If the local declaration statement is executed more than once, the variable initializer is executed an equal number of times. Static variables only execute their initializer the first time. If an exception occurs while initializing a static variable, the static variable is considered initialized with the default value of the static variable's type.

The following example shows the use of initializers:

```vb
Module Test
    Sub F()
        Static x As Integer = 5

        Console.WriteLine("Static variable x = " & x)
        x += 1
    End Sub

    Sub Main()
        Dim i As Integer

        For i = 1 to 3
            F()
        Next i

        i = 3
label:
        Dim y As Integer = 8

        If i > 0 Then
            Console.WriteLine("Local variable y = " & y)
            y -= 1
            i -= 1
            GoTo label
        End If
    End Sub
End Module
```

This program prints:

```
Static variable x = 5
Static variable x = 6
Static variable x = 7
Local variable y = 8
Local variable y = 8
Local variable y = 8
```

Initializers on static locals are thread-safe and protected against exceptions during initialization. If an exception occurs during a static local initializer, the static local will have its default value and not be initialized. A static local initializer

```vb
Module Test
    Sub F()
        Static x As Integer = 5
    End Sub
End Module
```

is equivalent to

```vb
Imports System.Threading
Imports Microsoft.VisualBasic.CompilerServices

Module Test
    Class InitFlag
        Public State As Short
    End Class

    Private xInitFlag As InitFlag = New InitFlag()

    Sub F()
        Dim x As Integer

        If xInitFlag.State <> 1 Then
            Monitor.Enter(xInitFlag)
            Try
                If xInitFlag.State = 0 Then
                    xInitFlag.State = 2
                    x = 5
                Else If xInitFlag.State = 2 Then
                    Throw New IncompleteInitialization()
                End If
            Finally
                xInitFlag.State = 1
                Monitor.Exit(xInitFlag)
            End Try
        End If
    End Sub
End Module
```

Local variables, local constants, and static variables are scoped to the statement block in which they are declared. Static variables are special in that their names may only be used once throughout the entire method. For example, it is not valid to specify two static variable declarations with the same name even if they are in different blocks.


### Implicit Local Declarations

In addition to local declaration statements, local variables can also be declared implicitly through use. A simple name expression that uses a name that does not resolve to something else declares a local variable by that name. For example:

```vb
Option Explicit Off

Module Test
    Sub Main()
        x = 10
        y = 20
        Console.WriteLine(x + y)
    End Sub
End Module
```

Implicit local declaration only occurs in expression contexts that can accept an expression classified as a variable. The exception to this rule is that a local variable may not be implicitly declared when it is the target of a function invocation expression, indexing expression, or a member access expression.

Implicit locals are treated as if they are declared at the beginning of the containing method. Thus, they are always scoped to the entire method body, even if declared inside of a lambda expression. For example, the following code:

```vb
Option Explicit Off 

Module Test
    Sub Main()
        Dim x = Sub()
                    a = 10
                End Sub
        Dim y = Sub()
                    Console.WriteLine(a)
                End Sub

        x()
        y()
    End Sub
End Module
```

will print the value `10`. Implicit locals are typed as `Object` if no type character was attached to the variable name; otherwise the type of the variable is the type of the type character. Local variable type inference is not used for implicit locals.

If explicit local declaration is specified by the compilation environment or by `Option Explicit`, all local variables must be explicitly declared and implicit variable declaration is disallowed.

## With Statement

A `With` statement allows multiple references to an expression's members without specifying the expression multiple times.

```antlr
WithStatement
    : 'With' Expression StatementTerminator
      Block?
      'End' 'With' StatementTerminator
    ;
```

The expression must be classified as a value and is evaluated once, upon entry into the block. Within the `With` statement block, a member access expression or dictionary access expression starting with a period or an exclamation point is evaluated as if the `With` expression preceded it. For example:

```vb
Structure Test
    Public x As Integer

    Function F() As Integer
        Return 10
    End Sub
End Structure

Module TestModule
    Sub Main()
        Dim y As Test

        With y
            .x = 10
            Console.WriteLine(.x)
            .x = .F()
        End With
    End Sub
End Module
```

It is invalid to branch into a `With` statement block from outside of the block.


## SyncLock Statement

A `SyncLock` statement allows statements to be synchronized on an expression, which ensures that multiple threads of execution do not execute the same statements at the same time.

```antlr
SyncLockStatement
    : 'SyncLock' Expression StatementTerminator
      Block?
      'End' 'SyncLock' StatementTerminator
    ;
```

The expression must be classified as a value and is evaluated once, upon entry to the block. When entering the `SyncLock` block, the `Shared` method `System.Threading.Monitor.Enter` is called on the specified expression, which blocks until the thread of execution has an exclusive lock on the object returned by the expression. The type of the expression in a `SyncLock` statement must be a reference type. For example:

```vb
Class Test
    Private count As Integer = 0

    Public Function Add() As Integer
        SyncLock Me
            count += 1
            Add = count
        End SyncLock
    End Function

    Public Function Subtract() As Integer
        SyncLock Me
            count -= 1
            Subtract = count
        End SyncLock
    End Function
End Class
```

The example above synchronizes on the specific instance of the class `Test` to ensure that no more than one thread of execution can add or subtract from the count variable at a time for a particular instance.

The `SyncLock` block is implicitly contained by a `Try` statement whose `Finally` block calls the `Shared` method `System.Threading.Monitor.Exit` on the expression. This ensures the lock is freed even when an exception is thrown. As a result, it is invalid to branch into a `SyncLock` block from outside of the block, and a `SyncLock` block is treated as a single statement for the purposes of `Resume` and `Resume Next`. The above example is equivalent to the following code:

```vb
Class Test
    Private count As Integer = 0

    Public Function Add() As Integer
        Try
            System.Threading.Monitor.Enter(Me)

            count += 1
            Add = count
        Finally
            System.Threading.Monitor.Exit(Me)
        End Try
    End Function

    Public Function Subtract() As Integer
        Try
            System.Threading.Monitor.Enter(Me)

            count -= 1
            Subtract = count
        Finally
            System.Threading.Monitor.Exit(Me)
        End Try
    End Function
End Class
```


## Event Statements

The `RaiseEvent`, `AddHandler`, and `RemoveHandler` statements raise events and handle events dynamically.

```antlr
EventStatement
    : RaiseEventStatement
    | AddHandlerStatement
    | RemoveHandlerStatement
    ;
```

### RaiseEvent Statement

A `RaiseEvent` statement notifies event handlers that a particular event has occurred.

```antlr
RaiseEventStatement
    : 'RaiseEvent' IdentifierOrKeyword
      ( OpenParenthesis ArgumentList? CloseParenthesis )? StatementTerminator
    ;
```

The simple name expression in a `RaiseEvent` statement is interpreted as a member lookup on `Me`. Thus, `RaiseEvent x` is interpreted as if it were `RaiseEvent Me.x`. The result of the expression must be classified as an event access for an event defined in the class itself; events defined on base types cannot be used in a `RaiseEvent` statement.

The `RaiseEvent` statement is processed as a call to the `Invoke` method of the event's delegate, using the supplied parameters, if any. If the delegate's value is `Nothing`, no exception is thrown. If there are no arguments, the parentheses may be omitted. For example:

```vb
Class Raiser
    Public Event E1(Count As Integer)

    Public Sub Raise()
        Static RaiseCount As Integer = 0

        RaiseCount += 1
        RaiseEvent E1(RaiseCount)
    End Sub
End Class

Module Test
    Private WithEvents x As Raiser

    Private Sub E1Handler(Count As Integer) Handles x.E1
        Console.WriteLine("Raise #" & Count)
    End Sub

    Public Sub Main()
        x = New Raiser
        x.Raise()        ' Prints "Raise #1".
        x.Raise()        ' Prints "Raise #2".
        x.Raise()        ' Prints "Raise #3".
    End Sub
End Module
```

The class `Raiser` above is equivalent to:

```vb
Class Raiser
    Public Event E1(Count As Integer)

    Public Sub Raise()
        Static RaiseCount As Integer = 0
        Dim TemporaryDelegate As E1EventHandler

        RaiseCount += 1

        ' Use a temporary to avoid a race condition.
        TemporaryDelegate = E1Event
        If Not TemporaryDelegate Is Nothing Then
            TemporaryDelegate.Invoke(RaiseCount)
        End If
    End Sub
End Class
```


### AddHandler and RemoveHandler Statements

Although most event handlers are automatically hooked up through `WithEvents` variables, it may be necessary to dynamically add and remove event handlers at run time. `AddHandler` and `RemoveHandler` statements do this.

```antlr
AddHandlerStatement
    : 'AddHandler' Expression Comma Expression StatementTerminator
    ;

RemoveHandlerStatement
    : 'RemoveHandler' Expression Comma Expression StatementTerminator
    ;
```

Each statement takes two arguments: the first argument must be an expression that is classified as an event access and the second argument must be an expression that is classified as a value. The second argument's type must be the delegate type associated with the event access. For example:

```vb
Public Class Form1
    Public Sub New()
        ' Add Button1_Click as an event handler for Button1's Click event.
        AddHandler Button1.Click, AddressOf Button1_Click
    End Sub 

    Private Button1 As Button = New Button()

    Sub Button1_Click(sender As Object, e As EventArgs)
        Console.WriteLine("Button1 was clicked!")
    End Sub

    Public Sub Disconnect()
        RemoveHandler Button1.Click, AddressOf Button1_Click
    End Sub
End Class
```

Given an event `E,` the statement calls the relevant `add_E` or `remove_E` method on the instance to add or remove the delegate as a handler for the event. Thus, the above code is equivalent to:

```vb
Public Class Form1
    Public Sub New()
        Button1.add_Click(AddressOf Button1_Click)
    End Sub 

    Private Button1 As Button = New Button()

    Sub Button1_Click(sender As Object, e As EventArgs)
        Console.WriteLine("Button1 was clicked!")
    End Sub

    Public Sub Disconnect()
        Button1.remove_Click(AddressOf Button1_Click)
    End Sub
End Class
```


## Assignment Statements

An assignment statement assigns the value of an expression to a variable. There are several types of assignment.

```antlr
AssignmentStatement
    : RegularAssignmentStatement
    | CompoundAssignmentStatement
    | MidAssignmentStatement
    ;
```

### Regular Assignment Statements

A simple assignment statement stores the result of an expression in a variable.

```antlr
RegularAssignmentStatement
    : Expression Equals Expression StatementTerminator
    ;
```

The expression on the left side of the assignment operator must be classified as a variable or a property access, while the expression on the right side of the assignment operator must be classified as a value. The type of the expression must be implicitly convertible to the type of the variable or property access.

If the variable being assigned into is an array element of a reference type, a run-time check will be performed to ensure that the expression is compatible with the array-element type. In the following example, the last assignment causes a `System.ArrayTypeMismatchException` to be thrown, because an instance of `ArrayList` cannot be stored in an element of a `String` array.

```vb
Dim sa(10) As String
Dim oa As Object() = sa
oa(0) = Nothing         ' This is allowed.
oa(1) = "Hello"         ' This is allowed.
oa(2) = New ArrayList() ' System.ArrayTypeMismatchException is thrown.
```

If the expression on the left side of the assignment operator is classified as a variable, then the assignment statement stores the value in the variable. If the expression is classified as a property access, then the assignment statement turns the property access into an invocation of the `Set` accessor of the property with the value substituted for the value parameter. For example:

```vb
Module Test
    Private PValue As Integer

    Public Property P As Integer
        Get
            Return PValue
        End Get

        Set (Value As Integer)
            PValue = Value
        End Set
    End Property

    Sub Main()
        ' The following two lines are equivalent.
        P = 10
        set_P(10)
    End Sub
End Module
```

If the target of the variable or property access is typed as a value type but not classified as a variable, a compile-time error occurs. For example:

```vb
Structure S
    Public F As Integer
End Structure

Class C
    Private PValue As S

    Public Property P As S
        Get
            Return PValue
        End Get

        Set (Value As S)
            PValue = Value
        End Set
    End Property
End Class

Module Test
    Sub Main()
        Dim ct As C = New C()
        Dim rt As Object = new C()

        ' Compile-time error: ct.P not classified as variable.
        ct.P.F = 10

        ' Run-time exception.
        rt.P.F = 10
    End Sub
End Module
```

Note that the semantics of the assignment depend on the type of the variable or property to which it is being assigned. If the variable to which it is being assigned is a value type, the assignment copies the value of the expression into the variable. If the variable to which it is being assigned is a reference type, the assignment copies the reference, not the value itself, into the variable. If the type of the variable is `Object`, the assignment semantics are determined by whether the value's type is a value type or a reference type at run time.


__Note.__ For intrinsic types such as `Integer` and `Date`, reference and value assignment semantics are the same because the types are immutable. As a result, the language is free to use reference assignment on boxed intrinsic types as an optimization. From a value perspective, the result is the same.

Because the equals character (`=`) is used both for assignment and for equality, there is an ambiguity between a simple assignment and an invocation statement in situations such as `x = y.ToString()`. In all such cases, the assignment statement takes precedence over the equality operator. This means that the example expression is interpreted as `x = (y.ToString())` rather than `(x = y).ToString()`.


### Compound Assignment Statements

A *compound assignment statement* takes the form `V op= E` (where `op` is a valid binary operator).

```antlr
CompoundAssignmentStatement
    : Expression CompoundBinaryOperator LineTerminator? Expression StatementTerminator
    ;

CompoundBinaryOperator
    : '^' '=' | '*' '=' | '/' '=' | '\\' '=' | '+' '=' | '-' '='
    | '&' '=' | '<' '<' '=' | '>' '>' '='
    ;
```

The expression on the left side of the assignment operator must be classified as a variable or property access, while the expression on the right side of the assignment operator must be classified as a value. The compound assignment statement is equivalent to the statement `V = V op E` with the difference that the variable on the left side of the compound assignment operator is only evaluated once. The following example demonstrates this difference:

```vb
Module Test
    Function GetIndex() As Integer
        Console.WriteLine("Getting index")
        Return 1
    End Function

    Sub Main()
        Dim a(2) As Integer

        Console.WriteLine("Simple assignment")
        a(GetIndex()) = a(GetIndex()) + 1

        Console.WriteLine("Compound assignment")
        a(GetIndex()) += 1
    End Sub
End Module
```

The expression `a(GetIndex())` is evaluated twice for simple assignment but only once for compound assignment, so the code prints:

```
Simple assignment
Getting index
Getting index
Compound assignment
Getting index
```


### Mid Assignment Statement

A `Mid` assignment statement assigns a string into another string. The left side of the assignment has the same syntax as a call to the function `Microsoft.VisualBasic.Strings.Mid`.

```antlr
MidAssignmentStatement
    : 'Mid' '$'? OpenParenthesis Expression Comma Expression
      ( Comma Expression )? CloseParenthesis Equals Expression StatementTerminator
    ;
```

The first argument is the target of the assignment and must be classified as a variable or a property access whose type is implicitly convertible to and from `String`. The second parameter is the 1-based start position that corresponds to where the assignment should begin in the target string and must be classified as a value whose type must be implicitly convertible to `Integer`. The optional third parameter is the number of characters from the right-side value to assign into the target string and must be classified as a value whose type is implicitly convertible to `Integer`. The right side is the source string and must be classified as a value whose type is implicitly convertible to `String`. The right side is truncated to the length parameter, if specified, and replaces the characters in the left-side string, starting at the start position. If the right side string contained fewer characters than the third parameter, only the characters from the right side string will be copied.

The following example displays `ab123fg`:

```vb
Module Test
    Sub Main()
        Dim s1 As String = "abcdefg"
        Dim s2 As String = "1234567"

        Mid$(s1, 3, 3) = s2
        Console.WriteLine(s1)
    End Sub
End Module
```

__Note.__ `Mid` is not a reserved word.


## Invocation Statements

An invocation statement invokes a method preceded by the optional keyword `Call`. The invocation statement is processed in the same way as the function invocation expression, with some differences noted below. The invocation expression must be classified as a value or void. Any value resulting from the evaluation of the invocation expression is discarded.

If the `Call` keyword is omitted, then the invocation expression must start with an identifier or keyword, or with `.` inside a `With` block. Thus, for instance, "`Call 1.ToString()`" is a valid statement but "`1.ToString()`" is not. (Note that in an expression context, invocation expressions also need not start with an identifier. For example, "`Dim x = 1.ToString()`" is a valid statement).

There is another difference between the invocation statements and invocation expressions: if an invocation statement includes an argument list, then this is always taken as the argument list of the invocation. The following example illustrates the difference:

```vb
Module Test
    Sub Main()
        Call {Function() 15}(0)
        ' error: (0) is taken as argument list, but array is not invokable

        Call ({Function() 15}(0))
        ' valid, since the invocation statement has no argument list

        Dim x = {Function() 15}(0)
        ' valid as an expression, since (0) is taken as an array-indexing

        Call f("a")
        ' error: ("a") is taken as argument list to the invocation of f

        Call f()("a")
        ' valid, since () is the argument list for the invocation of f

        Dim y = f("a")
        ' valid as an expression, since f("a") is interpreted as f()("a")
    End Sub

    Sub f() As Func(Of String,String)
        Return Function(x) x
    End Sub
End Module
```

```antlr
InvocationStatement
    : 'Call'? InvocationExpression StatementTerminator
    ;
```

## Conditional Statements

Conditional statements allow conditional execution of statements based on expressions evaluated at run time.

```antlr
ConditionalStatement
    : IfStatement
    | SelectStatement
    ;
```

### If...Then...Else Statements

An `If...Then...Else` statement is the basic conditional statement.

```antlr
IfStatement
    : BlockIfStatement
    | LineIfThenStatement
    ;

BlockIfStatement
    : 'If' BooleanExpression 'Then'? StatementTerminator
      Block?
      ElseIfStatement*
      ElseStatement?
      'End' 'If' StatementTerminator
    ;

ElseIfStatement
    : ElseIf BooleanExpression 'Then'? StatementTerminator
      Block?
    ;

ElseStatement
    : 'Else' StatementTerminator
      Block?
    ;

LineIfThenStatement
    : 'If' BooleanExpression 'Then' Statements ( 'Else' Statements )? StatementTerminator
    ;
```

Each expression in an `If...Then...Else` statement must be a Boolean expression, as per Section [Boolean Expressions](expressions.md#boolean-expressions). (Note: this does not require the expression to have Boolean type). If the expression in the `If` statement is true, the statements enclosed by the `If` block are executed. If the expression is false, each of the `ElseIf` expressions is evaluated. If one of the `ElseIf` expressions evaluates to true, the corresponding block is executed. If no expression evaluates to true and there is an `Else` block, the `Else` block is executed. Once a block finishes executing, execution passes to the end of the `If...Then...Else` statement.

The line version of the `If` statement has a single set of statements to be executed if the `If` expression is `True` and an optional set of statements to be executed if the expression is `False`. For example:

```vb
Module Test
    Sub Main()
        Dim a As Integer = 10
        Dim b As Integer = 20

        ' Block If statement.
        If a < b Then
            a = b
        Else
            b = a
        End If

        ' Line If statement
        If a < b Then a = b Else b = a
    End Sub
End Module
```

The line version of the If statement binds less tightly than ":", and its `Else` binds to the lexically nearest preceding `If` that is allowed by the syntax. For example, the following two versions are equivalent:

```vb
If True Then _
If True Then Console.WriteLine("a") Else Console.WriteLine("b") _
Else Console.WriteLine("c") : Console.WriteLine("d")

If True Then
    If True Then
        Console.WriteLine("a")
    Else
        Console.WriteLine("b")
    End If
    Console.WriteLine("c") : Console.WriteLine("d")
End If
```

All statements other than label declaration statements are allowed inside a line `If` statement, including block statements. However, they may not use LineTerminators as StatementTerminators except inside multi-line lambda expressions. For example:

```vb
' Allowed, since it uses : instead of LineTerminator to separate statements
If b Then With New String("a"(0),5) : Console.WriteLine(.Length) : End With

' Disallowed, since it uses a LineTerminator
If b then With New String("a"(0), 5)
              Console.WriteLine(.Length)
          End With

' Allowed, since it only uses LineTerminator inside a multi-line lambda
If b Then Call Sub()
                   Console.WriteLine("a")
               End Sub.Invoke()
```

### Select Case Statements

A `Select Case` statement executes statements based on the value of an expression.

```antlr
SelectStatement
    : 'Select' 'Case'? Expression StatementTerminator
      CaseStatement*
      CaseElseStatement?
      'End' 'Select' StatementTerminator
    ;

CaseStatement
    : 'Case' CaseClauses StatementTerminator
      Block?
    ;

CaseClauses
    : CaseClause ( Comma CaseClause )*
    ;

CaseClause
    : ( 'Is' LineTerminator? )? ComparisonOperator LineTerminator? Expression
    | Expression ( 'To' Expression )?
    ;

ComparisonOperator
    : '=' | '<' '>' | '<' | '>' | '>' '=' | '<' '='
    ;

CaseElseStatement
    : 'Case' 'Else' StatementTerminator
      Block?
    ;
```

The expression must be classified as a value. When a `Select Case` statement is executed, the `Select` expression is evaluated first, and the `Case` statements are then evaluated in order of textual declaration. The first `Case` statement that evaluates to `True` has its block executed. If no `Case` statement evaluates to `True` and there is a `Case Else` statement, that block is executed. Once a block has finished executing, execution passes to the end of the `Select` statement.

Execution of a `Case` block is not permitted to "fall through" to the next switch section. This prevents a common class of bugs that occur in other languages when a `Case` terminating statement is accidentally omitted. The following example illustrates this behavior:

```vb
Module Test
    Sub Main()
        Dim x As Integer = 10

        Select Case x
            Case 5
                Console.WriteLine("x = 5")
            Case 10
                Console.WriteLine("x = 10")
            Case 20 - 10
                Console.WriteLine("x = 20 - 10")
            Case 30
                Console.WriteLine("x = 30")
        End Select
    End Sub
End Module
```

The code prints:

```
x = 10
```

Although `Case 10` and `Case 20 - 10` select for the same value, `Case 10` is executed because it precedes `Case 20 - 10` textually. When the next `Case` is reached, execution continues after the `Select` statement.

A `Case` clause may take two forms. One form is an optional `Is` keyword, a comparison operator, and an expression. The expression is converted to the type of the `Select` expression; if the expression is not implicitly convertible to the type of the `Select` expression, a compile-time error occurs. If the `Select` expression is *E*, the comparison operator is *Op*, and the `Case` expression is *E1*, the case is evaluated as *E OP E1*. The operator must be valid for the types of the two expressions; otherwise a compile-time error occurs.

The other form is an expression optionally followed by the keyword `To` and a second expression. Both expressions are converted to the type of the `Select` expression; if either expression is not implicitly convertible to the type of the `Select` expression, a compile-time error occurs. If the `Select` expression is `E`, the first `Case` expression is `E1`, and the second `Case` expression is `E2`, the `Case` is evaluated either as `E = E1` (if no `E2` is specified) or `(E >= E1) And (E <= E2)`. The operators must be valid for the types of the two expressions; otherwise a compile-time error occurs.


## Loop Statements

Loop statements allow repeated execution of the statements in their body.

```antlr
LoopStatement
    : WhileStatement
    | DoLoopStatement
    | ForStatement
    | ForEachStatement
    ;
```

Each time a loop body is entered, a fresh copy is made of all local variables declared in that body, initialized to the previous values of the variables. Any reference to a variable within the loop body will use the most recently made copy. This code shows an example:

```vb
Module Test
    Sub Main()
        Dim lambdas As New List(Of Action)
        Dim x = 1

        For i = 1 To 3
            x = i
            Dim y = x
            lambdas.Add(Sub() Console.WriteLine(x & y))
        Next

        For Each lambda In lambdas
            lambda()
        Next
    End Sub
End Module
```

The code produces the output:

```
31    32    33
```

When the loop body is executed, it uses whichever copy of the variable is current. For example, the statement  `Dim y = x` refers to the latest copy of `y` and the original copy of `x`. And when a lambda is created, it remembers whichever copy of a variable was current at the time it was created. Therefore, each lambda uses the same shared copy of `x`, but a different copy of `y`. At the end of the program, when it executes the lambdas, that shared copy of `x` that they all refer to is now at its final value 3.

Note that if there are no lambdas or LINQ expressions, then it's impossible to know that a fresh copy is made on loop entry. Indeed, compiler optimizations will avoid making copies in this case. Note too that it's illegal to `GoTo` into a loop that contains lambdas or LINQ expressions.


### While...End While and Do...Loop Statements

A `While` or `Do` loop statement loops based on a Boolean expression.

```antlr
WhileStatement
    : 'While' BooleanExpression StatementTerminator
      Block?
      'End' 'While' StatementTerminator
    ;

DoLoopStatement
    : DoTopLoopStatement
    | DoBottomLoopStatement
    ;

DoTopLoopStatement
    : 'Do' ( WhileOrUntil BooleanExpression )? StatementTerminator
      Block?
      'Loop' StatementTerminator
    ;

DoBottomLoopStatement
    : 'Do' StatementTerminator
      Block?
      'Loop' WhileOrUntil BooleanExpression StatementTerminator
    ;

WhileOrUntil
    : 'While' | 'Until'
    ;
```

A `While` loop statement loops as long as the Boolean expression evaluates to true; a `Do` loop statement may contain a more complex condition. An expression may be placed after the `Do` keyword or after the `Loop` keyword, but not after both. The Boolean expression is evaluated as per Section [Boolean Expressions](expressions.md#boolean-expressions). (Note: this does not require the expression to have Boolean type). It is also valid to specify no expression at all; in that case, the loop will never exit. If the expression is placed after `Do`, it will be evaluated before the loop block is executed on each iteration. If the expression is placed after `Loop`, it will be evaluated after the loop block has executed on each iteration. Placing the expression after `Loop` will therefore generate one more loop than placement after `Do`. The following example demonstrates this behavior:

```vb
Module Test
    Sub Main()
        Dim x As Integer

        x = 3
        Do While x = 1
            Console.WriteLine("First loop")
        Loop

        Do
            Console.WriteLine("Second loop")
        Loop While x = 1
    End Sub
End Module
```

The code produces the output:

```
Second Loop
```

In the case of the first loop, the condition is evaluated before the loop executes. In the case of the second loop, the condition is executed after the loop executes. The conditional expression must be prefixed with either a `While` keyword or an `Until` keyword. The former breaks the loop if the condition evaluates to false, the latter when the condition evaluates to true.

__Note.__ `Until` is not a reserved word.


### For...Next Statements

A `For...Next` statement loops based on a set of bounds. A `For` statement specifies a loop control variable, a lower bound expression, an upper bound expression, and an optional step value expression. The loop control variable is specified either through an identifier followed by an optional `As` clause or an expression.

```antlr
ForStatement
    : 'For' LoopControlVariable Equals Expression 'To' Expression
      ( 'Step' Expression )? StatementTerminator
      Block?
      ( 'Next' NextExpressionList? StatementTerminator )?
    ;

LoopControlVariable
    : Identifier ( IdentifierModifiers 'As' TypeName )?
    | Expression
    ;

NextExpressionList
    : Expression ( Comma Expression )*
    ;
```

As per the following rules, the loop control variable refers either to a new local variable specific to this `For...Next` statement, or to a pre-existing variable, or to an expression.

* If the loop control variable is an identifier with an `As` clause, the identifier defines a new local variable of the type specified in the `As` clause, scoped to the entire `For` loop.

* If the loop control variable is an identifier without an `As` clause, then the identifier is first resolved using the simple name resolution rules (see Section [Simple Name Expressions](expressions.md#simple-name-expressions)), excepting that this occurrence of the identifier would not in and of itself cause an implicit local variable to be created (Section [Implicit Local Declarations](statements.md#implicit-local-declarations)).

  * If this resolution succeeds and the result is classified as a variable, then the loop control variable is that pre-existing variable.

  * If resolution fails, or if resolution succeeds and the result is classified as a type, then:
    * if local variable type inference is being used, the identifier defines a new local variable whose type is inferred from the bound and step expressions, scoped to the entire `For` loop;
    * if local variable type inference is not being used but implicit local declaration is, then an implicit local variable is created whose scope is the entire method (Section [Implicit Local Declarations](statements.md#implicit-local-declarations)), and the loop control variable refers to this pre-existing variable;
    * if neither local variable type inference nor implicit local declarations are used, it is an error.

  * If resolution succeeds with something classified as neither a type nor a variable, it is an error.

* If the loop control variable is an expression, the expression must be classified as a variable.

A loop control variable cannot be used by another enclosing `For...Next` statement. The type of the loop control variable of a `For` statement determines the type of the iteration and must be one of:

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single`, `Double`
* An enumerated type
* `Object`
* A type `T` that has the following operators, where `B` is a type that can be used in a Boolean expression:

```vb
Public Shared Operator >= (op1 As T, op2 As T) As B
Public Shared Operator <= (op1 As T, op2 As T) As B
Public Shared Operator - (op1 As T, op2 As T) As T
Public Shared Operator + (op1 As T, op2 As T) As T
```

The bound and step expressions must be implicitly convertible to the type of the loop control variable and must be classified as values. At compile time, the type of the loop control variable is inferred by choosing the widest type among the lower bound, upper bound, and step expression types. If there is no widening conversion between two of the types, then a compile-time error occurs.

At run time, if the type of the loop control variable is `Object`, then the type of the iteration is inferred the same as at compile time, with two exceptions. First, if the bound and step expressions are all of integral types but have no widest type, then the widest type that encompasses all three types will be inferred. And second, if the type of the loop control variable is inferred to be `String`, `Double` will be inferred instead. If, at run time, no loop control type can be determined or if any of the expressions cannot be converted to the loop control type, a `System.InvalidCastException` will occur. Once a loop control type has been chosen at the beginning of the loop, the same type will be used throughout the iteration, regardless of changes made to the value in the loop control variable.

A `For` statement must be closed by a matching `Next` statement. A `Next` statement without a variable matches the innermost open `For` statement, while a `Next` statement with one or more loop control variables will, from left to right, match the `For` loops that match each variable. If a variable matches a `For` loop that is not the most nested loop at that point, a compile-time error results.

At the beginning of the loop, the three expressions are evaluated in textual order and the lower bound expression is assigned to the loop control variable. If the step value is omitted, it is implicitly the literal `1`, converted to the type of the loop control variable. The three expressions are only ever evaluated at the beginning of the loop.

At the beginning of each loop, the control variable is compared to see if it is greater than the end point if the step expression is positive, or less than the end point if the step expression is negative. If it is, the `For` loop terminates; otherwise the loop block executes. If the loop control variable is not a primitive type, the comparison operator is determined by whether the expression `step >= step - step` is true or false. At the `Next` statement, the step value is added to the control variable and execution returns to the top of the loop.

Note that a new copy of the loop control variable is *not* created on each iteration of the loop block. In this respect, the `For` statement differs from `For Each` (Section [For Each...Next Statements](statements.md#for-eachnext-statements)).

It is not valid to branch into a `For` loop from outside the loop.


### For Each...Next Statements

A `For Each...Next` statement loops based on the elements in an expression. A `For Each` statement specifies a loop control variable and an enumerator expression. The loop control variable is specified either through an identifier followed by an optional `As` clause or an expression.

```antlr
ForEachStatement
    : 'For' 'Each' LoopControlVariable 'In' LineTerminator? Expression StatementTerminator
      Block?
      ( 'Next' NextExpressionList? StatementTerminator )?
    ;
```

Following the same rules as `For...Next` statements (Section [For...Next Statements](statements.md#fornext-statements)), the loop control variable refers either to a new local variable specific to this For Each...Next statement, or to a pre-existing variable, or to an expression.

The enumerator expression must be classified as a value and its type must be a collection type or `Object`. If the type of the enumerator expression is `Object`, then all processing is deferred until run-time. Otherwise, a conversion must exist from the element type of the collection to the type of the loop control variable.

The loop control variable cannot be used by another enclosing `For Each` statement. A `For Each` statement must be closed by a matching `Next` statement. A `Next` statement without a loop control variable matches the innermost open `For Each`. A `Next` statement with one or more loop control variables will, from left to right, match the `For Each` loops that have the same loop control variable. If a variable matches a `For Each` loop that is not the most nested loop at that point, a compile-time error occurs.

A type `C` is said to be a *collection type* if one of:

* All of the following are true:
  * `C` contains an accessible instance, shared or extension method with the signature `GetEnumerator()` that returns a type `E`.
  * `E` contains an accessible instance, shared or extension method with the signature `MoveNext()` and the return type `Boolean`.
  * `E` contains an accessible instance or shared property named `Current` that has a getter. The type of this property is the element type of the collection type.

* It implements the interface `System.Collections.Generic.IEnumerable(Of T)`, in which case the element type of the collection is considered to be `T`.

* It implements the interface `System.Collections.IEnumerable`, in which case the element type of the collection is considered to be `Object`.

Following is an example of a class that can be enumerated:

```vb
Public Class IntegerCollection
    Private integers(10) As Integer

    Public Class IntegerCollectionEnumerator
        Private collection As IntegerCollection
        Private index As Integer = -1

        Friend Sub New(c As IntegerCollection)
            collection = c
        End Sub

        Public Function MoveNext() As Boolean
            index += 1

            Return index <= 10
        End Function

        Public ReadOnly Property Current As Integer
            Get
                If index < 0 OrElse index > 10 Then
                    Throw New System.InvalidOperationException()
                End If

                Return collection.integers(index)
            End Get
        End Property
    End Class

    Public Sub New()
        Dim i As Integer

        For i = 0 To 10
            integers(i) = I
        Next i
    End Sub

    Public Function GetEnumerator() As IntegerCollectionEnumerator
        Return New IntegerCollectionEnumerator(Me)
    End Function
End Class
```

Before the loop begins, the enumerator expression is evaluated. If the type of the expression does not satisfy the design pattern, then the expression is cast to `System.Collections.IEnumerable` or `System.Collections.Generic.IEnumerable(Of T)`. If the expression type implements the generic interface, the generic interface is preferred at compile-time but the non-generic interface is preferred at run-time. If the expression type implements the generic interface multiple times, the statement is considered ambiguous and a compile-time error occurs.

__Note.__ The non-generic interface is preferred in the late bound case, because picking the generic interface would mean that all the calls to the interface methods would involve type parameters. Since it is not possible to know the matching type arguments at run-time, all such calls would have to be made using late-bound calls. This would be slower than calling the non-generic interface because the non-generic interface could be called using compile-time calls.

`GetEnumerator` is called on the resulting value and the return value of the function is stored in a temporary. Then at the beginning of each iteration, `MoveNext` is called on the temporary. If it returns `False`, the loop terminates. Otherwise, each iteration of the loop is executed as follows:

1. If the loop control variable identified a new local variable (rather than a pre-existing one), then a fresh copy of this local variable is created. For the current iteration, all references within the loop block will refer to this copy.
2. The `Current` property is retrieved, coerced to the type of the loop control variable (regardless of whether the conversion is implicit or explicit), and assigned to the loop control variable.
3. The loop block executes.

__Note.__ There is a slight change in behavior between version 10.0 and 11.0 of the language. Prior to 11.0, a fresh iteration variable was *not* created for each iteration of the loop. This difference is observable only if the iteration variable is captured by a lambda or a LINQ expression which is then invoked after the loop:

```vb
Dim lambdas As New List(Of Action)
For Each x In {1,2,3}
   lambdas.Add(Sub() Console.WriteLine(x)
Next
lambdas(0).Invoke()
lambdas(1).Invoke()
lambdas(2).Invoke()
```

Up to Visual Basic 10.0, this produced a warning at compile-time and printed "3" three times. That was because there was only a single variable "x" shared by all iterations of the loop, and all three lambdas captured the same "x", and by the time the lambdas were executed it then held the number 3. As of Visual Basic 11.0, it prints "1, 2, 3". That is because each lambda captures a different variable "x".


__Note.__ The current element of the iteration is converted to the type of the loop control variable even if the conversion is explicit because there is no convenient place to introduce a conversion operator in the statement. This became particularly troublesome when working with the now-obsolete type `System.Collections.ArrayList`, because its element type is `Object`. This would have required casts in a great many loops, something we felt was not ideal. Ironically, generics enabled the creation of a strongly-typed collection, `System.Collections.Generic.List(Of T)`, which might have made us rethink this design point, but for compatibility's sake, this cannot be changed now.


When the `Next` statement is reached, execution returns to the top of the loop. If a variable is specified after the `Next` keyword, it must be the same as the first variable after the `For Each`. For example, consider the following code:

```vb
Module Test
    Sub Main()
        Dim i As Integer
        Dim c As IntegerCollection = New IntegerCollection()

        For Each i In c
            Console.WriteLine(i)
        Next i
    End Sub
End Module
```

It is equivalent to the following code:

```vb
Module Test
    Sub Main()
        Dim i As Integer
        Dim c As IntegerCollection = New IntegerCollection()

        Dim e As IntegerCollection.IntegerCollectionEnumerator

        e = c.GetEnumerator()
        While e.MoveNext()
            i = e.Current

            Console.WriteLine(i)
        End While
    End Sub
End Module
```

If the type `E` of the enumerator implements `System.IDisposable`, then the enumerator is disposed upon exiting the loop by calling the `Dispose` method. This ensures that resources held by the enumerator are released. If the method containing the `For Each` statement does not use unstructured error handling, then the `For Each` statement is wrapped in a `Try` statement with the `Dispose` method called in the `Finally` to ensure cleanup.

__Note.__ The `System.Array` type is a collection type, and since all array types derive from `System.Array`, any array type expression is permitted in a `For Each` statement. For single-dimensional arrays, the `For Each` statement enumerates the array elements in increasing index order, starting with index 0 and ending with index Length - 1. For multidimensional arrays, the indices of the rightmost dimension are increased first.

For example, the following code prints `1 2 3 4`:

```vb
Module Test
    Sub Main()
        Dim x(,) As Integer = { { 1, 2 }, { 3, 4 } }
        Dim i As Integer

        For Each i In x
            Console.Write(i & " ")
        Next i
    End Sub
End Module
```

It is not valid to branch into a `For Each` statement block from outside the block.


## Exception-Handling Statements

Visual Basic supports structured exception handling and unstructured exception handling. Only one style of exception handling may be used in a method, but the `Error` statement may be used in structured exception handling. If a method uses both styles of exception handling, a compile-time error results.

```antlr
ErrorHandlingStatement
    : StructuredErrorStatement
    | UnstructuredErrorStatement
    ;
```

### Structured Exception-Handling Statements

Structured exception handling is a method of handling errors by declaring explicit blocks within which certain exceptions will be handled. Structured exception handling is done through a `Try` statement.

```antlr
StructuredErrorStatement
    : ThrowStatement
    | TryStatement
    ;

TryStatement
    : 'Try' StatementTerminator
      Block?
      CatchStatement*
      FinallyStatement?
      'End' 'Try' StatementTerminator
    ;
```


For example:

```vb
Module Test
    Sub ThrowException()
        Throw New Exception()
    End Sub

    Sub Main()
        Try
            ThrowException()
        Catch e As Exception
            Console.WriteLine("Caught exception!")
        Finally
            Console.WriteLine("Exiting try.")
        End Try
    End Sub
End Module
```

A `Try` statement is made up of three kinds of blocks: try blocks, catch blocks, and finally blocks. A *try block* is a statement block that contains the statements to be executed. A *catch block* is a statement block that handles an exception. A *finally block* is a statement block that contains statements to be run when the `Try` statement is exited, regardless of whether an exception has occurred and been handled. A `Try` statement, which can only contain one try block and one finally block, must contain at least one catch block or finally block. It is invalid to explicitly transfer execution into a try block except from within a catch block in the same statement.


#### Finally Blocks

A `Finally` block is always executed when execution leaves any part of the `Try` statement. No explicit action is required to execute the `Finally` block; when execution leaves the `Try` statement, the system will automatically execute the `Finally` block and then transfer execution to its intended destination. The `Finally` block is executed regardless of how execution leaves the `Try` statement: through the end of the `Try` block, through the end of a `Catch` block, through an `Exit Try` statement, through a `GoTo` statement, or by not handling a thrown exception.

Note that the `Await` expression in an async method, and the `Yield` statement in an iterator method, can cause flow of control to suspend in the async or iterator method instance and resume in some other method instance. However, this is merely a suspension of execution and does not involve exiting the respective async method or iterator method instance, and so does not cause `Finally` blocks to be executed.

It is invalid to explicitly transfer execution into a `Finally` block; it is also invalid to transfer execution out of a `Finally` block except through an exception.

```antlr
FinallyStatement
    : 'Finally' StatementTerminator
      Block?
    ;
```

#### Catch Blocks

If an exception occurs while processing the `Try` block, each `Catch` statement is examined in textual order to determine if it handles the exception.

```antlr
CatchStatement
    : 'Catch' ( Identifier ( 'As' NonArrayTypeName )? )?
	  ( 'When' BooleanExpression )? StatementTerminator
      Block?
    ;
```

The identifier specified in a `Catch` clause represents the exception that has been thrown. If the identifier contains an `As` clause, then the identifier is considered to be declared within the `Catch` block's local declaration space. Otherwise, the identifier must be a local variable (not a static variable) that was defined in a containing block.

A `Catch` clause with no identifier will catch all exceptions derived from `System.Exception`. A `Catch` clause with an identifier will only catch exceptions whose types are the same as or derived from the type of the identifier. The type must be `System.Exception`, or a type derived from `System.Exception`. When an exception is caught that derives from `System.Exception`, a reference to the exception object is stored in the object returned by the function `Microsoft.VisualBasic.Information.Err`.

A `Catch` clause with a `When` clause will only catch exceptions when the expression evaluates to `True`; the type of the expression must be a Boolean expression as per Section [Boolean Expressions](expressions.md#boolean-expressions). A `When` clause is only applied after checking the type of the exception, and the expression may refer to the identifier representing the exception, as this example demonstrates:

```vb
Module Test
    Sub Main()
        Dim i As Integer = 5

        Try
            Throw New ArgumentException()
        Catch e As OverflowException When i = 5
            Console.WriteLine("First handler")
        Catch e As ArgumentException When i = 4
            Console.WriteLine("Second handler")
        Catch When i = 5
            Console.WriteLine("Third handler")
        End Try

    End Sub
End Module
```

This example prints:

```
Third handler
```

If a `Catch` clause handles the exception, execution transfers to the `Catch` block. At the end of the `Catch` block, execution transfers to the first statement following the `Try` statement. The `Try` statement will not handle any exceptions thrown in a `Catch` block. If no `Catch` clause handles the exception, execution transfers to a location determined by the system.

It is invalid to explicitly transfer execution into a `Catch` block.

The filters in When clauses are normally evaluated prior to the exception being thrown. For instance, the following code will print "Filter, Finally, Catch".

```vb
Sub Main()
   Try
       Foo()
   Catch ex As Exception When F()
       Console.WriteLine("Catch")
   End Try
End Sub

Sub Foo()
    Try
        Throw New Exception
    Finally
        Console.WriteLine("Finally")
    End Try
End Sub

Function F() As Boolean
    Console.WriteLine("Filter")
    Return True
End Function
```

 However, Async and Iterator methods cause all finally blocks inside them to be executed prior to any filters outside. For instance, if the above code had `Async Sub Foo()`, then the output would be "Finally, Filter, Catch".


#### Throw Statement

The `Throw` statement raises an exception, which is represented by an instance of a type derived from `System.Exception`.

```antlr
ThrowStatement
    : 'Throw' Expression? StatementTerminator
    ;
```

If the expression is not classified as a value or is not a type derived from `System.Exception`, then a compile-time error occurs. If the expression evaluates to a null value at run time, then a `System.NullReferenceException` exception is raised instead.

A `Throw` statement may omit the expression within a catch block of a `Try` statement, as long as there is no intervening finally block. In that case, the statement rethrows the exception currently being handled within the catch block. For example:

```vb
Sub Test(x As Integer)
    Try
        Throw New Exception()
    Catch
        If x = 0 Then
            Throw    ' OK, rethrows exception from above.
        Else
            Try
                If x = 1 Then
                    Throw   ' OK, rethrows exception from above.
                End If
            Finally
                Throw    ' Invalid, inside of a Finally.
            End Try
        End If
    End Try
End Sub
```


### Unstructured Exception-Handling Statements

Unstructured exception handling is a method of handling errors by indicating statements to branch to when an exception occurs. Unstructured exception handling is implemented using three statements: the `Error` statement, the `On Error` statement, and the `Resume` statement.

```antlr
UnstructuredErrorStatement
    : ErrorStatement
    | OnErrorStatement
    | ResumeStatement
    ;
```

For example:

```vb
Module Test
    Sub ThrowException()
        Error 5
    End Sub

    Sub Main()
        On Error GoTo GotException

        ThrowException()
        Exit Sub

GotException:
        Console.WriteLine("Caught exception!")
        Resume Next
    End Sub
End Module
```

When a method uses unstructured exception handling, a single structured exception handler is established for the entire method that catches all exceptions. (Note that in constructors this handler does not extend over the call to the call to `New` at the beginning of the constructor.) The method then keeps track of the most recent exception-handler location and the most recent exception that has been thrown. At entry to the method, the exception-handler location and the exception are both set to `Nothing`. When an exception is thrown in a method that uses unstructured exception handling, a reference to the exception object is stored in the object returned by the function `Microsoft.VisualBasic.Information.Err`.

Unstructured error handling statements are not allowed in iterator or async methods.


#### Error Statement

An `Error` statement throws a `System.Exception` exception containing a Visual Basic 6 exception number. The expression must be classified as a value and its type must be implicitly convertible to `Integer`.

```antlr
ErrorStatement
    : 'Error' Expression StatementTerminator
    ;
```

#### On Error Statement

An `On Error` statement modifies the most recent exception-handling state.

```antlr
OnErrorStatement
    : 'On' 'Error' ErrorClause StatementTerminator
    ;

ErrorClause
    : 'GoTo' '-' '1'
    | 'GoTo' '0'
    | GoToStatement
    | 'Resume' 'Next'
    ;
```

It may be used in one of four ways:

* `On Error GoTo -1` resets the most recent exception to `Nothing`.

* `On Error GoTo 0` resets the most recent exception-handler location to `Nothing`.

* `On Error GoTo LabelName` establishes the label as the most recent exception-handler location. This statement cannot be used in a method that contains a lambda or query expression.

* `On Error Resume Next` establishes the `Resume Next` behavior as the most recent exception-handler location.


#### Resume Statement

A `Resume` statement returns execution to the statement that caused the most recent exception.

```antlr
ResumeStatement
    : 'Resume' ResumeClause? StatementTerminator
    ;

ResumeClause
    : 'Next'
    | LabelName
    ;
```

If the `Next` modifier is specified, execution returns to the statement that would have been executed after the statement that caused the most recent exception. If a label name is specified, execution returns to the label.

Because the `SyncLock` statement contains an implicit structured error-handling block, `Resume` and `Resume Next` have special behaviors for exceptions that occur in `SyncLock` statements. `Resume` returns execution to the beginning of the `SyncLock` statement, while `Resume Next` returns execution to the next statement following the `SyncLock` statement. For example, consider the following code:

```vb
Class LockClass
End Class

Module Test
    Sub Main()
        Dim FirstTime As Boolean = True
        Dim Lock As LockClass = New LockClass()

        On Error GoTo Handler

        SyncLock Lock
            Console.WriteLine("Before exception")
            Throw New Exception()
            Console.WriteLine("After exception")
        End SyncLock

        Console.WriteLine("After SyncLock")
        Exit Sub

Handler:
        If FirstTime Then
            FirstTime = False
            Resume
        Else
            Resume Next
        End If
    End Sub
End Module
```

It prints the following result.

```
Before exception
Before exception
After SyncLock
```

The first time through the `SyncLock` statement, `Resume` returns execution to the beginning of the `SyncLock` statement. The second time through the `SyncLock` statement, `Resume Next` returns execution to the end of the `SyncLock` statement. `Resume` and `Resume Next` are not allowed within a `SyncLock` statement.

In all cases, when a `Resume` statement is executed, the most recent exception is set to `Nothing`. If a `Resume` statement is executed with no most recent exception, the statement raises a `System.Exception` exception containing the Visual Basic error number `20` (Resume without error).


## Branch Statements

Branch statements modify the flow of execution in a method. There are six branch statements:

1. A `GoTo` statement causes execution to transfer to the specified label in the method. It is not allowed to `GoTo` into a `Try`, `Using`, `SyncLock`, `With`, `For` or `For Each` block, nor into any loop block if a local variable of that block is captured in a lambda or LINQ expression.
2. An `Exit` statement transfers execution to the next statement after the end of the immediately containing block statement of the specified kind. If the block is the method block, then control flow exits the method as described in Section [Control Flow](statements.md#control-flow). If the `Exit` statement is not contained within the kind of block specified in the statement, a compile-time error occurs.
3. A `Continue` statement transfers execution to the end of the immediately containing block loop statement of the specified kind. If the `Continue` statement is not contained within the kind of block specified in the statement, a compile-time error occurs.
4. A `Stop` statement causes a debugger exception to occur.
5. An `End` statement terminates the program. Finalizers are run before shutdown, but the finally blocks of any currently executing `Try` statements are not executed. This statement may not be used in programs that are not executable (for example, DLLs).
6. A `Return` statement with no expression is equivalent to an `Exit Sub` or `Exit Function` statement. A `Return` statement with an expression is only allowed in a regular method that is a function, or in an async method that is a function with return type `Task(Of T)` for some `T`. The expression must be classified as a value which is implicitly convertible to the *function return variable* (in the case of regular methods) or to the *task return variable* (in the case of async methods). Its behavior is to evaluate its expression, then store it in the return variable, then execute an implicit `Exit Function` statement.

```antlr
BranchStatement
    : GoToStatement
    | ExitStatement
    | ContinueStatement
    | StopStatement
    | EndStatement
    | ReturnStatement
    ;

GoToStatement
    : 'GoTo' LabelName StatementTerminator
    ;

ExitStatement
    : 'Exit' ExitKind StatementTerminator
    ;

ExitKind
    : 'Do' | 'For' | 'While' | 'Select' | 'Sub' | 'Function' | 'Property' | 'Try'
    ;

ContinueStatement
    : 'Continue' ContinueKind StatementTerminator
    ;

ContinueKind
    : 'Do' | 'For' | 'While'
    ;

StopStatement
    : 'Stop' StatementTerminator
    ;

EndStatement
    : 'End' StatementTerminator
    ;

ReturnStatement
    : 'Return' Expression? StatementTerminator
    ;
```

## Array-Handling Statements

Two statements simplify working with arrays: `ReDim` statements and `Erase` statements.

```antlr
ArrayHandlingStatement
    : RedimStatement
    | EraseStatement
    ;
```

### ReDim Statement

A `ReDim` statement instantiates new arrays.

```antlr
RedimStatement
    : 'ReDim' 'Preserve'? RedimClauses StatementTerminator
    ;

RedimClauses
    : RedimClause ( Comma RedimClause )*
    ;

RedimClause
    : Expression ArraySizeInitializationModifier
    ;
```

Each clause in the statement must be classified as a variable or a property access whose type is an array type or `Object`, and be followed by a list of array bounds. The number of the bounds must be consistent with the type of the variable; any number of bounds is allowed for `Object`. At run time, an array is instantiated for each expression from left to right with the specified bounds and then assigned to the variable or property. If the variable type is `Object`, the number of dimensions is the number of dimensions specified, and the array element type is `Object`. If the given number of dimensions is incompatible with the variable or property at run time a compile-time error occurs. For example:

```vb
Module Test
    Sub Main()
        Dim o As Object
        Dim b() As Byte
        Dim i(,) As Integer

        ' The next two statements are equivalent.
        ReDim o(10,30)
        o = New Object(10, 30) {}

        ' The next two statements are equivalent.
        ReDim b(10)
        b = New Byte(10) {}

        ' Error: Incorrect number of dimensions.
        ReDim i(10, 30, 40)
    End Sub
End Module
```

If the `Preserve` keyword is specified, then the expressions must also be classifiable as a value, and the new size for each dimension except for the rightmost one must be the same as the size of the existing array. The values in the existing array are copied into the new array: if the new array is smaller, the existing values are discarded; if the new array is bigger, the extra elements will be initialized to the default value of the element type of the array. For example, consider the following code:

```vb
Module Test
    Sub Main()
        Dim x(5, 5) As Integer

        x(3, 3) = 3

        ReDim Preserve x(5, 6)
        Console.WriteLine(x(3, 3) & ", " & x(3, 6))
    End Sub
End Module
```

It prints the following result:

```
3, 0
```

If the existing array reference is a null value at run time, no error is given. Other than the rightmost dimension, if the size of a dimension changes, a `System.ArrayTypeMismatchException` will be thrown.

__Note.__ `Preserve` is not a reserved word.


### Erase Statement

An `Erase` statement sets each of the array variables or properties specified in the statement to `Nothing`. Each expression in the statement must be classified as a variable or property access whose type is an array type or `Object`. For example:

```vb
Module Test
    Sub Main()
        Dim x() As Integer = New Integer(5) {}

        ' The following two statements are equivalent.
        Erase x
        x = Nothing
    End Sub
End Module
```

```antlr
EraseStatement
    : 'Erase' EraseExpressions StatementTerminator
    ;

EraseExpressions
    : Expression ( Comma Expression )*
    ;
```

## Using statement

Instances of types are automatically released by the garbage collector when a collection is run and no live references to the instance are found. If a type holds on a particularly valuable and scarce resource (such as database connections or file handles), it may not be desirable to wait until the next garbage collection to clean up a particular instance of the type that is no longer in use. To provide a lightweight way of releasing resources before a collection, a type may implement the `System.IDisposable` interface. A type that does so exposes a `Dispose` method that can be called to force valuable resources to be released immediately, as such:

```vb
Module Test
    Sub Main()
        Dim x As DBConnection = New DBConnection("...")

        ' Do some work
        ...

        x.Dispose()        ' Free the connection
    End Sub
End Module
```

The `Using` statement automates the process of acquiring a resource, executing a set of statements, and then disposing of the resource. The statement can take two forms: in one, the resource is a local variable declared as a part of the statement and treated as a regular local variable declaration statement; in the other, the resource is the result of an expression.

```antlr
UsingStatement
    : 'Using' UsingResources StatementTerminator
      Block?
      'End' 'Using' StatementTerminator
    ;

UsingResources
    : VariableDeclarators
    | Expression
    ;
```

If the resource is a local variable declaration statement then the type of the local variable declaration must be a type that can be implicitly converted to `System.IDisposable`. The declared local variables are read-only, scoped to the `Using` statement block and must include an initializer. If the resource is the result of an expression then the expression must be classified as a value and must be of a type that can be implicitly converted to `System.IDisposable`. The expression is only evaluated once, at the beginning of the statement.

The `Using` block is implicitly contained by a `Try` statement whose finally block calls the method `IDisposable.Dispose` on the resource. This ensures the resource is disposed even when an exception is thrown. As a result, it is invalid to branch into a `Using` block from outside of the block, and a `Using` block is treated as a single statement for the purposes of `Resume` and `Resume Next`. If the resource is `Nothing`, then no call to `Dispose` is made. Thus, the example:

```vb
Using f As C = New C()
    ...
End Using
```

is equivalent to:

```vb
Dim f As C = New C()
Try
    ...
Finally
    If f IsNot Nothing Then
        f.Dispose()
    End If
End Try
```

A `Using` statement that has a local variable declaration statement may acquire multiple resources at a time, which is equivalent to nested `Using` statements.  For example, a `Using` statement of the form:

```vb
Using r1 As R = New R(), r2 As R = New R()
    r1.F()
    r2.F()
End Using
```

is equivalent to:

```vb
Using r1 As R = New R()
    Using r2 As R = New R()
        r1.F()
        r2.F()
    End Using
End Using
```


## Await Statement

An await statement has the same syntax as an await operator expression (Section [Await Operator](expressions.md#await-operator)), is allowed only in methods that also allow await expressions, and has the same behavior as an await operator expression.

However, it may be classified as either a value or void. Any value resulting from evaluation of the await operator expression is discarded.

```antlr
AwaitStatement
    : AwaitOperatorExpression StatementTerminator
    ;
```

## Yield Statement

Yield statements are related to iterator methods, which are described in Section [Iterator Methods](statements.md#iterator-methods).

```antlr
YieldStatement
    : 'Yield' Expression StatementTerminator
    ;
```

`Yield` is a reserved word if the immediately enclosing method or lambda expression in which it appears has an `Iterator` modifier, and if the `Yield` appears after that `Iterator` modifier; it is unreserved elsewhere. It is also unreserved in preprocessor directives. The yield statement is only allowed in the body of a method or lambda expression where it is a reserved word. Within the immediately enclosing method or lambda, the yield statement may not occur inside the body of a `Catch` or `Finally` block, nor inside the body of a `SyncLock` statement.

The yield statement takes a single expression which must be classified as a value and whose type is implicitly convertible to the type of the *iterator current variable* (Section [Iterator Methods](statements.md#iterator-methods)) of its enclosing iterator method.

Control flow only ever reaches a `Yield` statement when the `MoveNext` method is invoked on an iterator object. (This is because an iterator method instance only ever executes its statements due to the `MoveNext` or `Dispose` methods being called on an iterator object; and the `Dispose` method will only ever execute code in `Finally` blocks, where `Yield` is not allowed).

When a `Yield` statement is executed, its expression is evaluated and stored in the *iterator current variable* of the iterator method instance associated with that iterator object. The value `True` is returned to the invoker of `MoveNext`, and the control point of this instance stops advancing until the next invocation of `MoveNext` on the iterator object.

