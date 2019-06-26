# Json description standard

It is recommended to describe JSON-Structures __additional__ in a standardised and machine readable format (besides the examples).

## Current Situation / Problems

The documentation only describes the used JSON-Structures in various and not machine readable ways:
- <https://github.com/CCDirectLink/CLS/blob/master/proposals/1/standardized-mod-format.md#the-packagejson-format>
- <https://github.com/CCDirectLink/CCLoader/wiki/Mod-Framework#packagejson>

## Solution

There is already a standard for describing JSON-structures called `JSON Schema` (Reference: <http://json-schema.org>, <http://json-schema.org/understanding-json-schema/> and <http://json-schema.org/understanding-json-schema/UnderstandingJSONSchema.pdf>).

### Advantage

- Machine readable
- Can directly used to validate the JSON data

### Disadvantage

- More work by defining the JSON Schema files

## Requirements and dependencies

__R0010__: [__Optional__ Dependency] JSON Schema validator (e.g. <https://github.com/epoberezkin/ajv>)

---

Author: streetclaw <streetclaw@c2dl.info> – Discord: 156489960021688320