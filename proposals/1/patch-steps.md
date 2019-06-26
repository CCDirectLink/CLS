# Patch Steps: A Standardized Extended Patch Format

## Description Of Issue

The current CCLoader patch format appears to be the closest for a de-facto standard for JSON patching there is.

It's effective on objects, but has flaws elsewhere.

It cannot handle arrays effectively, and if CrossCode game objects are being copied from other files, they end up copied verbatim.

The format as-is can't be extended, but always involves a tree of objects, so a transparent replacement could be introduced without changing the formats that use patches.

This would require a new format, though.

## Solution

A new format based on an Array root with a series of steps allows for three workflows:

1. Automatic generation of a patch as before, but with better (i.e. less likely to break) handling of addition of elements to arrays and such

2. Manual patch creation to allow targetted copy-object-with-modification scenarios using IMPORT. (Particularly useful for player AnimationSheets)

3. Manual patch creation to separate out the individual added objects into separate files using IMPORT.

### Advantages

- Format is in the "Steps" style that manual editing software may expect.

- Format can be extended in future if issues are encountered.

- Format follows a sequenced order that may be useful given the aforementioned expandability requirements. 

### Disadvantages

- Due to being in the "steps" style, the format is sequence-based, which makes removing sections require keeping track of the current position.

- The format will still require hand-editing to import objects due to there being no reliable detection mechanism for these cases.

- Making a modified object requires a blank object and a patch that first imports the source and then applies the modifications.

- The format is more complicated than the previous one.

## The Patch Format

A Patch is an array of objects, known as PatchSteps.

The "step" structure is used to allow existing editing tools that understand it to incorporate it with ease.

(When defining a Step, try to ensure it can be defined in the form of a, well, Step, if possible.)

And furthermore, as such, the "type" field describes the specific step type as with other types of Step.

Further details are type-specific.

When executing a Patch, some state is maintained.

There is the Current Value, and a stack which backs up the Current Value to return to parent values.

The final target of the Current Value doesn't matter - it's merely used as an editing cursor.

### ENTER

`ENTER` steps push the Current Value onto the stack, then replace the Current Value with a property of it, thus 'entering' that property.

They have a single additional string property, "index".

This property is a property name of the current object or array.

JavaScript rules mean that "0" and 0 are equivalent, and thus this can access array elements.

### EXIT

`EXIT` steps pop a new Current Value off the stack, replacing it.

This allows going back to a parent value to modify other sub-values.

### SET_KEY

`SET_KEY` steps have two additional properties:

"index" (containing the property name)

And optionally "content" (Arbitrary content value)

The property in the Current Value is set to a copy of the content.

If content is not provided, the property is deleted.

Note that SET_KEY with a non-numeric index is valid on Arrays, but will probably end badly.

### REMOVE_ARRAY_ELEMENT

`REMOVE_ARRAY_ELEMENT` steps have a numeric property "index" (an Array.splice index).

It removes the element at that index from the Current Value.

### ADD_ARRAY_ELEMENT

`ADD_ARRAY_ELEMENT` steps have an optional numeric property "index" (again, an Array.splice index)...

... and an arbitrary value "content".

A copy of the content is inserted into the Current Value at the index.

If the index is not present, the content is appended.

### IMPORT

`IMPORT` steps have a string property "src" (a `assets/`-less filename such as `"data/database.json"`), optionally "index" (another string property) and optionally "path" (an array of keys).

JSON is loaded from the file specified with "src". The loaded JSON value (be it an array or object) is called the Loaded Value.

If "path" is present, it's iterated over. For each entry in "path", the Loaded Value is indexed by that item, and the result of that indexing replaces the Loaded Value.

As such, a "path" of `["a", 0]` or `["a", "0"]` on the initial Loaded Value of `{"a": ["hi"]}` results in the Loaded Value becoming the `"hi"` string.

Finally, if "index" is present, the property of the Current Value targetted by "index" is then set to the Loaded Value.

If it is not present, each item of the Loaded Value is appended (if the Loaded Value is an array) or the keys are copied in (if the Loaded Value is an object).

The primary use of this is for cleanup of manual editing workflows, but it can also be useful to cherry-pick objects to avoid copyright issues.

Any implementation that supports full file replacement/addition should allow the JSON to come from an overridden/added file - such files are virtually "added to the game" and should be treated as such.

If an imported JSON file should be patched before import, however, is at the discretion of the implementation, as a particular issue that could come up here is unexpected circular dependencies.

### INCLUDE

`INCLUDE` steps have a string property "src", a path inside the mod's directory. (The value "package.json" would be erroneous due to it being of the wrong format, but would point at the intended file.)

This path points at a JSON file.
This JSON is treated as a Patch Steps patch and executed on the Current Value.
Note, however, that this occurs in a separate Patch Steps interpreter, so the Current Value stack is not shared.

## Reference Implementations

A reference CommonJS module implementation is available at:

https://github.com/20kdc/CLS-20kdc-Code/blob/master/tools/patch-steps-lib.js

A more integrated Impact module implementation of the patcher (but not the differ) is available at:

https://github.com/20kdc/CLS-20kdc-Code/tree/master/modules/impact/feature/patches

---

```
Author: 20kdc (Discord: 234666977765883904)
```
