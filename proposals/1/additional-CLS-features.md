# CLS Version handling and additional features

## Description Of Issue

The CLS Version contains only a `major` version. This version will be incremented if a breaking change will be defined in the CLS standard. Any non breaking feature is added to the latest version.

Implementations should not have to contain all existing features. Therfore different implementation may vary in feature coverage. But mods may require specific features.

## Solution

Mods are using feature flags to check if a specific feature is required or available.

## Advantages

- Reduced loader implementaions possible
- Implementing a loader is easier if not every feature has to be implemented
- All new features are optional. Makes adding new features easier

## Disadvantages

- Adds more complexity to the loader

## General Implementation Details

### Check Features

- Mods can check at runtime if a specific feature is available

```
(TODO)
```

- Mods can define required features in the package.json. The mod will not be loaded if the feature is not available.

JSON Schema:

```
{
	"type": "object",
  	"properties": {
  		"requiredFeatures": {
  			"type": "array"
			"items": {
		    	"type": "string"
			}
  		}
  	},
	"additionalProperties": true
}
```

Type:

```
declare type StandardizedModPackageExtended extends StandardizedModPackage = {
	"requiredFeatures"?: array<string>;
}
```

Example:

```
{
	...
	"requiredFeatures": [
		"a", "b", "c"
	]
}
```

### Feature Flags

Feature flags are provided in the form of a string.

The only predefined feature flag is `core`. All features labeled as `core` are required and have to be implemented for the specific CLS version. `core` features can differ based on the CLS Version. Adding or removing new features as part of `core` is always a breaking change and result in a version incrementation.

It is not necessary for mods to add the feature flag `core` to the package.json file or check it at runtime.

- The loader can e.g. show a warning if a mod requires a feature that is not available.
- The loader can e.g. show a warning if a deprecated feature is used.
- Mods can e.g. show a warning if a feature is not available.

### CLS Version Example

1. Added feature is `core` feature in newer versions

| CLS version `n`  | CLS version `n+1` |
| ---------------- | ----------------- |
| Feature A `core` | Feature A `core`  |
| Feature B `core` | Feature B `core`  |
| Feature C `c`    | Feature C `core`  |
| Feature D `d`    | Feature D `d`     |


2. Feature removed from `core` in newer versions (e.g. deprecated features)

| CLS version `n`  | CLS version `n+1`        |
| ---------------- | ------------------------ |
| Feature A `core` | Feature A `core`         |
| Feature B `core` | Feature B `b-deprecated` |
| Feature C `core` | Feature C `core`         |
| Feature D `d`    | Feature D `d`            |

Now Feature B can removed at any time from the version.

---

Author: streetclaw <streetclaw@c2dl.info> – Discord: 156489960021688320
