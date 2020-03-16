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

It is important to note that some features of this specification may, themselves, be implemented within mods or some equivalent.
This is fine - the implementation of this specification is then a matter of "that modloader + these mods".
This in particular applies to how CCLoader handles patch files via `simplify`, and how CCInjector requires plugins to handle dependency order & assets.

## Mod Format

A mod is in a directory (virtual or otherwise). The directory's name is irrelevant.

The directory then contains a file called "package.json" (see The package.json format).

The directory may contain a sub-directory called "assets" (see Asset Handling).

### The package.json format

The package.json format describes a mod.

It is a JSON file, the object being a standardized mod package as described here:

```ts
declare type Semver = string;
declare type SemverConstraint = string;
declare type StandardizedModPackage = {
    // String, serves as the mod's ID.
    "name": string;
    // Version of the mod.
    "version": Semver;
    /*
     * Optional - maps mod IDs or virtual dependency IDs to the versions of them required.
     * Prefixed with 'ccmod' to prevent possible Node package collision.
     * If not present, the mod has no dependencies.
     */
    "ccmodDependencies"?: { [name: string]: SemverConstraint; };
    // Optional human-readable string.
    "description"?: string;

    // Defaults to false. If true, scripts are loaded as ES6 modules.
    "module"?: boolean;
    // Defaults to false. If true, the mod uses require(), and thus may not be sandboxable.
    "usesRequire"?: boolean;

    // Optional filename of a script to run at the 'postload' stage.
    "preload"?: string;
    // Optional filename of a script to run at the 'postload' stage.
    "postload"?: string;
    // Optional filename of a script to run at the 'prestart' stage.
    "prestart"?: string;

    // Optional filename of a 'plugin' script. (See "Plugins")
    "plugin"?: string;

    // Adding additional fields may have implementation-dependent effects, including changing behaviors defined in this specification.
};
```

#### Active Mods

`window.activeMods` is an Array containing instances of the Mod interface.

It is initialized before `preload` (see "The Load Order").

```ts
// Do be aware that this is a subset of the actual interface exposed by the modloader.
interface Mod {
	get name(): string | undefined;
	get version(): string | undefined;
	get description(): string | undefined;
	get baseDirectory(): string;
}
```

#### Plugins

Plugins are a special form of script.

They are always ES6 modules, that export a default class that extends a global class called `Plugin`.

```ts
class Plugin {
	// parentMod is the mod that contains the plugin script.
	constructor(parentMod: Mod);
	// For all loading phases supported by the loader, an equivalent stub exists.
	// The loading process is blocked on the returned promise, if any.
	// These stubs are called at the respective times,
	//  as if the mod containing the plugin had additional scripts to run at those times.
	// It is undefined whether defined scripts occur before or after plugin scripts.
	preload(): Promise<void>;
	postload(): Promise<void>;
	prestart(): Promise<void>;
}
```

### Asset Handling

If the mod has a sub-directory called `assets`, then that directory is scanned for available assets.

Structurally, that sub-directory mirrors the CrossCode `assets` directory.

The scan finds some subset of the files in that sub-directory (recursively, so it includes sub-sub-directories and so on), and hooks at least CrossCode's image loading, audio loading, and AJAX requests to redirect requests for files with those names.

The subset must contain least those files with extensions: `.png`, `.json`, `.json.patch`, `.ogg`.

Other extensions may also be supported.

Regarding `.json.patch` files, these are treated specially. See Patch Files for more details.

### Patch Files

Patch files are JSON files with the `.json.patch` extension. They contain instructions to patch the relevant `.json` asset (the path of which is gained by removing the `.patch` suffix).

Patch files must still be applied on assets that have been overridden, and thus also on assets that have been created by mods.

There are two patch file formats: The old format and Patch Steps. (Patch Steps is recorded in a separate standard.)

### Patch Files (Old Format)

The format of an old-format patch file is a tree of Objects.

(This implies that the format of an old-format patch file is always an Object.)

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
2. `postload`. This is executed after game.compiled.js is executed and before game onload (i.e. before `ig.modules["dom.ready"].loaded` is true).
3. `prestart`. This is executed in a hook wrapping `ig.main`, before `ig.main` itself.

## Conformance Tests

A pair of mods implementing a conformance test is available at: https://github.com/ac2pic/CLS-20kdc-Code/blob/master/mods/

These mods are CCStandardizedModsConformanceTest and CCZZStandardizedModsConformanceTestDependency.

---

```
Author: 20kdc
```
