Standard for packed CrossCode mods. This should simplify sharing and installation of mods.

# Current Situation / Problems

Mods have to be unzip by the user and installed my manually moving them into the mods folder. "Packed mods" are not standartized (rar, 7zip, tar.gz, ...). Not every OS supports every format.

# Solution

Standard for packed mods

* zip of a mod with the extension `.ccmod` (<https://www.iana.org/assignments/media-types/application/zip>)
* requires package.json in the main directory (see unpacked mods)

## Advantage

* Simplified installation (file association)
* Standardised container
* Easy purpose recognition after downloads (`What was this for?`)

## Disadvantage

* More dependencies (if the optional part is implemented)
* System/OS dependend parts (if the optional part is implemented)
* Difficulties in testing increase (if the optional part is implemented)
* Higher programming costs

# Requirement

* Library to unzip the `ccmod` file (e.g. `zlib` - <https://nodejs.org/api/zlib.html>)
* fs (read `package.json`)
* fs (move folder)

## Optional

* Library to access windows regestry (e.g. `winreg` - <https://www.npmjs.com/package/winreg>)
* Tool packed as .app (macOS specific)
* fs (file edit)

# File handling

Possible installation methods:

## Added manually

__Required__

This can be done e.g. by adding the packed mod to a specific loation (e.g. mods folder), use drag and drop or hand over a path to the .ccmod file to the tool.

* Extraction of the zipped data
* Validate package.json
* Installation of the mod if accepted

It is OPTIONAL possible to ask the user for installation and/or show the information of the mod (based on the package.json file)

## Added by file association

__Optional__

### Setup

* a: Add file association for .ccmod automatic [OR]
* b: Ask user if file association should be added

* Ask user if a file association already exist
* Possiblility to trigger file association setup manually

### Handling

* Extraction of the zipped data
* Validate package.json
* Ask the user for installation and show the information of the mod (based on the package.json file) [REQUIRED]
* Installation of the mod if accepted

# Notes

Conflict handling and the installation of already installed mods is handled modloader/tool specific.

---

Author: streetclaw <streetclaw@c2dl.info> – Discord: 156489960021688320