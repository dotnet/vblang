# [VB.Net "Under the Hood"](README.md)    
### Bound Nodes

These represent the semantic structure of VB.net code.

#### Structure of BoundNodes.xml

Bounds Nodes are implemented via a XML document.   
Located at [.../src/Compilers/VisualBasic/Portable/BoundTree/BoundNodes.xml](https://github.com/dotnet/roslyn/blob/master/src/Compilers/VisualBasic/Portable/BoundTree/BoundNodes.xml)

#### Enabling Changes

```cmd
cd /d {your repo}

...\build\scripts\generate-compiler-code.cmd

build -buildAll
```

#### Hints & Tips

 * Becareful of your spellings as XML is case-sensitive.