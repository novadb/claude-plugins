---
name: set-form-fields
description: "Replace ALL fields on a form with a complete ordered list."
user-invocable: false
allowed-tools: object_update
---

# Set Form Fields

## Scope

**This skill ONLY handles:** Replacing the entire field list (attribute 5053) on an existing form with a new complete list.

**For adding a single field without replacing others** → use `add-form-field`
**For reading a form's current fields** → use `get-form`

Replace ALL fields on a form. Provide the COMPLETE ordered list of attribute definition IDs — any omitted fields will be removed from the form.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`object_update`

## Parameters

```json
{
  "branchId": 2100347,
  "objectId": <formId>,
  "values": "[{\"attribute\":5053,\"language\":0,\"variant\":0,\"value\":[<attrDefId1>,<attrDefId2>,<attrDefId3>]}]",
  "comment": "Replaced form fields"
}
```

- `branchId` — Numeric branch ID (int).
- `objectId` — Form ID (int).
- `values` — JSON-encoded **string** of a CmsValue array. Multi-value attributes can be passed as an array inside `value` — the C# MCP auto-expands to individual entries.
- `comment` — (optional) Audit trail.

## Multi-Value Pattern

**CRITICAL**: Because the new MCP auto-expands arrays for multi-value attributes, either shape works — pick one. With an explicit array, the order of the array IS the sort order:

```json
[{"attribute":5053,"language":0,"variant":0,"value":[1001,1002,1003]}]
```

Or equivalent sortReverse form:

```json
[{"attribute":5053,"language":0,"variant":0,"value":1001,"sortReverse":0},
 {"attribute":5053,"language":0,"variant":0,"value":1002,"sortReverse":1},
 {"attribute":5053,"language":0,"variant":0,"value":1003,"sortReverse":2}]
```

## Response

Returns `UpdateObjectResult` with `objectId`, `updatedObjects`, `createdValues`, `transaction`.

## Important

This is a **full replacement** operation. All existing fields are replaced by the provided list. To add a single field without replacing all others, use the `add-form-field` skill instead.

## Common Patterns

**WARNING:** This is a full replacement operation. Omitted fields will be removed from the form. Always read the current form first to avoid data loss.

### API Response (PATCH/Update)
Returns `{ transaction }`. Fetch the form afterward to confirm changes.
