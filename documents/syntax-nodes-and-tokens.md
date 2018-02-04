# [VB.Net "Under the Hood"](README.md)    
### Syntax Nodes and Tokens

These represent the textual structure of VB.net code.

#### Structure of Syntax.xml

Syntax Nodes and Syntax Tokens are implemented via a XML document.   
Located at [.../src/Compilers/VisualBasic/Portable/Syntax/Syntax.xml](https://github.com/dotnet/roslyn/blob/master/src/Compilers/VisualBasic/Portable/Syntax/Syntax.xml)

#### Enabling Changes

```cmd
cd /d {your repo}

...\build\scripts\generate-compiler-code.cmd

build -buildAll
```

#### Hints & Tips

 * Becareful of your spellings as XML is case-sensitive.