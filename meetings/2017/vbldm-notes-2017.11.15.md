# Visual Basic Language Design Meeting
November 15th, 2017

## Agenda
* Proposal [#195](https://github.com/dotnet/vblang/issues/195) - `Static` Property Variables
* Proposal [#196](https://github.com/dotnet/vblang/issues/196) - Implicit Property Backing Fields
* Proposal [#197](https://github.com/dotnet/vblang/issues/197) - Inferred `Set` Parameter Type
* Scenario [#219](https://github.com/dotnet/vblang/issues/219) - Implementing INotifyPropertyChanged is Tedious
    * Proposal [#107](https://github.com/dotnet/vblang/issues/107) - Replaceable Members
    * Proposal [#198](https://github.com/dotnet/vblang/issues/198) - Bindable Classes and Properties
    * Proposal [#107](https://github.com/dotnet/vblang/issues/194) - `WithPropertyEvents` Modifier to Enable Easy INotifyPropertyChanged Implementations

## Proposal [#195](https://github.com/dotnet/vblang/issues/195) - `Static` Property Variables

**Decisions**
* **Rejected**. We don't see why this one narrow use case for members nested within other members get's special treatment. If we want to enable nesting we'd look at local functions and types as well.

## Proposal [#196](https://github.com/dotnet/vblang/issues/196) - Implicit Property Backing Fields

**Decisions**
* **Rejected**. The design team found this idea deeply unsettling.

## Proposal [#197](https://github.com/dotnet/vblang/issues/197) - Inferred `Set` Parameter Type

**Decisions**
* **Approved-in-Principle**.

**Discussion**

Technically the entire parameter list on `Set` is already optional and we could get most of the value of this proposal by stopping the IDE from generating it because an implicit parameter named `Value` is created. We could just color it blue. But it's also fairly easy to just do the "inference" as when we're creating the setter `MethodSymbol` we already have the type of the property in hand.

## Scenario [#219](https://github.com/dotnet/vblang/issues/219) - Implementing INotifyPropertyChanged is Tedious

These three proposals fall on a spectrum from most general (#107) to most specific (#198), with #194 falling somewhere in between. We 

Proposal #194 seems to strike a good balance. We like that it could support a few more scenarios than just `INotifyPropertyChanged`, e.g. logging, "Freezable" objects. How do we choose?

* We know that we wouldn't want both `Bindable` properties and `WithPropertyEvents`; they're mutually exclusive.
* We know that if we had either of the above that wouldn't stop us from doing `Replaces` w/ source generators at a future date.
* We're fairly confident that if we had `Replaces` w/ source generators we wouldn't do `Bindable` or `WithPropertyEvents` as `INotifyPropertyChanged` is basically the poster child for the source generator feature.

We need to know the status of source generators and how far out they are before deciding. We should investigate and revisit with findings in a future meeting.

**Decisions**
* **Investigate status of source generators and revisit in future meeting**.

### Proposal [#107](https://github.com/dotnet/vblang/issues/107) - Replaceable Members

We don't know the status of this and source generators. We should investigate and come back with findings in future meeting.

### Proposal [#198](https://github.com/dotnet/vblang/issues/198) - Bindable Classes and Properties

_"Are the savings of this over #194 just the IDE automatically generating the bridge method? If so, we should go with the more general feature and just generate the correct code by default if you implement `INotifyPropertyChanged`"_.

### Proposal [#194](https://github.com/dotnet/vblang/issues/194) - `WithPropertyEvents` Modifier to Enable Easy INotifyPropertyChanged Implementations

Can see a few use-cases for this. You could imagine implementing a "freezable" type immutable class whose properties all throw if you try to set them after the object has been "frozen". Need a better modifier though.
