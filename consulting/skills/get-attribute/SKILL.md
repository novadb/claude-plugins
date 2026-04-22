---
name: get-attribute
description: "Fetch a single attribute definition by ID."
user-invocable: false
allowed-tools: object_get
---

# Get Attribute

## Scope

**This skill ONLY handles:** Fetching a single attribute definition (typeRef=10) by its ID.

**For searching/filtering attributes by name or properties** → use `search-attributes`

Fetch a single attribute definition with all inherited properties.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`object_get`

## Parameters

```json
{
  "branchId": 2100347,
  "objectIds": [12345],
  "languages": [201, 202],
  "attributes": []
}
```

- `branchId` — Numeric branch ID (int32). Always use the branch the user is currently working on.
- `objectIds` — Array of object IDs (int[]). Pass `[<attrId>]` for a single attribute.
- `languages` — Languages to load, e.g. `[201, 202]` for EN + DE. Use `language_query` if unknown.
- `attributes` — Optional list of attribute IDs to restrict the fetch (empty = all).

## Response

Returns `ObjectQueryResult` containing one object with `meta` (id, guid, apiIdentifier, typeRef=10) and `values` array containing all attribute properties.

## Common Patterns

### CmsValue Format
Every value entry follows: `{ attribute, language, variant, value, sortReverse? }`
- `language`: 201=EN, 202=DE, 0=language-independent
- `variant`: 0=default

### API Response (GET)
Returns an `ObjectQueryResult` wrapping a `CmsObject` array with `meta` and `values`.
