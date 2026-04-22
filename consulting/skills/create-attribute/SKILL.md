---
name: create-attribute
description: "Create a new attribute definition (typeRef=10) in NovaDB."
user-invocable: false
allowed-tools: object_create, object_get
---

# Create Attribute

## Scope

**This skill ONLY handles:** Creating new attribute definitions (typeRef=10).

**Do NOT use this skill for:**
- Reading/fetching existing attributes → use `get-attribute`
- Updating existing attributes → use `update-attribute`
- Deleting attributes → use `delete-attribute`
- Searching for attributes → use `search-attributes`
- Creating object types, forms, or branches → use the respective create skill

Create an attribute definition (typeRef=10) in NovaDB. Supports all data types, multi-value, ObjRef constraints, language-dependence, and optional JS validation.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tools

1. `object_create` — Create the attribute definition (returns `createdObjectId`)
2. `object_get` — Optional: fetch the full created attribute for confirmation

## Parameters

- `branchId` — Numeric branch ID (int). Always use the branch the user is currently working on.
- `objectTypeId` — Always `10`.
- `values` — JSON-encoded **string** of a CmsValue array.
- `comment` — (optional) Audit trail.

### Attribute Property IDs

Only include values for fields that are explicitly needed. Name (EN) and data type are always required. Apply the **principle of minimal configuration** — do not set properties unless the user requests them or the business case clearly requires them.

### Decision Guidelines

- **`apiIdentifier`**: Not settable on create via this tool — set after creation via `object_update` if needed.
- **Language dependent (1017)**: Always question whether language-dependence is appropriate for the business case. Only set to `true` when the attribute genuinely holds translatable content.
- **Inheritance (1013)**: Default is `"None"`. Only set when explicitly requested.
- **Allow multiple (1004)**: Default is `false`. Verify case-by-case before enabling.
- **All other boolean/optional fields**: Only set when explicitly necessary. Do not speculatively enable features.

| Field | Attr ID | Type | Language | Notes |
|-------|---------|------|----------|-------|
| Name (EN) | 1000 | String | 201 | **Required** |
| Name (DE) | 1000 | String | 202 | |
| Description (EN) | 1012 | TextRef | 201 | |
| Description (DE) | 1012 | TextRef | 202 | |
| Data type | 1001 | String.DataType | 0 | **Required** — see enum below |
| Language dependent | 1017 | Boolean | 0 | Only set when appropriate |
| Required | 1018 | Boolean | 0 | |
| Allow multiple | 1004 | Boolean | 0 | |
| Predefined values | 1006 | Boolean | 0 | |
| Unique values | 1032 | Boolean | 0 | |
| Allowed types | 1015 | ObjRef | 0 | For ObjRef — multi-value (pass as array in `value`) |
| Inheritance behavior | 1013 | String | 0 | `"None"` / `"Inheriting"` / `"InheritingAll"` |
| Virtual | 1020 | Boolean | 0 | |
| Parent group | 1019 | ObjRef | 0 | |
| Sortable values | 1005 | Boolean | 0 | |
| Format string | 1007 | String | 0 | e.g. `"dd.MM.yyyy"` |
| Search filter | 1010 | Boolean | 0 | |
| List view column | 1011 | Boolean | 0 | |
| Variant axis | 1002 | ObjRef | 0 | |
| Unit of measure | 1003 | ObjRef | 0 | Ref type 30 |
| Reverse relation name (EN) | 1051 | String | 201 | |
| Reverse relation name (DE) | 1051 | String | 202 | |
| Sortable child objects | 1023 | Boolean | 0 | |
| Disable spell checking | 1028 | Boolean | 0 | |
| Validation code | 1008 | TextRef.JS | 0 | JS: receives `value` global, return string to reject |
| Virtualization code | 1009 | TextRef.JS | 0 | Requires `isVirtual: true` |

### Data Type Enum

Valid values for attribute 1001:

```
String, TextRef, TextRef.JavaScript, TextRef.CSS,
XmlRef.SimpleHtml, XmlRef.VisualDocument,
Integer, Decimal, Float, Boolean,
DateTime, DateTime.Date,
ObjRef, BinRef, BinRef.Icon, BinRef.Thumbnail,
String.DataType, String.InheritanceBehavior, String.UserName, String.RGBColor
```

### Call Example

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"Industry\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Branche\"},{\"attribute\":1001,\"language\":0,\"variant\":0,\"value\":\"String\"}]",
  "comment": "Created via AI assistant"
}
```

### ObjRef with Allowed Types Example

For an ObjRef attribute that references specific types, multi-value on 1015 — pass as array inside `value`:

```json
[{"attribute":1001,"language":0,"variant":0,"value":"ObjRef"},
 {"attribute":1015,"language":0,"variant":0,"value":[12345,67890]}]
```

## Step 2: Fetch the Created Attribute

The response returns `{ createdObjectId, transaction }`. Use `object_get` to fetch the full object:

```json
{
  "branchId": 2100347,
  "objectIds": [<createdObjectId>],
  "languages": [201, 202],
  "attributes": []
}
```

## Minimum Required

- Name in English (attribute 1000, language 201)
- Data type (attribute 1001)

## Common Patterns

### CmsValue Format (inside the JSON string)
Every value entry follows: `{ attribute, language, variant, value, sortReverse? }`
- `language`: 201=EN, 202=DE, 0=language-independent
- `variant`: 0=default

### Multi-Value ObjRef
Either form works:
- ✓ Array: `{"attribute":1015,"value":[id1,id2]}`
- ✓ Entries with sortReverse: `{"attribute":1015,"value":id1,"sortReverse":0}`

### API Response (Create)
Returns `{ createdObjectId, transaction }`.
