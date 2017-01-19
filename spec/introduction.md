# Introduction

The Microsoft&reg; Visual Basic&reg; programming language is a high-level programming language for the Microsoft .NET Framework. Although it is designed to be an approachable and easy-to-learn language, it is also powerful enough to satisfy the needs of experienced programmers. The Visual Basic programming language has a syntax that is similar to English, which promotes the clarity and readability of Visual Basic code. Wherever possible, meaningful words or phrases are used instead of abbreviations, acronyms, or special characters. Extraneous or unneeded syntax is generally allowed but not required.

The Visual Basic programming language can be either a strongly typed or a loosely typed language. Loose typing defers much of the burden of type checking until a program is already running. This includes not only type checking of conversions but also of method calls, meaning that the binding of a method call can be deferred until run-time. This is useful when building prototypes or other programs in which speed of development is more important than execution speed. The Visual Basic programming language also provides strongly typed semantics that performs all type checking at compile-time and disallows run-time binding of method calls. This guarantees maximum performance and helps ensure that type conversions are correct. This is useful when building production applications in which speed of execution and execution correctness is important.

This document describes the Visual Basic language. It is meant to be a complete language description rather than a language tutorial or a user's reference manual.

## Grammar Notation

This specification describes two grammars: a lexical grammar and a syntactic grammar. The lexical grammar defines how characters can be combined to form tokens; the syntactic grammar defines how the tokens can be combined to form Visual Basic programs. There are also several secondary grammars used for preprocessing operations like conditional compilation.

The grammars in this specification are written in ANTLR format -- see http://www.antlr.org/.

Case is unimportant in Visual Basic programs. For simplicity, all terminals will be given in standard casing, but any casing will match them. Terminals that are printable elements of the ASCII character set are represented by their corresponding ASCII characters. Visual Basic is also width insensitive when matching terminals, allowing full-width Unicode characters to match their half-width Unicode equivalents, but only on a whole-token basis. A token will not match if it contains mixed half-width and full-width characters.

Line breaks and indentation may be added for readability and are not part of the production.

## Compatibility

An important feature of a programming language is compatibility between different versions of the language. If a newer version of a language does not accept the same code as a previous version of the language, or interprets it differently than the previous version, then a burden can be placed on a programmer when upgrading his code from one version of the language to another. As such, compatibility between versions must be preserved except when the benefit to language consumers is of a clear and overwhelming nature.

The following policy governs changes to the Visual Basic language between versions. The term language, when used in this context, refers only to the syntactic and semantic aspects of the Visual Basic language itself and does not include any .NET Framework classes included as a part of the `Microsoft.VisualBasic` namespace (and sub-namespaces). All classes in the .NET Framework are covered by a separate versioning and compatibility policy outside the scope of this document.

### Kinds of compatibility breaks

In an ideal world, compatibility would be 100% between the existing version of Visual Basic and all future versions of Visual Basic. However, there may be situations where the need for a compatibility break may outweigh the cost it may impose on programmers. Such situations are:

* *New warnings.* Introducing a new warning is not, per se, a compatibility break. However, because many developers compile with "treat warnings as errors" turned on, extra care must be taken when introducing warnings.

* *New keywords.* Introducing new keywords may be necessary when introducing new language features. Reasonable efforts will be made to choose keywords that minimize the possibility of collision with users' identifiers and to use existing keywords where it makes sense. Help will be provided to upgrade projects from previous versions and escape any new keywords.

* *Compiler bugs.* When the compiler's behavior is at odds with a documented behavior in the language specification, fixing the compiler behavior to match the documented behavior may be necessary.

* *Specification bug.* When the compiler is consistent with the language specification but the language specification is clearly wrong, changing the language specification and the compiler behavior may be necessary. The phrase "clearly wrong" means that the documented behavior runs counter to what a clear and unambiguous majority of users would expect and produces highly undesirable behavior for users.

* *Specification ambiguity.* When the language specification should spell out what happens in a particular situation but doesn't, and the compiler handles the situation in a way that is either inconsistent or clearly wrong (using the same definition from the previous point), clarifying the specification and correcting the compiler behavior may be necessary. In other words, when the specification covers cases a, b, d and e, but omits any mention of what happens in case c, and the compiler behaves incorrectly in case c, it may be necessary to document what happens in case c and change the behavior of the compiler to match. (Note that if the specification was ambiguous as to what happens in a situation and the compiler behaves in a manner that is not clearly wrong, the compiler behavior becomes the de facto specification.)

* *Making run-time errors into compile-time errors.* In a situation where code is 100% guaranteed to fail at runtime (i.e. the user code has an unambiguous bug in it), it may be desirable to add a compile-time error that catches the situation.

* *Specification omission.* When the language specification does not specifically allow or disallow a particular situation and the compiler handles the situation in a way that is undesirable (if the compiler behavior was clearly wrong, it would a specification bug, not a specification omission), it may be necessary to clarify the specification and change the compiler behavior. In addition to the usual impact analysis, changes of this kind are further restricted to cases where the impact of the change is considered to be extremely minimal and the benefit to developers is very high.

* *New features.* In general, introducing new features should not change existing parts of the language specification or the existing behavior of the compiler. In the situation where introducing a new feature requires changing the existing language specification, such a compatibility break is reasonable only if the impact would be extremely minimal and the benefit of the feature is high.

* *Security.* In extraordinary situations, security concerns may necessitate a compatibility break, such as removing or modifying a feature that is inherently insecure and poses a clear security risk for users.

The following situations are not acceptable reasons for introducing compatibility breaks:

* *Undesirable or regrettable behavior.* Language design or compiler behavior which is reasonable but considered undesirable or regrettable in retrospect is not a justification for breaking backward compatibility. The language deprecation process, covered below, must be used instead.

* *Anything else.* Otherwise, compiler behavior remains backwards compatible.

### Impact Criteria

When considering whether a compatibility break might be acceptable, several criteria are used to determine what the impact of the change might be. The greater the impact, the higher the bar for accepting the compatibility breaks.

The criteria are:

* What is the scope of the change? In other words, how many programs are likely to be affected? How many users are likely to be affected? How common will it be to write code that is affected by the change?

* Do any workarounds exist to get the same behavior prior to the change?

* How obvious is the change? Will users get immediate feedback that something has changed, or will their programs just execute differently?

* Can the change be reasonably addressed during upgrade? Is it possible to write a tool that can find the situation in which the change occurs with perfect accuracy and change the code to work around the change?

* What is the community feedback on the change?

### Language deprecation

Over time, parts of the language or compiler may become deprecated. As discussed previously, it is not acceptable to break compatibility to remove such deprecated features. Instead, the following steps must be followed:

1. Given a feature that exists in version *A* of Visual Studio, feedback must be solicited from the user community on deprecation of the feature and full notice given before any final deprecation decision is made. The deprecation process may be reversed or abandoned at any point based on user community feedback.

2. full version (i.e. not a point release) *B* of Visual Studio must be released with compiler warnings that warn of deprecated usage. The warnings must be on by default and can be turned off. The deprecations must be clearly documented in the product documentation and on the web.

3. A full version *C* of Visual Studio must be released with compiler warnings that cannot be turned off.

4. A full version *D* of Visual Studio must subsequently be released with the deprecated compiler warnings converted into compiler errors. The release of *D* must occur after the end of the Mainstream Support Phase (5 years as of this writing) of release *A*.

5. Finally, a version *E* of Visual Studio may be released that removes the compiler errors.

Changes that cannot be handled within this deprecation framework will not be allowed.
