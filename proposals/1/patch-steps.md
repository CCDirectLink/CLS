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

## File Paths

File paths in Patch Steps are checked to see if they are valid absolute URLs.

If they are, then the result depends on the protocol.

Importantly, only the protocol and the pathname in the URL matter - other details are ignored.

For the `mod:` protocol, the pathname is a file relative to the mod's directory.
(`mod:package.json` would be the mod's package.json file)

For the `game:` protocol, the pathname is a file in the game's "virtual filesystem" - this is the filesystem after asset overrides and patches have been applied.
(`data/database.json` would be the Database)

Other protocols aren't defined.

If the path is not an absolute URL, the string as a whole is treated as the pathname, and is given a contextual default protocol.
This default protocol is specified in the relevant step's details.

## Step Formats

```ts
// In practice, these are functionally identical,
//  and weak JavaScript typing causes this to be readily apparent.
declare type JSONIndex = string | number;

declare type PatchStepsPatch = PatchStep[];

declare type PatchStepObject = {
	[name: string]: string; 
};

declare type PatchStepObjectMatch = {
	[name: string]: RegExp;
};

declare type PatchStep =
	PatchStepEnter |
	PatchStepExit |
	PatchStepSetKey |
	PatchStepInitKey |
	PatchStepRemoveArrayElement |
	PatchStepAddArrayElement |
	PatchStepImport |
	PatchStepInclude |
	PatchStepForIn |
	PatchStepCopy |
	PatchStepPaste
;

/**
 * Abstract machine state for a Patch Steps interpreter;
 *  not a formal part of the format, merely a guideline.
 * Do not confuse with internal structures used by any specific implementation.
 */
declare class PatchStepMachineState {
	/*
	 * Decodes a path or URL according to the 'File Paths' chapter and retrieves that object.
	 * The contextual conversion is specified as defaultProtocol.
	 */
	loader: (defaultProtocol: string, path: string) => Promise<object>;
	// The current value being operated on.
	currentValue: object;
	/* Used by COPY/PASTE to store and 
	 * retrieve saved values based upon an
	 * alias.
	 */  
	cloneStack: Map;
	// Used by ENTER/EXIT to place and retrieve parent values.
	parentStack: object[];
}

/*
 * The ENTER patch step, should index not be an array, pushes the Current Value onto the parent stack,
 *  and then sets the Current Value to currentValue[index].
 * However, if the "index" is an array, this instead acts as multiple ENTER patch steps,
 *  one with each element of the array replacing the index.
 */
declare type PatchStepEnter = {
	"type": "ENTER";
	"index": JSONIndex[] | JSONIndex;
};

/*
 * The EXIT patch step is the inverse of the ENTER patch step.
 * It sets the Current Value to a value popped off the parent stack.
 * Like ENTER, it can be applied multiple times.
 * If 'count' is present, EXIT will be applied that many times.
 */
declare type PatchStepExit = {
	"type": "EXIT";
	"count"?: number;
};

/*
 * The SET_KEY patch step sets a property of the Current Value.
 * If "content" is not provided, then the property is deleted.
 * This is in effect:
 * currentValue[index] = content;
 *  and
 * delete currentValue[index];
 */
declare type PatchStepSetKey = {
	"type": "SET_KEY";
	"index": JSONIndex;
	"content"?: any;
};

/*
 * The INIT_KEY patch step sets a property of the Current Value if it isn't present (index in currentValue).
 * Since this would be completely useless otherwise, content must be present.
 */
declare type PatchStepInitKey = {
	"type": "INIT_KEY";
	"index": JSONIndex;
	"content": any;
};

/*
 * The REMOVE_ARRAY_ELEMENT patch step splices out a single element from the Current Value.
 * The Current Value must be an array.
 * The index used is treated as an Array.splice index.
 * The element is discarded.
 */
declare type PatchStepRemoveArrayElement = {
	"type": "REMOVE_ARRAY_ELEMENT";
	"index": number;
};

/*
 * The ADD_ARRAY_ELEMENT patch step splices in a single element into the Current Value.
 * Index may not be provided - in this case, it pushes the element onto the end.
 * The element is discarded.
 */
declare type PatchStepAddArrayElement = {
	"type": "ADD_ARRAY_ELEMENT";
	"index"?: number;
	"content": any;
};

/*
 * The IMPORT patch step loads a JSON file, which we'll refer to as "obj".
 * For each element in path 'idx', if present, the following is performed:
 *  obj = obj[idx];
 * Finally:
 * If index is present, this finishes with effectively 'SET_KEY': currentValue[index] = obj;
 * Otherwise, if index is not present, a merge occurs:
 *  If obj is an Object, all properties in obj are copied into the Current Value.
 *  If obj is an Array, all elements in obj are pushed onto the Current Value.
 *  Otherwise, an error occurs (the value is unmergable).
 */
declare type PatchStepImport = {
	"type": "IMPORT";
	// File Path, default protocol "game:"
	"src": string;
	"path"?: JSONIndex[];
	"index"?: JSONIndex;
};

/*
 * The INCLUDE patch step loads a JSON file containing a patch,
 *  and executes it on the Current Value in a new interpreter.
 * The current interpreter, and thus the loading, waits for it to complete.
 */
declare type PatchStepInclude = {
	"type": "INCLUDE";
	// File Path, default protocol "mod:"
	"src": string;
};


/**
* The FOR_IN patch step takes a value entry from the values property,
* goes through the body statements,
* clones and replaces keyword match with the current value entry inside the statement object,
* and then executes the current statement.
* Note: If the keyword is a string, then the values must be an array of strings.
*       If the keyword is a PatchStepObjectMatch, then the values must be an array of PatchStepObjects.
**/
declare type PatchStepForIn = {
	"type": "FOR_IN";
	"values": string[] | PatchStepObject[];
	// interpreted as a Regular Expression
	"keyword": string | PatchStepObjectMatch;
	"body": PatchStepsPatch;
};

/*
* The COPY patch step takes the Current Value stored in the Interpreter 
* and maps it to the provided alias key.
*/
declare type PatchStepCopy = {
	"type": "COPY";
	"alias": string;
};


/**
* The PASTE patch step retrieves the stored value based upon 
* the provided alias and merges it with the Current Value.
*
* If the Current Value is an Array, then the index can be a number or blank.
* The stored value will be pushed to the end if the index is blank,
* or put into the positions specified index.
*
* If the Current Value is an Object, then the index can be a number or a string.
* The stored value will be set to the property named after the index provided.
*
* If the Current Value is none of these, then it will throw an error.
*/
declare type PatchStepPaste = {
	"type": "PASTE";
	"alias": string;
	"index"?: JSONIndex;
};
```



## Reference Implementations

A reference ES6 module implementation is available at:

https://github.com/20kdc/CLS-20kdc-Code/blob/master/tools/patch-steps-es6.js

A reference CommonJS module implementation is available at:

https://github.com/20kdc/CLS-20kdc-Code/blob/master/tools/patch-steps-lib.js

---

```
Author: 20kdc (Discord: 234666977765883904) 
        Emileyah (Discord: 208763015657553921)
```
