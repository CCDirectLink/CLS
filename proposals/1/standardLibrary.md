# Standard Mod Library

## Current Situation

Currently, there exists no acceptable API for mods to easily perform certain tasks non-destructively. For example, if one mod wants to modify the menu, it needs to work around other mods doing the same. At the moment these tasks are most commonly handled through the `Simplify` API, but this mod is outdated, messy, and contains CCLoader-specific code.

## Solution

This problem can be solved by defining a set of APIs in CLS to be implemented by modloaders and used by mods to perform these common tasks. This set of APIs will be collectively referred to below as the "Standard Library," or "stdlib" for short.

### Advantages

* Simple interfaces for mod developers; porting between modloaders becomes trivial.
* Possibility of shared code between modloaders (see `API specification requirements` > `Implementation requirements`).
* Better documentation.

### Disadvantages

* The stdlib specification needs to be created in CLS first.
* A Standard Library "mod" needs special treatment from all modloaders (See `Load order`).
* The stdlib needs to be designed carefully to allow for future changes and/or expansion.

## Implementation details

### Load order

As a Standard Library may be used by (i.e. a dependency of) any other mod, it ideally should not have any dependencies of its own. Thus, a modloader should load the Standard Library before all other mods.

## API specification requirements

### Naming

#### Package names

The Standard Library must use a `ccmod` Javascript object at global scope as its only entry point. The properties of this `ccmod` object should only consist of references to zero or more **packages**. A package is itself a Javascript object that can have zero or more classes, methods, fields, or sub-packages as its properties.

Since all references to packages in the stdlib must be routed through the `ccmod` object, designers are free to use simple package names without fear of polluting the global namespace.

(Note that an implementation of the Standard Library may have additional non-standard ways to access its contents.)

Requirements:
* A package name must be in `camelCase`.
* A package name should strive to be as simple as possible; prefer subpackages to long package names.

Examples:

`ccmod.resources` is a simple and rather intuitive name.

`ccmod.crossCodeModdingResourceManager` is both long and includes repetition when combined with `ccmod`.

`ccmod.menu.buttons` is preferable to `ccmod.menuButtons`.

#### Class names

Requirement:
* Names must be in `PascalCase`.
* Names should take their enclosing package name into account to avoid redundancy with that name.

For example:

`ccmod.resources.Loader` takes the package name into account.

`ccmod.resources.ResourceLoader` repeats the word "resource"; it does not take its package name into account.

#### Member and Field names

Requirement:
* Names must be in `camelCase`.
* Names should take their enclosing package name into account to avoid redundancy with that name.

For example:

`ccmod.resources.load` takes the package name into account.

`ccmod.resources.loadResource` repeats the word "resource"; it does not take its package name into account.

### Fields

Fields should be marked `readonly`. The Standard Library can expect users to adhere to this requirement.

## Declarations

An API in the Standard Library must provide TypeScript declarations for all accessible fields, methods, and classes. The full contents of the corresponding `.d.ts` file should be provided in the API.

## Versioning

An API in the Standard Library must have a version number that adheres to Semantic Versioning.

## Documentation

The behavior of all methods, classes, and resources in the stdlib should be documented in their respective specifications. They also should be documented in the TSDoc format in the source of their TypeScript declarations (see `Declarations`).

## Stage

The specification for a method, class, resource, or package must include the first loading stage that the interface is guaranteed to be available from.

## Native Dependencies

If a method or class requires native dependencies, this must be documented in its specification. A specification for one of these methods or classes must either define an alternate behavior in the event that no native dependencies are available, or provide a well-defined way to check for the presence of aforementioned native dependencies.

## Implementation Requirements

It must be possible to implement the specification by only using CLS specifications and the game's source code, and implementor code both within ES6 modules and within nodejs modules.

## Example API specification

---

# Hooks (package `ccmod.hooks`)
**Version 1.0.0**  
**Stage: preload**

An API to manage hooks other than `ig.Class#inject`.

### func

A function that takes an object, a member name, and a callback function. The callback function takes two arguments:
* A function that calls the original function, bound to the original context.
* The original arguments of the call.

The return value of `callback` is passed to the original caller.

The function will return `true` if its target was hooked successfully.


```ts
function func(obj: any, name: string, callback: (original: any) => any): bool;
```
