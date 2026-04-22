---
name: link-form-to-type
description: "Attach a form to an object type as create form or detail tab."
user-invocable: false
allowed-tools: objecttype_describe, object_update
---

# Link Form to Type

## Scope

**This skill ONLY handles:** Linking an existing form to an object type as either the create form (attribute 5001) or a detail tab (attribute 5002).

**For creating new forms** → use `create-form`
**For editing form fields** → use `set-form-fields` or `add-form-field`

Attach a form to an object type as either a **create form** (shown when creating new objects) or a **detail tab** (shown in the edit view).

> **Atomicity caveat:** Creating a form and linking it to a type happen in two separate transactions on the new C# MCP. If the link step fails after a successful form creation, the form is still around — clean up or retry as needed.

## Tools Used

- `object_update` — Set form reference on the type
- `objecttype_describe` — (detail role only) Read the current list of detail forms before appending

## Two Roles

### Create Form (attr 5001)

A type has a single create form. Setting it replaces any previous create form.

```json
{
  "branchId": 2100347,
  "objectId": <typeId>,
  "values": "[{\"attribute\":5001,\"language\":0,\"variant\":0,\"value\":<formId>}]"
}
```

Tool: `object_update`.

### Detail Tab (attr 5002)

A type can have multiple detail forms (tabs). New forms are **appended** to the existing list.

#### Step 1: Describe the Type

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "languages": [201, 202]
}
```

Tool: `objecttype_describe`. Extract the list of detail forms from the result.

#### Step 2: Append and Update

Add the new form id to the list, preserving existing order. Send the full list via `object_update`:

```json
{
  "branchId": 2100347,
  "objectId": <typeId>,
  "values": "[{\"attribute\":5002,\"language\":0,\"variant\":0,\"value\":[<existingFormId1>,<existingFormId2>,<newFormId>]}]"
}
```

Array order inside `value` = tab order (left-to-right).

## Response

Returns `UpdateObjectResult` with `objectId`, `updatedObjects`, `createdValues`, `transaction`.

## Common Patterns

### Multi-Value ObjRef (Detail Forms)
Either form works:
- ✓ Array: `{"attribute":5002,"value":[formId1,formId2]}`
- ✓ `sortReverse` entries: `{"attribute":5002,"value":formId1,"sortReverse":0}`

Create form (attribute 5001) is single-value — no sortReverse needed.

### Read-Modify-Write Pattern (Detail Forms)
1. Describe the type to get current detail forms
2. Append the new form id to the list
3. Send complete form list via `object_update`

### API Response (Update)
Returns `{ transaction }`. Describe the type afterward to confirm.
