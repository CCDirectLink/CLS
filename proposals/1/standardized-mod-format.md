# Standardized Mod Format (DRAFT!)

## Description Of Issue

Lack of a clear specification between modloaders on a common supported format.

## Solution

A specification which exactly defines specific terms, and does not define legacy formats from the previous era.

### Advantages

- Modloaders and modders know what is supported everywhere.

- Less likelihood to use outdated mechanisms from the old era

- APIs defined in other specifications may gain reference implementations using this specification - these can then become the only implementation of those APIs, preventing fragmentation.

### Disadvantages

- Per-modloader extensions not covered, theoretically limiting features for more portable mods.

### Regarding Implementation

It is important to note that some features of this specification may, themselves, be implemented within mods.

This is fine - the implementation of this specification is then a matter of "that modloader + these mods".

This in particular applies to how CCLoader handles patch files via `simplify`.

## Mod Format

A mod is in a directory. The directory's name is the ID of the mod.

The directory then contains a file called "package.json".

### The package.json format

The package.json format describes a mod.

It has the following elements:

```
{
 "version": <Semver-compliant version string>,
 "ccmodDependencies": <Optional Object - keys are mod IDs, values are semver-compliant version constraints - This is used between different mods>,
 "dependencies": <
  Optional Object - due to compatibility, this is a bit awkward
  If ccmodDependencies is provided, keys are Node modules, values are semver-compliant version constraints
  If ccmodDependencies is NOT provided, this replaces that, and has no function by itself
 >,

 "name": <Optional human-readable string - otherwise, the directory name is used>,
 "description": <Optional human-readable string>,

 "main": <Optional filename of a script. "mod.js" loads the file "mod.js" within the mod folder>,
 "preload": <Optional filename of a script>,
 "postload": <Optional filename of a script>,
 "usesRequire": <Optional boolean - if true, this mod uses the 'require' function. Defaults to true>,

 "assets": <
  Optional array of strings, such as "media/icons.png".
  An asset ending in ".json.patch" is a patch file - see Patch Files.
  If not provided, assets may be automatically guessed or completely ignored.
  (NOTE! THIS PART CONFLICTS WITH CURRENT BEHAVIOR - CCLOADER RIGHT NOW WILL ALWAYS GUESS ASSETS, EVEN IF PROVIDED.)
 >
}
```

### Patch Files

Patch files have the `.json.patch` extension, and patch the relevant `.json` asset.

Patch files are a tree of objects.

Running a patch file follows a recursive routine.

This routine has the form (current patch object, current target object).

The current target object is treated the same regardless of actual type.

The routine is:

For each key in the current patch object...

1. If the current target's value for the key is undefined, set it to the value for the key in the patch object.
2. Else, if the patch object's value for the key is an Object (and not a subclass - think boxed type coercion), apply the algorithm for (the current target's value for the key, the patch object's value for the key).
3. Else, set the current target's value for the key to the value for the key in the patch object.

Note from the future: If the `patch-steps` proposal becomes part of CLS, it then also becomes a valid patch format.

### Virtual Dependencies

There are two "virtual dependencies" known. These do not have mod IDs.

"ccloader": Forces a dependency on CCLoader. A mod that uses this is not compliant.

"crosscode": Used to specify a specific CrossCode version constraint.

### The Load Order

There are a few 'phases' to mod loading.

Each 'phase' can have scripts.

Within a given 'phase', mods must be loaded in dependency order.

The behavior for a circular dependency situation is undefined.

The 'phases' are:

1. `preload`. This is executed before game.compiled.js is executed.
2. `postload`. This is executed after game.compiled.js is executed, but before window.startCrossCode.
3. `ThisIsAVeryTemporaryNameThatWeNeedToAgreeOn` (Name subject to change. Unspecified.). This occurs directly before `ig.main`, which ensures the whole game has loaded and effectively gives the code free reign over Cubic Impact structures.
4. `main`. This is executed at any time after the game has completely loaded and is running, i.e. `ig.ready` is true, `ig.game` exists, etc.

---

No implementations are registered here at this time because this spec is still way too far into flux for anything to be settled as "the" implementation.

```
Author: 20kdc
```

