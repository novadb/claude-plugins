---
name: get-object-type
description: "Fetch an object type with all resolved attribute definitions (via objecttype_describe)."
user-invocable: false
allowed-tools: objecttype_describe, objecttype_query
---

# Get Object Type

Fetch an object type with its attributes, forms, and metadata already resolved. ONLY for reading object types — NOT for creating, updating, or adding attributes.

## Scope

**This skill ONLY handles:** Fetching an object type with the resolved attribute and form information.

**For updating type name/description** → use `update-object-type`
**For adding new attributes to a type** → use `add-attribute-to-type`
**Don't have a type ID yet?** → use `objecttype_query` to search by name or API identifier.

## Tool

`objecttype_describe` — the C# MCP's purpose-built schema reader.

It returns the full structured type description in a single call: id, apiIdentifier, display names, default create form, detail forms, and every attribute with `{ dataType, multiValue, languageDependent, variantDependent, defaultForm, apiIdentifier }`. No more manually walking Type → 5001 / 5002 → form → 5053 → attributes.

## Parameters

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "languages": [201, 202]
}
```

- `branchId` — Numeric branch ID (int).
- `objectTypeId` — The type to describe (int).
- `languages` — Languages to load for display names (e.g. `[201, 202]`).

## Response Shape

```json
{
  "id": 2100000,
  "apiIdentifier": "industry",
  "name": { "201": "Industry", "202": "Branche" },
  "description": { "201": "...", "202": "..." },
  "displayNameAttrId": 1000,
  "createFormId": 2100101,
  "detailFormIds": [2100101, 2100102],
  "attributes": [
    {
      "id": 2100050,
      "apiIdentifier": "name",
      "name": { "201": "Name", "202": "Name" },
      "dataType": "String",
      "multiValue": false,
      "languageDependent": true,
      "variantDependent": false,
      "defaultForm": 2100101
    }
  ]
}
```

## Fallback (raw attribute reads)

If `objecttype_describe` does not return information you need (rare — it covers the standard schema model), the old chain still works: read attributes 5001 (create form) / 5002 (detail forms) on the type, then attribute 5053 (fields) on each form, then the referenced attribute definitions. Use `object_get` for each hop.

## Edge Case

A type with no forms (neither 5001 nor any 5002) gets an empty `attributes` array. This happens for abstract or system-level types.

## Common Patterns

### Type → Form → Attribute Chain
`objecttype_describe` walks this chain for you. Prefer it over manual resolution.

### BinRef Attributes
Attributes with data type `BinRef` point to binary objects. To download the file → use `get-file` skill (currently unsupported in the C# MCP — see that skill for details).

### API Response
Returns an `ObjectTypeDescription` JSON object — no raw `CmsObject` envelope.
