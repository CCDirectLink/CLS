# Mod modules

## Current situation

If mods want to import node modules they are forced to either add them to the games node_modules folder or do a dynamic lookup at runtime. This causes a lot of ugly code or a hard to install mod.

## Solutions

### Module resolution per mod

It is possible for a modloader to hook require add a node_module search path for each mod by inspecting the `document.currentScript` property. This can allow for mods to have their own separate modules.

#### Advantages

* Does not cause interference with other mods

#### Disadvantages

* Can cause a lot of bloat because each mod needs to come with all dependencies
* Does not work with delayed requires because `document.currentScript` will loose it's value.

### Global modules

A modloader can add all node_module folders of all mods to the require path.

#### Advantages

* Mods can access the dependencies of other mods
* It will always find a installed module
* If implemented as a global folder it can reduce bloat

#### Disadvantages

* Mods can cause interference if the same module with a different version is installed

### Per mod with fallback

A modloader can used the method described in "Module resolution per mod" until `document.currentScript` looses it's value and then fall back to the method of "Global modules".

#### Advantages

* Always works
* Causes less interference with other mods

#### Disadvantages

* Inconsistent behavior
* Can cause a lot of bloat because each mod needs to come with all dependencies
* Mods can cause interference if the same module with a different version is installed and not required immediately


## Implementation

The module resolution algorithm works by looking in all path defined by [`global.module.paths`](https://nodejs.org/api/modules.html#modules_module_paths). A modloader can extend this array to add new search paths.

A modloader can determine the current module by bootstrap code around the mod or by using the [`document.currentScript`](https://developer.mozilla.org/en-US/docs/Web/API/Document/currentScript) property.

---

Author: 2767mr (Discord: 224155607278551040)