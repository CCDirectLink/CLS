# Standardized Mod Format (DRAFT!)

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

A mod is in a directory. The directory's name is irrelevant.

The directory then contains a file called "package.json" (see The package.json format).

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

 "preload": <Optional filename of a script>,
 "postload": <Optional filename of a script>,
 "prestart": <Optional filename of a script>,
 "main": <Optional filename of a script. For example, "mod.js" loads the file "mod.js" within the mod folder>,

 "usesRequire": <Optional boolean - if true, this mod uses the 'require' function. Defaults to true>,
}
```

### Asset Handling

Mods are scanned for available assets.

Assets are defined as those files with extensions: `.png`, `.json`, `.json.patch`, `.ogg`.
Other extensions may be added later, but note that adding `.js` may cause conflicts due to files added unnecessarily.

Regarding `.json.patch` files, these are treated specially. See Patch Files for more details.

The mod's directory tree structurally mirrors the `assets` directory of CrossCode.

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

### The Load Order

There are a few 'phases' to mod loading.
Each 'phase' can have scripts.

Within a given 'phase', mods must be loaded in dependency order.
The behavior for a circular dependency situation is undefined.

The 'phases' are:

1. `preload`. This is executed before game.compiled.js is executed.
2. `postload`. This is executed after game.compiled.js is executed, but before `window.startCrossCode`.
3. `prestart` (Name subject to change) This occurs before `ig.main` in a hook wrapping `ig.main`, which ensures the whole game has loaded and effectively gives the code free reign over Cubic Impact structures.
4. `main`. This is executed at any time after the game has completely loaded and is running, i.e. `ig.ready` is true, `ig.game` exists, etc.

---

```
Author: 20kdc
```

