# VB.Net "Under the Hood"

  The following documentation will cover the internals of VB.net code, giving hints and tips.

### Internal Gubbins
  * [Syntax Nodes and Syntax Tokens](syntax-nodes-and-tokens.md)
    * Represents the visual structure of VB.net code.
  * [Bound Nodes](bound-nodes.md)
    * Represents the executional structure of VB.net code.
  * [Scanner](scanner.md)
    * Reads text characters from the source and converts them syntax nodes / syntax tokens.  
  * [Parser](parser.md)
    * Reads tokens / nodes according to the syntax rules of the VB.net language.
  * [Binding](binding.md)
    * Transforms tokens into BoundNodes according to the semantic rules of the VB.net language.
  * [Lowering](lowering.md)
    * Transformation of BoundNodes into simpler forms.
  * [Emitter](emitter.md)
    * Generate the actual IL used.  

### Editor Features
  
  * Classification
    * Related to the colourisation of the code.

### IDE Tools
  
 * Code Fix
 * 