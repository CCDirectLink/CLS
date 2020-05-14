# Standard Mod Library

## Current Situation

Currently, there is no acceptable API for mods to easily execute actions common to all mods. For example, if one mod wants to modify the menu, it needs to work around other mods doing the same. At the moment this is handled through `Simplify` but the mod is very outdated, messy and contains CCLoader specific code.

## Solution

The problem can be solved by a set of APIs defined by CLS that define a common interface across all mods and modloaders.

### Advantages

* Simple interfaces for mods.
* Possibility of shared code between modloaders (see `API specification requirements` > `Implementation requirements`).
* Better documentation.

### Disadvantages

* The API needs to be defined by CLS.
* Need special treatment by all modloaders (See `Load order`).
* APIs need to be carefully designed to allow future expansion and/or changes.

## Implementation details

### Load order

As a standard library is expected to be used by every mod it should not require a dependency. Thus a modloader must load the standard library before all other mods.

## API specification requirements

### Naming

#### Class names

All standard libary APIs must only be accessable through a `ccmod` object. As such the naming should consider using simple names as class names.

Requirement:
* A name must be written in camelCase. 

For example:

`ccmod.resources` is a simple and rather intuitive name.

`ccmod.crossCodeModdingResouceManager` is both long and includes repetition when combined with `ccmod`.

#### Members and Fields

Requirement:
* All names must be written in camelCase
* All names must take scope into account.

For example:

`ccmod.resources.load` takes scope into account

`ccmod.resources.loadResouce` does not takes scope into account

### Fields

Fields must be marked as `readonly`. There is no `Implementation Requirement` for API specifications to force API implementations to ensure this. APIs can expect consumers to apply to this requirement.

## Declarations

All APIs must provide Typescript Declarations for all accessable elements. The full contents of the corresponding `.d.ts` file must be provided.

## Versioning

All APIs must define a version that appies to the rules of semantic versioning inside the specification.

## Documention

All declared elements need descriptions in the form TSDoc and as textual description inside the specification.

Should an element require native dependencies this must be included in the textual description.

## Stage

The specification must include the loading stage at which the interface becomes available.

## Native Dependencies

Should an API require native dependencies it bust either require dummy implemention in the event that none are available or provide a clearly defined way to check if it is.

## Implementation Requirements

It must be possible to implement the specification
* by only using CLS specifications and original game code.
* within ES6 modules.
* within nodejs modules.

## Example API specification

# Hooks
**Version 1.0.0**  
**Stage: preload**

An API that manages hooks other than `ig.Class.inject`

### func

A function that takes an object, member name and callback function. The callback function takes two arguments:
* A function that contains the original function that is bound to the original context.
* The original arguments of the call.

The return value of the callback is passed to the original caller.

The function will return true if the target was hooked successfully.


```ts
function func(obj: any, name: string, callback: (original: any) => any): bool;
```

## Full declaration

```ts
declare namespace ccmod {
    declare const hooks: Hooks;
}

export class Hooks {
    public func(obj: any, name: string, callback: (original: any) => any): bool;
}
```

---

Author: 2767mr (Discord: 224155607278551040)
