---
name: add-form-field
description: "Append a single attribute definition to a form's field list."
user-invocable: false
allowed-tools: object_get, object_update
---

# Add Form Field

## Scope

**This skill ONLY handles:** Appending a single attribute definition to an existing form's field list without replacing other fields.

**For replacing all fields at once** → use `set-form-fields`
**For reading a form's current fields** → use `get-form`

Append an attribute definition to the end of a form's field list without replacing existing fields.

## Tools Used

- `object_get` — Read current form to get existing fields
- `object_update` — Update the form with the new complete field list

## Workflow

### Step 1: Read Current Form

```json
{
  "branchId": 2100347,
  "objectIds": [<formId>],
  "languages": [201, 202],
  "attributes": []
}
```

Tool: `object_get`.

### Step 2: Extract Existing Fields

From the form's `values` array, find all entries with `attribute: 5053`. Sort by `sortReverse` ascending. Extract the `value` (attribute definition ID) from each.

### Step 3: Build Updated Field List

Append the new attribute definition id to the existing list. The MCP auto-expands arrays for multi-value 5053 — no need to emit individual sortReverse entries.

### Step 4: Update the Form

```json
{
  "branchId": 2100347,
  "objectId": <formId>,
  "values": "[{\"attribute\":5053,\"language\":0,\"variant\":0,\"value\":[<existingId1>,<existingId2>,<newAttrDefId>]}]"
}
```

Tool: `object_update`.

## Key Details

- The complete field list must be sent — this is a replacement of all 5053 values.
- Array order inside `value` determines display order.
- The new field is appended at the end of the array.

## Response

Returns `UpdateObjectResult` with `objectId`, `updatedObjects`, `createdValues`, `transaction`.

## Common Patterns

### Multi-Value ObjRef (Form Fields)
Either form works:
- ✓ Array: `{"attribute":5053,"value":[fieldId1,fieldId2]}`
- ✓ `sortReverse` entries: `{"attribute":5053,"value":fieldId1,"sortReverse":0}`

### Read-Modify-Write Pattern
1. Read current form via `object_get`
2. Extract existing field IDs from attribute 5053
3. Append new field to the array
4. Send complete field list back via `object_update`

### API Response (Update)
Returns `{ transaction }`. Fetch the form afterward to confirm.
