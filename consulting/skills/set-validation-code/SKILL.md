---
name: set-validation-code
description: "Set JavaScript validation code on an attribute definition."
user-invocable: false
allowed-tools: object_update
---

# Set Validation Code

## Scope

**This skill ONLY handles:** Setting JavaScript validation code (attribute 1008) on an existing attribute definition.

**For virtualization code** → use `set-virtualization-code`
**For other attribute property changes** → use `update-attribute`

Set JavaScript validation code on an attribute definition. The script validates user input server-side.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`object_update`

## Parameters

```json
{
  "branchId": 2100347,
  "objectId": 12345,
  "values": "[{\"attribute\":1008,\"language\":0,\"variant\":0,\"value\":\"if (value && !/^[A-Z]{2}$/.test(value)) { 'Must be exactly 2 capital letters'; }\"}]",
  "comment": "Added country code validation"
}
```

- `branchId` — Numeric branch ID (int).
- `objectId` — Attribute definition ID.
- `values` — JSON-encoded **string** of a CmsValue array. The single entry sets attribute 1008.
- `comment` — (optional) Audit trail.

## Validation Script Pattern

The script receives `value` as a global variable containing the user's input.

- **Return a string** to reject the value — the string becomes the error message
- **Return nothing** (or falsy) to accept the value

### Examples

```javascript
// Reject if not exactly 2 uppercase letters
if (value && !/^[A-Z]{2}$/.test(value)) {
  'Must be exactly 2 capital letters';
}
```

```javascript
// Reject negative numbers
if (value < 0) {
  'Value must be non-negative';
}
```

```javascript
// Reject empty strings
if (typeof value === 'string' && value.trim() === '') {
  'Value must not be empty';
}
```

## Response

Returns `UpdateObjectResult` with `objectId`, `updatedObjects`, `createdValues`, `transaction`.

## Common Patterns

### CmsValue Format
Every value entry in the JSON string follows: `{ attribute, language, variant, value }`
- `language`: 0=language-independent (always 0 for code attributes)
- `variant`: 0=default

### API Response (PATCH/Update)
Returns `{ transaction }`. Fetch the object afterward to confirm changes.
