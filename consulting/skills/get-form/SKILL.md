---
name: get-form
description: "Fetch a form and resolve all referenced field attribute definitions."
user-invocable: false
allowed-tools: object_get
---

# Get Form

## Scope

**This skill ONLY handles:** Fetching a form definition and resolving its field references to attribute definitions.

**For editing form fields** → use `set-form-fields` or `add-form-field`
**For creating new forms** → use `create-form`

Fetch a form with its field attribute definitions resolved. Returns the form object and all referenced field definitions.

## Tool

`object_get` — used twice (form, then its fields in bulk)

## Workflow

### Step 1: Fetch the Form

```json
{
  "branchId": 2100347,
  "objectIds": [<formId>],
  "languages": [201, 202],
  "attributes": []
}
```

### Step 2: Extract Field IDs

From the form's `values` array, find all entries with `attribute: 5053`. Sort them by `sortReverse` (ascending) to get the correct field order. Extract the `value` from each — these are the attribute definition IDs.

### Step 3: Fetch Field Definitions

If there are field IDs, call `object_get` again with them:

```json
{
  "branchId": 2100347,
  "objectIds": [<attrDefId1>, <attrDefId2>, <attrDefId3>],
  "languages": [201, 202],
  "attributes": []
}
```

### Step 4: Return Combined Result

Present the result as `{ form, fields }` where `form` is the full form object and `fields` is the array of resolved attribute definition objects.

## Key Details

- Form content is stored in attribute **5053** as multi-value ObjRef entries
- Sort by `sortReverse` ascending to get the correct field order
- If the form has no fields (no 5053 entries), return an empty `fields` array
- `object_get` accepts numeric IDs only — resolve GUID/apiIdentifier via `object_query` first if needed

## Common Patterns

### ObjRef Resolution
Form field references (attribute 5053) are numeric IDs pointing to attribute definitions. Resolve them with another `object_get` call.

### API Response (GET)
Returns an `ObjectQueryResult` with `objects[]` each having `meta` and `values`.
