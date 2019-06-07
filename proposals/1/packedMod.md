# Packed Mods

Standard for packed CrossCode mods. This should simplify sharing and installation of mods.

## Current Situation / Problems

Mods have to be unzip by the user and installed by manually moving them into the mods folder. "Packed mods" are not standardized (rar, 7zip, tar.gz, ...). Not every OS supports every format.

## Solution

Standard for packed mods

* zip of a mod with the extension `.ccmod` (<https://www.iana.org/assignments/media-types/application/zip>)
* requires package.json in the main directory (see unpacked mods)

### Advantage

* Simplified installation (file association)
* Standardized container format and structure
* Easy purpose recognition after downloads (`What was this for?`)

### Disadvantage

* More dependencies (if the optional part is implemented)
* System/OS dependent parts (if the optional part is implemented)
* Difficulties in testing increase (if the optional part is implemented)
* Higher programming costs

## Requirements and dependencies

* __R0010__: [Dependency] Library to unzip the `ccmod` file (e.g. `zlib` - <https://nodejs.org/api/zlib.html>)
* __R0020__: [Dependency] Library to access windows registry (e.g. `winreg` - <https://www.npmjs.com/package/winreg>)
* __R0030__: [Dependency] fs (read `package.json`, move folder, file edit)
* __R0040__: Tool packed as .app (macOS specific)
* __R0050__: Possibility to manually trigger the file association setup
* __R0060__: [Dependency] Hash-Library (`sha256`)

## Installation/Interpreter methods

The installation or interpretation of packed mods in any kind is separated into two different methods.

### Automatic or manually

This method describes the possibility to install or interpret a packed mod via interaction with a tool or by automatisms. The second method is optional so this method is the minimal requirement to work with packed mods.

__Implementation:__ Required

__Minimal requirements & dependencies:__ R0010

__Methods:__

* Get the path of the packed mod by using drag and drop to interpret or install it.
* Search automatic or manual (in an specific location) for ccmod files. Use the path to interpret or install the mod.
* Manually input the path to a ccmod file.
* Manually input the URL to a ccmod file.

#### Any Interpreter tool

This describes how packed mods should be interpreted. This can be used for tools that are showing additional information, change the mod, publish it into a database or interpreted the mod for other purposes. This is valid for every action and tool that is not installing the mod. Tools that are interpreter and installer at the same time have to combine both parts. The part `Installation via Modloader` is only required after triggering the installation. The sections 1 and 2 are already done and don't need to be repeated.

1. Extraction the zipped data and store it in memory
2. Validate package.json

What the tool is doing with the mod afterwards is highly specific. This also can be extracting the zipped data and store them on HDD (+R0030).

##### Examples

```
$ cli show path/to/mod.ccmod

CrossCode mod
mod-title v 1.0.0
mod-description

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

#### Installation via Modloader

This describes how packed mods should be installed by a modloader.

1. Extraction the zipped data and store it on HDD [OR] in memory
2. __Optional:__ Validate package.json (+R0030)
3. __Optional:__ Ask to user if the mod should be installed and show it's information (requires 2)
4. Installation of the mod if accepted
5. __Optional:__ Inform the user if the installation failed

The installation may fail if the mod is not compatible with the game version, the CLS version or doesn't implement required components or contains required data.

##### Examples

* The modloader is searching for ccmod files in a defined mods folder
* The path to a ccmod file is put into a form to add the mod to the game


### File association

The tool is setting up the system to load ccmod files with a tool by adding the tool to the file association list. This allows the user to install or interpert packed mods with a simple double-click on the packed mod.

__Implementation:__ Optional

__Minimal requirements & dependencies:__ R0010, R0020, R0030, R0040, R0050, R0060

__Methods:__

* Load ccmod files with a double-click

#### Setup

Defines the setup of the ccmod file association.

1. Check if a program is already associated with ccmod

Possibilities if a program is associated:

* Skip the setup
* Ask the user if the tool should be associated instead (only once) and if register it if accepted
* Do not register the tool if declined

Possibilities if no program is associated:

* Set the file association automatic
* Ask the user if the tool should be associated instead (only once) and if register it if accepted
* Do not register the tool if declined

#### Any Interpreter tool

This describes how packed mods should be interpreted. This can be used for tools that are showing additional information, change the mod, publish it into a database or interpreted the mod for other purposes. This is valid for every action and tool that is not installing the mod. Tools that are interpreter and installer at the same time have to combine both parts. The part `Installation via Modloader` is only required after triggering the installation. The sections 1 and 2 are already done and don't need to be repeated.

1. Extraction the zipped data and store it in memory
2. Validate package.json

##### Examples

* Double-click on a ccmod file to show and edit mod information.

#### Installation via Modloader

This describes how packed mods should be installed by a modloader. The mod was not added manual (e.g. by placing it in a specific mod folder). So this action requires the user to accept the installation of the mod.

1. Extraction the zipped data and store it on HDD [OR] in memory
2. Validate package.json
3. Ask to user if the mod should be installed and show it's information
4. Installation of the mod if accepted
5. __Optional:__ Inform the user if the installation failed

The installation may fail if the mod is not compatible with the game version, the CLS version or dosn't implement required components or contains required data.

##### Examples

* Double-click on a ccmod file to install the mod

## Mod Information

This section describes what information the user should receive when prompting for the installation.

### Minimal Content

* Name

  Gives the user the ability to identify which mod will be installed.

* Version (if available)

  Gives the user the ability to check if the right version will be installed.

* Description (if available)

  Should describes the function of the mod.

* Author & Contact information (if available)

  Gives the user the ability to check for the right author. Useful if there exists mods with the same name.

* Mod Identifier (`MI`) [AND] Mod file Path (`MP`)

  Gives the user the ability to check if the right mod is installed. Useful for testing mods without changing the version or other characteristics to keep them apart.

* Game Identifier (`GI`) [AND/OR] Path of the modded game instance (`GP`)

  It is possible to have multiple modded games. Makes it possible to check that the mod is installed for the correct instance. It is recommended to also show the Game Identifier (`GI`).

### Game Identifier (GI)

A abbreviation for the game. This id should be unique for the device.

1. Requires the path of the game (including the executable) as string (`GP`)
2. Hash the string with `sha256`
3. The full hash is the `long id` the last 8 characters are the `short id`

* Any path must be absolute
* Do not use any environment variable or path abbreviation

This prevents ambiguous paths. __Do not use the GI-Hash to identify game instances across multiple devices. Use it only to differentiate multiple Game instances.__ The ID can for example be used as key for a Database of `current known game instances` or similar use cases.

__Game path Examples:__

* macOS: `/Users/myuser/Library/Application Support/Steam/steamapps/common/CrossCode/CrossCode.app` (not `~/Library/...`)
* Windows: `C:\Program Files (x86)\Steam\steamapps\common\CrossCode\CrossCode.exe` (not `%programfiles(x86)%\Steam\...`)
* Linux: `/home/myuser/.steam/steam/SteamApps/common/CrossCode/CrossCode` (not `~/.steam/...`)

### Mod Identifier (MI)

Mods can be moved and have no fixed location like the game. Therefor the `sha256`-Hash (see: `sha256sum`) of the mod file (`.ccmod`) will be used instead to identify the mod. Unpacked mods have to use the absolute path (without the use of environment variable or path abbreviation) of the `package.json` instead to provide the same amount of information as packed mods (`MP`). __Do not use the MI-Hash for unpacked or packed mods to identify mods across multiple devices. Use it only to differentiate Mods.__ The ID can for example be used as key for a `current loaded`-Database or similar use cases.

If packed:

1. Hash the .ccmod file with `sha256`
2. The full hash is the `long id` the last 8 characters are the `short id`

If unpacked:

1. Requires the path of the mod (including `package.json`) as string (`MP`)
2. Hash the string with `sha256`
3. The full hash is the `long id` the last 8 characters are the `short id`

__Examples – Mod Installation:__

```
$ cli add mod.ccmod
CrossCode Mod

ModName - v 1.0.0
mod description

CCDirectLinkAuthor <info@c2dl.info>

Mod: c92b4510 (/User/myuser/documents/example.ccmod)
Installed for: cb7dcbc5 (/Users/myuser/Library/Application Support/Steam/steamapps/common/CrossCode/CrossCode.app)

Install mod? (y/n):
```

__Comparison – Unpacked:__

```
C:> cli add mods\example
CrossCode Mod

ModName - v 1.0.0
mod description

CCDirectLinkAuthor <info@c2dl.info>

Mod: d38bdb9f (C:\Program Files (x86)\Steam\steamapps\common\CrossCode\mods\example\package.json)
Installed for: b1dfa4d4 (C:\Program Files (x86)\Steam\steamapps\common\CrossCode\CrossCode.exe)

Install mod? (y/n):
```

## Notes

Conflict handling and the installation of already installed mods is handled tool specific.

---

Author: streetclaw <streetclaw@c2dl.info> – Discord: 156489960021688320
