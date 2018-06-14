# Visual Basic Language Design Meeting

June 13, 2018

Last week's meeting subsumed with other issues.

## Agenda

* Review process
* Issues reviewed

## Review process

We'll review issues and use one of the following labels

LDM In Process: LDM review is in progress
LDM Considering: LDM reviewed and think this has merit
LDM No Plans: LDM reviewed and this feature is unlikely to move forward in the foreseeable future(*)
LDM Rejected: LDM reviewed and rejected

(*) This is the description we decided after our first description didn't have as much transparency as we intended.

We will begin reviewing with issues that have been quiet for a couple of months. Our goals with this  is not to disrupt active conversations.

We will put our logic in the issue itself, not in the meeting notes as we think it will be more accessible.

A few notes on our overall thinking:

* We will almost never make breaking changes to Visual Basic (often resulting in rejected label)
  * There may be some extreme edge cases that we believe highly unlikely ever
  * We may be forced, and we will do everything to avoid being forced
  * We are just beginning planning for .NET Core 3 version of Visual Basic, but currently anticipate a very high bar for breaking changes.
* We strongly believe that Visual Basic has a stance - a way of doing things. We will strive to maintain consistency with things being "VB-like" (often resulting in a No Plans or rejected label))
* Our observation is that the vast majority of Visual Basic users are not asking for us for new features so our bar for expansion of the surface area - making a second way to do things - will be relatively high even when it's a good idea (often resulting in a No Plans label)
* Consistent with the [Visual Basic language strategy](https://blogs.msdn.microsoft.com/dotnet/2017/02/01/the-net-language-strategy/) C# will take the lead on some issues - particularly those that would involve changes to the CRL or .NET libraries. We'll note this and suggest discussion in CSharpLang

# Reviewed

Logic for the decisions are included in comments adjacent to the label change for these issues

[#47](https://github.com/dotnet/vblang/issues/47) labeled: No Plans
[#256](https://github.com/dotnet/vblang/issues/256) labeled: No Plans
[#272](https://github.com/dotnet/vblang/issues/272) labeled: No Plans
[#244](https://github.com/dotnet/vblang/issues/244) labeled: No Plans
[#62](https://github.com/dotnet/vblang/issues/62) labeled: No Plans
[#229](https://github.com/dotnet/vblang/issues/229) labeled: No Plans

[#288](https://github.com/dotnet/vblang/issues/288) labeled: Rejected

