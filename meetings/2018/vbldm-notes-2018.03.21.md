# Visual Basic Language Design Meeting

March 21, 2018

## Agenda

* MVP Summit recap
* Issue #86 [Optimize conversions from calls to Int/Fix functions and System.Math methods known to return whole Single/Double values to native opcodes](https://github.com/dotnet/vblang/issues/86)
* Issue #280 [Can The Visual Basic DLL Be Open Sourced?](https://github.com/dotnet/vblang/issues/280)
* Issue #282 [An update from the Design Safari on the `INotifyPropertyChanged` scenario](https://github.com/dotnet/vblang/issues/282)

Anthony D. Green attended as a guest!

## MVP Summit recap and poll discussion

We were able to communicate that helping with documentation counts as OSS contributions for MVP status and that testing is also considered. Need more work to track testing, but definitely issues raised on GitHub.

Work was done on the EF Visual Basic generators at the Hackathon. This continues for now in [@BriceLam 's repo](https://github.com/bricelam/EFCore.VisualBasic)


## Issue #86 [Optimize conversions from calls to Int/Fix functions and System.Math methods known to return whole Single/Double values to native opcodes](https://github.com/dotnet/vblang/issues/86)

CInt and Fix call methods on System.Math, and are thus slow(ish).

There are two issues, this addressed only the optimization. This does not effect or comment on considering syntax enhancements.

People that are currently using CInt(Fix()) want that optimized. While it seems slightly cryptic, it's the right way to get the truncated value and nothing else. This is what we can optimize.

Treat it as an optimization, not a feature to keep it narrow and get it done.

Probably some people that use CInt() probably don't care and do want optimization. But this is a breaking change and can't be done.

Do we want an analyzer that finds slow CInts? It would change the semantics of the program. Probably won't do this. Folks might write this, that would be fine if they communicate usage. 

Maybe consider up-for-grabs once we have a proposal.

Anthony has an analysis that he's going to send analysis that evaluates that results are identical.

What about expression trees?

Expression trees themselves will be unchanged. The IL output of expression trees is not expected to be high performance. Thus, plan to do what is easy. If code is shared, the optimization will also happen in the IL generation from expression trees.

Neal will create an issue on Roslyn.

## Issue #280 [Can The Visual Basic DLL Be Open Sourced?](https://github.com/dotnet/vblang/issues/280)

Is this a request to build a new VB runtime that is based on .NET Core and standard?

Looking at a new layering. Those things forever typed to Windows can't be handled the same way as things that could be in standard.

We'lll think further on this which needs discussion with the runtime folks.

We need more information on the goals of this request.

## Issue #282 [An update from the Design Safari on the `INotifyPropertyChanged` scenario](https://github.com/dotnet/vblang/issues/282)

We spent the rest of the meeting with Anthony's demo for affecting behavior using attributes and compile time rewriting. Overall impression was positive. 

## Meetings

We will miss next week's meeting due to Kathleen's travel. 