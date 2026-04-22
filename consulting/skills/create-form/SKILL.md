---
name: create-form
description: "Create a new form definition (typeRef=50) in NovaDB."
user-invocable: false
allowed-tools: object_create, object_get
---

# Create Form

## Scope

**This skill ONLY handles:** Creating a new form definition (typeRef=50) with initial field list.

**For editing fields on an existing form** → use `set-form-fields` or `add-form-field`
**For linking a form to an object type** → use `link-form-to-type`

Create a new form (typeRef=50) in NovaDB. Forms define which attribute fields are shown when editing objects. Use `link-form-to-type` to attach the form to an object type after creation.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tools Used

- `object_create` — Create the form object
- `object_get` — Fetch the created form for confirmation

## Workflow

### Step 1: Create the Form

```json
{
  "branchId": 2100347,
  "objectTypeId": 50,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"Form Name EN\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Form Name DE\"},{\"attribute\":5056,\"language\":0,\"variant\":0,\"value\":false},{\"attribute\":5053,\"language\":0,\"variant\":0,\"value\":[1234,5678]}]"
}
```

- `objectTypeId: 50` = Form
- `values` is a JSON-encoded **string** of a CmsValue array.
- `5053` (form content) can be passed as an array inside `value` — array order is the display order.
- Response: `createdObjectId` = new form id.

### Step 2: Fetch and Return

Call `object_get` with the new form id to confirm and return the result:

```json
{
  "branchId": 2100347,
  "objectIds": [<formId>],
  "languages": [201, 202],
  "attributes": []
}
```

## Form Attribute IDs

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Name (EN) | 1000 | String | `language: 201`, `variant: 0` |
| Name (DE) | 1000 | String | `language: 202`, `variant: 0` |
| Form content (fields) | 5053 | ObjRef | **Multi-value** — array in `value` OR separate entries with `sortReverse` |
| Condition attribute | 5054 | ObjRef | Form visible only when this attribute has a value |
| Condition refs | 5055 | ObjRef | Multi-value |
| Is single editor | 5056 | Boolean | Render as single code editor instead of individual fields |

## Form Content (5053) — Multi-Value Pattern

The new MCP auto-expands array values for multi-value attributes. Either form works:

- ✓ Array: `{"attribute":5053,"value":[id1,id2,id3]}`
- ✓ Entries with sortReverse:
  ```json
  {"attribute":5053,"language":0,"variant":0,"value":<id1>,"sortReverse":0},
  {"attribute":5053,"language":0,"variant":0,"value":<id2>,"sortReverse":1}
  ```

## Minimum Required

Only `nameEn` (attribute 1000, language 201) and `isSingleEditor` (5056, typically `false`) are required. Fields, German name, condition attribute are all optional.

## Common Patterns

### CmsValue Format (inside the JSON string)
Every value entry follows: `{ attribute, language, variant, value, sortReverse? }`
- `language`: 201=EN, 202=DE, 0=language-independent
- `variant`: 0=default

### API Response (Create)
Returns `{ createdObjectId, transaction }`. Use the id to fetch the full object.
