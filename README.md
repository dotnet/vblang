# Visual Basic .NET Language Design

Welcome to the official repo for Visual Basic .NET language design.

* Full Language Specification: [Markdown](spec)
* List of proposals can be found in the [proposals folder](proposals).
* Archives of notes from design meetings, etc., can be found in the [meetings folder](meetings).

If you discover bugs or deficiencies in the above, please leave an issue to raise them, or even better: a pull request to fix them.

For *new feature proposals*, however, please raise them for [discussion](https://github.com/dotnet/vblang/labels/Discussion), and *only* submit a proposal as a pull request if invited to do so by a member of the Language Design Team (a "champion").

## Contribute

### Moving to .NET Core

VB is moving to .NET Core. This is the move that was important since there will be no more language development on framework in any language. And it will be a while until we start work on VB Next. We're still months away from getting Visual Basic .NET 16.0 out the door.

The biggest thing you could do for Visual Basic.NET right now is to download the latest [.NET Core Preview](https://dotnet.microsoft.com/download/dotnet-core/3.0) and give feedback on how well it works for you. Testing is not the same as having it in the hands of real users. That's the foundation for the language features we'll do in the future.

### Discussion

Discussion pertaining to language features takes place in the form of issues in this repo, under the [Discussion label](https://github.com/dotnet/vblang/labels/Discussion).

If you want to suggest a feature, discuss current design notes or proposals, etc., please [open a new issue](https://github.com/dotnet/vblang/issues/new), and it will be tagged Discussion.

GitHub is not ideal for discussions, but it is beneficial to have language features discussed nearby to where the design artifacts are. Comment threads that are short and stay on topic are much more likely to be read. If you leave comment number fifty, chances are that only a few people will read it. To make discussions easier to navigate and benefit from, please observe a few rules of thumb:

- Discussion should be relevant to Visual Basic .NET language design. Issues that are not will be summarily closed.
- Choose a descriptive title for the issue, that clearly communicates the scope of discussion.
- Stick to the topic of the issue title. If a comment is tangential, start a new issue and link back.
- If a comment goes into detail on a subtopic, also consider starting a new issue and linking back.
- Is your comment useful for others to read, or can it be adequately expressed with an emoji reaction to an existing comment?

### Design Process

Visual Basic .NET is designed by the Visual Basic .NET Language Design Team (LDT).

1. To submit, support, and discuss ideas please use the [Discussion label](https://github.com/dotnet/vblang/labels/Discussion).

2. Ideas that the LDT feel could potentially make it into the language should be turned into [proposals](proposals), based on this [template](proposals/proposal-template.md), either by members of the LDT or by community members by invitation from the LDT. The lifetime of a proposal is described in [proposals/README.md](proposals/README.md). A good proposal should:
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

## Language Design Meetings

Language Design Meetings (LDMs) are held by the LDT and occasional invited guests, and are documented in Design Meeting Notes in the [meetings](meetings) folder, organized in folders by year. The lifetime of a design meeting note is described in [meetings/README.md](meetings/README.md). LDMs are where decisions about future Visual Basic .NET versions are made, including which proposals do work on, how to evolve the proposals, and whether and when to adopt them.

## Implementation

The reference implementation of the Visual Basic .NET language can be found in the [Roslyn repository](https://github.com/dotnet/roslyn). Until recently, that was also where language design artifacts were tracked. Please allow a little time as we move over active proposals.

**DISCLAIMER**: An active proposal is under active consideration for inclusion into a future version of the Visual Basic .NET programming language but is not in any way guaranteed to ultimately be included in the next or any version of the language. A proposal may be postponed or rejected at any time during any phase of the above process based on feedback from the design team, community, code reviewers, or testing.
