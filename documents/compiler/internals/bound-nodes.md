# [VB.Net "Under the Hood"](README.md)    
### Bound Nodes

These represent the semantic structure of VB.net code.

#### Structure of BoundNodes.xml

Bounds Nodes are implemented via a XML document.   
Located at [.../src/Compilers/VisualBasic/Portable/BoundTree/BoundNodes.xml](https://github.com/dotnet/roslyn/blob/master/src/Compilers/VisualBasic/Portable/BoundTree/BoundNodes.xml)

##### Nodes and Attributes

| Node             | Attribute           | Description                 | Values                               |
|:---------------- | ------------------- |:--------------------------- |:------------------------------------ |
| `Tree`           |                     |                             |                                      |
|                  | `Root`              |                             |                                      |
| `ValueType`      | `Name`              |                             |                                      |
|                  |                     |                             |                                      |
| `AbstractNode`   | `Name`              |                             |                                      |
|                  | 'Base               |                             |                                      |
|                  |                     |                             |                                      |
| `Node`           | `Name`              |                             |                                      |
|                  | `Base`              |                             |                                      |
|                  | `HasValidate`       |                             | `true` \| `false`                    |
|                  |                     |                             |                                      |
| `Field`          | `Name`              |                             |                                      | 
|                  | `Type`              |                             |                                      |
|                  | `Null`              |                             | `allow` \| `always` | `disallow`     |
|                  | `Override`          |                             |                                      |
|                  | `PropertyOverrides` |                             | `true` \| `false`                    |
|                  | `SkipInVisitor`     |                             | `true` \| `false`                    |
|                  |                     |                             | `true` \| `false`                    |
|                  |                     |                             |                                      |

##### Example
```xml
    <Node Name="BoundUnaryOperator" Base="BoundExpression" HasValidate="true">
        <!-- Type is required for this node type; may not be null -->
        <Field Name="Type" Type="TypeSymbol" Override="true" Null="disallow"/>

        <Field Name="OperatorKind" Type="UnaryOperatorKind"/>
        <Field Name="Operand" Type="BoundExpression"/>
        <Field Name="Checked" Type="Boolean"/>
        <Field Name="ConstantValueOpt" PropertyOverrides="true" Type="ConstantValue" Null="allow"/>
    </Node>
```

#### Enabling Changes

```cmd
cd /d {your repo}

...\build\scripts\generate-compiler-code.cmd

build -buildAll
```

#### Hints & Tips

 * Becareful of your spellings as XML is case-sensitive.