# Visual Basic Language Design Meeting

February 21,2018

## Agenda

* Community Update
* Upgrade Project - Discussion
* Private Protected - Discussion
* Issue [#37 -Champion "Await in Catch and Finally"](https://github.com/dotnet/vblang/issues/37)
* Issue [#228 - Default property for reading and setting bit flags](https://github.com/dotnet/vblang/issues/228) (and others)
* [C# proposal: Default interface methods](https://github.com/dotnet/csharplang/blob/master/proposals/default-interface-methods.md)

## Community Update

### Introduce?

There has been a request by the community for people to introduce themselves.

This will remain at the discretion of the individual.

### Brice & EF

@BriceLam has volunteered (side project) to maintain a repository for the development of a generator so migrations and reverse engineering work with VB projects.

Kathleen will help publicize.

When the project nears completion, Kathleen will explore best permanent location with Brice and contributors.

Yeah Brice!

### MVP Summit

MVP Summit is March 5-8.

#### Language Designers Panel

Kathleen will represent VB on the Language Designers Panel.

#### Lunch session

There will be a lunch session on Visual Basic. Kathleen determined this odd time with Jeff to avoid placing it opposite a session with likely interest to VB developers. We assume all VB MVPs are also interested in other things.

#### Hackathon

Thursday (March 8) at the MVP Summit there will be an MVP Hackathon.

Do we want to organize something?

No, leave it to Brice on the EF generator and community for organization

### Docs

There are some really out of date docs for VB and way too many docs for the docs teams to fix.

Kathleen will discuss with the community and hopefully get some help with this.

### Blogpost with WebAPI example

@KlausLoeffelmann is working on a blog post (with help from @obelink and others) that will illustrate VB working with a VB backend and an Angular front end.

Kathleen is working planning other blog posts.

## Upgrade Project - Discussion

The customer problem: To change the version of VB a project is targeting, you have to edit the .vbproj file. Ick. Extra Ick because VB programmers put high value on simplicity/low ceremony.

The technical problem: 3-4 non-trivial features need to land to fix this. Want to do it, but won't be immediate.

With the delay, do we hold feature announcements or apologize that people have to edit right now?

Kathleen will corral the solution.

Do not hold any announcements. The folks that want new features want them.

## Private Protected - Discussion

Private protected specifies that a member is only visible from code that is both within the same assembly and within a derived class.

The earlier decision was to parallel the IL names with VB styling and use FriendAndProtected and FriendOrProtected. FriendOrProtected would be a pretty listing fix for Protected Friend for symmetry.

For context, in C# this feature was requested loudly by a few people. An agonizing and relatively extreme attempt at a better named and the community accepted that the feature by any name was better than no feature. Thus it is `private protected` in C#. 

Kathleen did not find an issue on this in VBLang.

During implementation, we messed up. The feature was implemented in VB as `Private Protected` (in any order). The feature is currently present, but not announced. Because it is present, we will support `Private Protected` even if we did something else. 

Clearly the earlier decision was more VB like. But, very few people care about the feature, it exists right now in VB as `Private Protected`, and coders may hear about the concept via C# blogs so parallel naming may have one advantage.

Decision: Stick with what we've got because we've got it. If people use the feature and there is a push for clearer naming, deal with it then.

## Issue [#37 - Champion "Await in Catch and Finally"](https://github.com/dotnet/vblang/issues/37)

Partial discussion because @VSadov not present and he has the most history.

This was considered for an earlier release, but we ran out of time.

In general the work is similar to C#. But VB specific things, GoTo and OnError may make it more difficult.

At least some of the challenging scenarios aren't legal in async methods anyway.

GoTo and OnError seem unlikely in "modern" code that would use Async. Perhaps we could narrow the scenarios to simplify implementation. 

Reconsider with Vlad, probably next week.

We should check that this is important and how much work before we commit. Kathleen will see if a prioritization survey at the Summit makes sense and there will be time.

## Issue [#228 - Default property for reading and setting bit flags ](https://github.com/dotnet/vblang/issues/228) (and others)

Flagged enums have challenges in both C# and VB. In VB they are rather painful. A solution at the API level would be great.

An enum constraint by itself does not allow this to be solved via extension methods. Darn.

Tabled for Kathleen to discuss with API team to see if this is on their radar.

## [C# proposal: Default interface methods](https://github.com/dotnet/csharplang/blob/master/proposals/default-interface-methods.md)

C# has a proposal for default interface members. A compelling scenario for this involves allowing interfaces to change over time without breaking all implementors. The implication is that people will then evolve interfaces, and if VB can't handle this, it will have a serious problem.

C# and Runtime are designed and prototyped (for C# 8 timeframe). The design is not set.

There will still be issues with older compiler versions unable to access interfaces that have changed (C#/VB). So it's not clear how public interfaces will take advantage of this feature.

Talk to @AnthonyDGreen for background.

No action.