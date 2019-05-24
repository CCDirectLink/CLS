# Packed Mods

Standard for packed CrossCode mods. This should simplify sharing and installation of mods.

## Current Situation / Problems

Mods have to be unzip by the user and installed my manually moving them into the mods folder. "Packed mods" are not standartized (rar, 7zip, tar.gz, ...). Not every OS supports every format.

## Solution

Standard for packed mods

* zip of a mod with the extension `.ccmod` (<https://www.iana.org/assignments/media-types/application/zip>)
* requires package.json in the main directory (see unpacked mods)

### Advantage

* Simplified installation (file association)
* Standardised container format and structure
* Easy purpose recognition after downloads (`What was this for?`)

### Disadvantage

* More dependencies (if the optional part is implemented)
* System/OS dependend parts (if the optional part is implemented)
* Difficulties in testing increase (if the optional part is implemented)
* Higher programming costs

## Requirements and dependencies

* __R0010__: [Dependency] Library to unzip the `ccmod` file (e.g. `zlib` - <https://nodejs.org/api/zlib.html>)
* __R0020__: [Dependency] Library to access windows regestry (e.g. `winreg` - <https://www.npmjs.com/package/winreg>)
* __R0030__: [Dependency] fs (read `package.json`, move folder, file edit)
* __R0040__: Tool packed as .app (macOS specific)
* __R0050__: Possibility to manually trigger the file association setup

## Installation/Interpreter methods

The installation or interpretation of packed mods in any kind is seperated into two different methods.

### Automatic or manually

This method describes the possibility to install or interpret a packed mod via interaction with a tool or by automatisms. The second method is optional so this method is the minimal requirement to work with packed mods.

__Implementaion:__ Required

__Minimal requirements & dependencies:__ R0010

__Methods:__

* Get the path of the packed mod by using drag and drop to interpret or install it.
* Search automatic or manual (in an specific location) for ccmod files. Use the path to interpret or install the mod.
* Manually input the path to a ccmod file.
* Manually input the URL to a ccmod file.

#### Interpreter

This describes how packed mods should be interpreted. This can be used to show additional information, change the mod, publish it into a databse or interprete the mod for other purposes. This method will not install the mod.

1. Extraction the zipped data and store it in memory
2. Validate package.json

What the tool is doing with the mod afterwards is highly sepecific. This also can be excracting the zipped data and store them on HDD (+R0030).

__Examples:__

```
$ cli show path/to/mod.ccmod

CrossCode mod
modtitle v 1.0.0
moddescription

Required
- CrossCode 1.0.0
- CLS 1

Dependencies
- x

[...]
```

```
$ cli show path/to/mod.ccmod -json
{ [...] }
```

* Click on [Load mod] in an tool and input the path for example.ccmod to show and edit mod information.
* Click on [Load mod] in an tool and extract the data to a specific path on HDD (+R0030).

#### Installation

This describes how packed mods should be installed by a modloader.

1. Extraction the zipped data and store it on HDD [OR] in memory
2. __Optional:__ Validate package.json
3. __Optional:__ Ask to user if the mod should be installed and show it's information (requires 2)
4. Installation of the mod if accepted

__Examples:__

* The modloader is seaching for ccmod files in a defined mods folder
* The path to a ccmod file is put into a form to add the mod to the game


### File association

The tool is setting up the system to load ccmod files with a tool by adding the tool to the file association list. This allows the user to install or interpert packed mods with a simple doubleclick on the packed mod.

__Implementaion:__ Optional

__Minimal requirements & dependencies:__ R0010, R0020, R0030, R0040, R0050

__Methods:__

* Load ccmod files with a doubleclick

#### Setup

Defines the setup of the ccmod file association.

1. Check if a program is already associated with ccmod

Possiblities if a program is associated:

* Skip the setup
* Ask the user if the tool should be associated instead (only once) and if register it if accepted
* Do not register the tool if declined

Possiblities if no program is associated:

* Set the file association automatic
* Ask the user if the tool should be associated instead (only once) and if register it if accepted
* Do not register the tool if declined

#### Interpreter

This describes how packed mods should be interpreted. This can be used to show additional information, change the mod, publish it into a databse or interprete the mod for other purposes. This method will not install the mod.

1. Extraction the zipped data and store it in memory
2. Validate package.json

__Examples:__

* Doubleclick on a ccmod file to show and edit mod information.

#### Installation

This describes how packed mods should be installed by a modloader. The mod was not added manual (e.g. by placing it in a specific mod folder). So this action requires the user to accept the installtion of the mod.

1. Extraction the zipped data and store it on HDD [OR] in memory
2. Validate package.json
3. Ask to user if the mod should be installed and show it's information
4. Installation of the mod if accepted

__Examples:__

* Doubleclick on a ccmod file to install the mod

## Notes

Conflict handling and the installation of already installed mods is handled tool specific.

---

Author: streetclaw <streetclaw@c2dl.info> – Discord: 156489960021688320