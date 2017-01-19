* Full Language Specification: [Markdown](spec)
* List of [Active](proposals), [Adopted](proposals/adopted), and [Rejected](proposals/rejected) proposals can be found in the [proposals folder](proposals).
* Archives of mailing lists discussions can be found [here](https://mailinglist-archives.org/).
* Archives of notes from design meetings, etc., can be found in the [notes folder](notes).

# Design Process

1. To submit, support, and discuss ideas please subscribe to the [language design mailing list](https://vblang.org/design-mailing-list).

2. Ideas deemed cool by the language design team should be turned into a more fleshed out proposal based on the [proposal template](proposals/proposal-template.md). A good idea should:
    * Fit with the general theme and aesthetic of the language.
    * Not introduce subtly alternate syntax for existing features.
    * Add a lot of value for a clear set of users.
    * Not add significantly to the complexity of the language, especially for new users.  

3. A prototype owner (who may or may not be proposal owner) should implement a prototype in their own fork of the [Roslyn repo](https://github.com/dotnet/roslyn) and share it with the design team and community for feedback. A prototype must meet the following bar:
	* Parsing (if applicable) should be resilient to experimentation--typing should not cause crashes.
	* Include minimal tests demonstrating the feature at work end-to-end.
	* Include minimal IDE support (keyword coloring, formatting, completion).

4. Once a prototype has proven out the proposal and the proposal has been _approved-in-principle_ by the design team, a feature owner (who may or may not be proposal or prototype owner(s)) implemented in a feature branch of the [Roslyn repo](https://github.com/dotnet/roslyn). The bar for implementation quality can be found [here](https://github.com/dotnet/roslyn).

5. Design changes during the proposal or feature implementation phase should be fed back into the original proposal as a PR describing the nature of the change and the rationale.

6. A PR should be submitted amending the formal language specification with the new feature or behavior.

7. Once a feature is implemented and merged into shipping branch of Roslyn and the appropriate changes merged into the language specification, the proposal should be archived under a folder corresponding to the version of the language in which it was included, e.g. [VB 15.1 proposals](proposals/adopted/vb15.1)). Rejected proposals are archived under the [rejected folder](proposals/rejected).

**DISCLAIMER**: An active proposal is under active consideration for inclusion into a future version of the Visual Basic .NET programming language but is not in any way guaranteed to ultimately be included in the next or any version of the language. A proposal may be postponed or rejected at any time during any phase of the above process based on feedback from the design team, community, code reviewers, or testing.
