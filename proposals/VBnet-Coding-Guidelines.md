# VB.net Coding Guidelines
 
 Coding Conventions
 
 * 80 character line limit    
 Is the recommended maximum length of single line of code.    
*As seen in a text editor, not in the context of the actual language specification*

 * Use 4 space indentation, not tabs.
 
 * Use plain code to validation parameters at public boundaries.    
   * Do not use Contracts or magic helpers.
   * Use `Debug.Assert()` for checks not needed in release builds. Always include a contextually meaningful message string in your assert to identify failure conditions.
   * Add assertions to documentation assumptions on non-local program state or parameter values, eg.    
   	*"At this point in parsing the scanner should have been advanced to a '.' token by the caller".*
   * Avoid allocations in compiler hot paths.
     * Avoid LINQ
     * Avoid using `For Each` over collections that do not have a struture based enumerator.
     * Consider using an object pool. There are many usages of object pools in the compiler to see an example.
   
 * Method Parameters
   * Prefer single Line *(if within 80c recommendation)*    
```vbnet
ExampleMethod( arg0 As Integer ) As Integer
```
Otherwise use a single parameter per a line. eg
```vbnet
ExampleMethod(
	       arg0 As Integer,
               arg1 As Integer,
      Optional arg2 As String = Nothing,
    ParamArray args As String() 
	     ) As ...
```
  * `If ... Then`    
Single Line `If ... Then `, is permissable usage for example: Parameter validation aka Guards
```vbnet
If someArgument Is Nothing Then Throw New NullArgumentException(NameOf(someArgument))
```
Provided that the total length the line is smaller or equal to the recommended line length, otherwise it is recommend to use the block form.
```vbnet
If someArgument Is Nothing Then
    Throw New NullArgumentException(NameOf(someArgument))
End If
``` 

 * `Dim x = ...`    
 Provided that the resultant type can easily be inferred by the reader, the specification of the type can be omitted.
 * `Dim x As New ...` is prefered over `Dim x As ... = New ....`    
 To reduce the visual clutter of repeatition of easily knownable type specifiers.

 * Field names.
   * Begin with a `_` or `m_`
 * Where viable do not specify `Me.`
 * Use `IsNot` over `Not ... Is ...`
 
 
 * `Select Case`
```vbnet
Select Case obj
    Case 0
        ...
    Case 1
       ...
    Case Else
       ...
End Select
```

 * XML Document comments on Public API boundaries.
```vbnet
''' <summary>Add two integers together.</summary>
''' <param name="left">First integer.</param>
''' <param name="right">Second integer.</param>
''' <returns>The result (an integer) of adding two integers together.</returns>
Public Function Add(left As Integer, right As Integer) As Integer
End Function
```