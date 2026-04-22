---
name: set-virtualization-code
description: "Set JavaScript virtualization code on a virtual attribute."
user-invocable: false
allowed-tools: object_update
---

# Set Virtualization Code

## Scope

**This skill ONLY handles:** Setting JavaScript virtualization code (attribute 1009) on an existing virtual attribute definition.

**For validation code** → use `set-validation-code`
**For other attribute property changes** → use `update-attribute`

Set JavaScript code that computes a virtual attribute's value server-side. The attribute must have `isVirtual=true` (attribute 1020) for this code to take effect.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`object_update`

## Parameters

```json
{
  "branchId": 2100347,
  "objectId": 12345,
  "values": "[{\"attribute\":1009,\"language\":0,\"variant\":0,\"value\":\"// compute value here\"}]",
  "comment": "Set virtualization logic"
}
```

- `branchId` — Numeric branch ID (int).
- `objectId` — Attribute definition ID.
- `values` — JSON-encoded **string** of a CmsValue array. The single entry sets attribute 1009.
- `comment` — (optional) Audit trail.

## Prerequisite

The attribute must be marked as virtual (`isVirtual=true`, attribute 1020). If it is not, first update the attribute via `update-attribute` to set it as virtual before setting virtualization code.

## Response

Returns `UpdateObjectResult` with `objectId`, `updatedObjects`, `createdValues`, `transaction`.

## Common Patterns

### CmsValue Format
Every value entry in the JSON string follows: `{ attribute, language, variant, value }`
- `language`: 0=language-independent (always 0 for code attributes)
- `variant`: 0=default

### API Response (PATCH/Update)
Returns `{ transaction }`. Fetch the object afterward to confirm changes.
