# Standardized Mod Format

## Description Of Issue

Lack of a clear specification between modloaders on a commonly supported mod structure & format (such as package.json details).
Right now the featureset that's common between CCLoader and CCInjector isn't clear without reading their code.
This would define a reliable featureset between these modloaders and any other modloaders that enter the scene.

## Solution

A specification which exactly defines specific terms, and does not define legacy formats from the previous era.
(Note in regards to this: Defining legacy formats from the previous era would either require support for them or waste space or be completely pointless)

### Advantages

- Modloaders and modders know what is supported everywhere.

- Less likelihood to use outdated mechanisms from the old era

- APIs defined in other specifications may gain reference implementations using this specification.

### Disadvantages

- No specific system in place for per-modloader extensions other than extra unspecified fields (for optional dependency) or Virtual Dependencies (which cause non-compliance and make the mod non-portable).

### Regarding Implementation

It is important to note that some features of this specification may, themselves, be implemented within mods.
This is fine - the implementation of this specification is then a matter of "that modloader + these mods".
This in particular applies to how CCLoader handles patch files via `simplify`.

## Mod Format

A mod is in a directory (virtual or otherwise). The directory's name is irrelevant.

The directory then contains a file called "package.json" (see The package.json format).

The directory may contain a sub-directory called "assets" (see Asset Handling).

### The package.json format

The package.json format describes a mod.

It is a JSON file, with an object containing the following keys ('Optional' allows the key to be missing):

```
{
 "name": <String, serves as the mod's ID>,
 "version": <Semver-compliant version string>,
 "ccmodDependencies": <
  Optional Object - keys are mod IDs, values are semver-compliant version constraints
  This is used between different mods to establish load order & required other mods.
  Note that some entries in here may not actually be mods - see Virtual Dependencies for further information.
 >,
 "dependencies": <
  Optional Object - due to compatibility, this is a bit awkward
  If ccmodDependencies is provided, keys are Node modules, values are semver-compliant version constraints
  If ccmodDependencies is NOT provided, this key is effectively renamed to it - meaning keys are mod IDs, not Node modules.
 >,

 "description": <Optional human-readable string>,

 "postload": <Optional filename of a script>,
 "module": <Optional boolean - if true, all scripts are loaded as ES6 modules, otherwise they aren't. Defaults to false>,

 "usesRequire": <Optional boolean - if true, "require" is being used by the mod, otherwise it isn't. Defaults to true>,
}
```

Note that additional unspecified fields may also be present.

These fields may have implementation-dependent effects, including changing behaviors defined in this specification.

### Asset Handling

If the mod has a sub-directory called `assets`, then that directory is scanned for available assets.

Structurally, that sub-directory mirrors the CrossCode `assets` directory.

The scan finds some subset of the files in that sub-directory (recursively, so it includes sub-sub-directories and so on), and hooks at least CrossCode's image loading, audio loading, and AJAX requests to redirect requests for files with those names.

The subset must contain least those files with extensions: `.png`, `.json`, `.json.patch`, `.ogg`.

Other extensions may also be supported.

Regarding `.json.patch` files, these are treated specially. See Patch Files for more details.

### Patch Files

Patch files are JSON files with the `.json.patch` extension. They contain instructions to patch the relevant `.json` asset (the path of which is gained by removing the `.patch` suffix).

The format of a patch file is a tree of Objects.

Running a patch file follows a recursive routine with the form (current patch object, current target object).
(The current target object is treated the same regardless of actual type.)

The routine is:

For each key in the current patch object...

1. If the current target's value for the key is undefined, set it to the value for the key in the patch object.
2. Else, if the patch object's value for the key is an Object (and not a subclass, in case of boxed type coercion), apply the algorithm for (the current target's value for the key, the patch object's value for the key).
3. Else, set the current target's value for the key to the value for the key in the patch object.

### Virtual Dependencies

"Virtual Dependencies" are dependencies set on mods that do not actually exist, but are part of the modloader.

Mods that use modloader-specific virtual dependencies, such as "ccloader", are not compliant.
There is a virtual dependency that is standardized and thus compliant, however: "crosscode". This specifies a version constraint on the CrossCode version in use.

Some virtual dependencies may be defined by APIs that have reference mod implementations.
In this case, the virtual dependency is to be treated as a mod integrated into the modloader and is not non-compliant.

### The Load Order

There are a few 'phases' to mod loading.
Each 'phase' can have scripts.

Note that loading *and execution* of the scripts for a mod in a given phase are both part of that mod's loading for that phase.

Within a given 'phase', mods must be loaded in 'dependency order' - the dependencies of a mod must be loaded and run before that mod.
The behavior for a circular dependency situation is undefined.

The 'phases' are:

1. `preload`. This is executed before game.compiled.js is executed.
    (There is no attribute for this phase - it exists for reference)
2. `postload`. This is executed after game.compiled.js is executed and before game onload (i.e. before `ig.modules["dom.ready"].loaded` is true).
3. `prestart`. This is executed in a hook wrapping `ig.main`, before `ig.main` itself.
   (There is no attribute for this phase - it exists for reference)
4. `main`. This is executed at any time after the game has completely loaded and is running, i.e. `ig.ready` is true, `ig.game` exists, etc.
   (There is no attribute for this phase - it exists for reference)

## Conformance Tests

A pair of mods implementing a conformance test is available at: https://github.com/20kdc/CLS-20kdc-Code/blob/master/mods/

These mods are CCStandardizedModsConformanceTest and CCZZStandardizedModsConformanceTestDependency.

---

```
Author: 20kdc
```

