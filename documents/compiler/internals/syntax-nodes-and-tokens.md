# [VB.Net "Under the Hood"](README.md)    
### Syntax Nodes and Tokens

These represent the textual structure of VB.net code.

#### Structure of Syntax.xml

Syntax Nodes and Syntax Tokens are implemented via a XML document.   
Located at [.../src/Compilers/VisualBasic/Portable/Syntax/Syntax.xml](https://github.com/dotnet/roslyn/blob/master/src/Compilers/VisualBasic/Portable/Syntax/Syntax.xml)

###### Nodes and Attributes


| Node             | Attribute   | Description                 | Values                               |
|:---------------- | ----------- |:--------------------------- |:------------------------------------ |
| `node-structure` |             |                             |                                      |
|                  | `name `     | The name of the syntax node |                                      |
|                  | `parent`    | Inherits from this node     |                                      |
| `description`    |             |                             |                                      |
| `lm-equiv`       |             |                             |                                      |
|                  | `name`      |                             |                                      | 
| `native-equiv`   | `name`      |                             |                                      |
| `node-kind`      | `name`      |                             | list of nodekinds separated by `|` . |
| `child`          |             |                             |                                      |
|                  | `name`      |                             |                                      |
|                  | `optional`  |                             | `true` \| `false`                    |
|                  | `list`      |                             | `true` \| `false`                    |
|                  | `kind`      |                             |                                      |

###### Example

```xml
 <!--********************
      -  EnumStatement
      **********************-->
    <node-structure name="EnumStatementSyntax" parent="DeclarationStatementSyntax">
      <description>
        Represents the beginning statement of an Enum declaration. This node
        always appears as the Begin of an EnumBlock with Kind=EnumDeclarationBlock.
      </description>
      <lm-equiv name="TypeDeclarationNode"></lm-equiv>
      <native-equiv name="TypeStatement"></native-equiv>

      <node-kind name="EnumStatement">
        <lm-equiv name="Type" />
        <native-equiv name="Statement.Opcodes.Enum" />
      </node-kind>

      <child name="AttributeLists" optional="true" list="true" kind="AttributeList">
        <description>A list of all attribute lists on this declaration. If no attributes were specified, an empty list is returned.</description>
        <lm-equiv name="Attributes"></lm-equiv>
        <native-equiv name="Attributes"></native-equiv>
      </child>

      <child name="Modifiers" optional="true" list="true" kind="PublicKeyword|PrivateKeyword|ProtectedKeyword|FriendKeyword|ShadowsKeyword">
        <description>
          A list of all the modifier tokens that were present on this declaration. If no modifiers were specified, an empty list is returned.
        </description>
        <lm-equiv name="ModifierTokens"></lm-equiv>
        <native-equiv name="Specifiers"></native-equiv>
      </child>

      <child name="EnumKeyword" >
        <description>The "Enum" keyword.</description>
        <lm-equiv name="TypeKeyword"></lm-equiv>
        <native-equiv name="TypeKeyword"></native-equiv>
        <kind name="EnumKeyword" node-kind="EnumStatement"/>
      </child>

      <child name="Identifier" kind="IdentifierToken">
        <description>The name of the enum being declared.</description>
        <lm-equiv name="Name"></lm-equiv>
        <native-equiv name="Name"></native-equiv>
      </child>

      <child name="UnderlyingType" optional="true" kind="@AsClauseSyntax">
        <description>Optional "As XXX" clause describing the underlying type of the enumeration. If no As clause was specified, Nothing is returned.</description>
        <lm-equiv name="UnderlyingType"></lm-equiv>
        <native-equiv name="UnderlyingRepresentation"></native-equiv>
      </child>

    </node-structure>

```



#### Enabling Changes

```cmd
cd /d {your repo}

...\build\scripts\generate-compiler-code.cmd

build -buildAll
```

#### Hints & Tips

 * Becareful of your spellings as XML is case-sensitive.