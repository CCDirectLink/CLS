# Standard Mod Library

CLS defines a set of APIs to be implemented by modloaders and used by mods to perform these common tasks. This set of APIs will be collectively referred to below as the "Standard Library," or "stdlib" for short.

## Implementation details

### Load order

As a Standard Library may be used by (i.e. a dependency of) any other mod, it must not have any dependencies to mods outside the Standard Library of its own. Thus, a modloader must load the Standard Library before all other mods.

## API specification requirements

### Naming

#### Package names

The Standard Library must use a `ccmod` Javascript object at global scope as its only entry point. The properties of this `ccmod` object should only consist of references to zero or more **packages**. A package is itself a Javascript object that can have zero or more classes, methods, fields, or sub-packages as its properties.

All references to packages in the stdlib must be routed through the `ccmod` object. An implementation of the Standard Library may have additional non-standard ways to access its contents.

Requirement:
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
/**
 * Hooks a function of an object.
 *
 * @param obj       The object to be hooked
 * @param name      The name of the function inside the object
 * @param callback  The callback that is called when the function is accessed
 * @returns         Returns true if hook was successful
 */
function func(obj: any, name: string, callback: (original: any) => any): boolean;
```
