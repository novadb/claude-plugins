---
name: update-attribute
description: "Update properties of an existing attribute definition (typeRef=10)."
user-invocable: false
allowed-tools: object_update
---

# Update Attribute

## Scope

**This skill ONLY handles:** Updating properties of an existing attribute definition (typeRef=10).

**Do NOT use this skill for:**
- Creating new attributes → use `create-attribute`
- Fetching/reading attributes → use `get-attribute`
- Deleting attributes → use `delete-attribute`
- Searching for attributes → use `search-attributes`

Update properties of an existing attribute definition (typeRef=10). Only send changed fields.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`object_update`

## Parameters

- `branchId` — Numeric branch ID (int). Always use the branch the user is currently working on.
- `objectId` — The attribute definition ID (int).
- `values` — JSON-encoded **string** of a CmsValue array (see example below).
- `comment` — (optional) Audit trail comment.

## Attribute Property IDs

Only include values for fields being changed. See the full table in the create-attribute skill.

| Field | Attr ID | Type | Language | Notes |
|-------|---------|------|----------|-------|
| Name (EN) | 1000 | String | 201 | |
| Name (DE) | 1000 | String | 202 | |
| Description (EN) | 1012 | TextRef | 201 | |
| Description (DE) | 1012 | TextRef | 202 | |
| Data type | 1001 | String.DataType | 0 | |
| Language dependent | 1017 | Boolean | 0 | |
| Required | 1018 | Boolean | 0 | |
| Allow multiple | 1004 | Boolean | 0 | |
| Predefined values | 1006 | Boolean | 0 | |
| Unique values | 1032 | Boolean | 0 | |
| Allowed types | 1015 | ObjRef | 0 | Separate entries with `sortReverse` |
| Inheritance behavior | 1013 | String | 0 | `"None"` / `"Inheriting"` / `"InheritingAll"` |
| Virtual | 1020 | Boolean | 0 | |
| Parent group | 1019 | ObjRef | 0 | |
| Sortable values | 1005 | Boolean | 0 | |
| Format string | 1007 | String | 0 | |
| Search filter | 1010 | Boolean | 0 | |
| List view column | 1011 | Boolean | 0 | |
| Variant axis | 1002 | ObjRef | 0 | |
| Unit of measure | 1003 | ObjRef | 0 | |
| Reverse relation name (EN) | 1051 | String | 201 | |
| Reverse relation name (DE) | 1051 | String | 202 | |
| Sortable child objects | 1023 | Boolean | 0 | |
| Disable spell checking | 1028 | Boolean | 0 | |
| Validation code | 1008 | TextRef.JS | 0 | |
| Virtualization code | 1009 | TextRef.JS | 0 | |

## Example Call

Rename an attribute and make it required:

```json
{
  "branchId": 2100347,
  "objectId": 12345,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"New Name\"},{\"attribute\":1018,\"language\":0,\"variant\":0,\"value\":true}]",
  "comment": "Renamed and set as required"
}
```

Note `values` is a **JSON-encoded string**, not an array literal — the new C# MCP expects it that way.

## Response

Returns `UpdateObjectResult` with `objectId`, `updatedObjects`, `createdValues`, and `transaction`.

## Important

- `apiIdentifier` CAN be changed via update, but must be unique across all branches and attributes. Only set when explicitly necessary.
- Only provide fields that are changing. Omitted fields are untouched (read-then-merge semantics).

## Common Patterns

### CmsValue Format
Every value entry in the JSON string follows: `{ attribute, language, variant, value, sortReverse? }`
- `language`: 201=EN, 202=DE, 0=language-independent
- `variant`: 0=default
- `sortReverse`: for multi-value ordering (0, 1, 2, ...)

### Multi-Value ObjRef
Separate entries with sortReverse (can also be passed as an array inside the JSON string — the MCP auto-expands):
- ✓ `{"attribute":1015,"value":id1,"sortReverse":0}, {"attribute":1015,"value":id2,"sortReverse":1}`
- ✓ `{"attribute":1015,"value":[id1,id2]}` (auto-expanded by the MCP)

### API Response (PATCH/Update)
Returns `{ transaction }`. Fetch the object afterward to confirm changes.
