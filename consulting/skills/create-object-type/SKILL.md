---
name: create-object-type
description: "Create a new object type with attributes and form in NovaDB."
user-invocable: false
allowed-tools: object_create, object_update, object_get, objecttype_describe
---

# Create Object Type

Create a new object type with attributes and form in NovaDB. ONLY for creating object types — NOT for updating, reading, or adding attributes to existing types.

## Scope

**This skill ONLY handles:** Creating a new object type (typeRef=0) including attribute definitions and form setup.

**For modifying an existing type** → use `update-object-type`
**For adding attributes to an existing type** → use `add-attribute-to-type`

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

> **Atomicity caveat:** The new C# MCP creates **one object per call**. Steps 1–4 below happen in separate transactions; a failure partway through will leave the system in a partial state. The old MCP supported batched `create_objects` — this is a known gap to surface back to whoever owns the C# MCP. Plan your flow to tolerate partial state (e.g. validate before each step, or delete-and-retry on failure).

## Tools Used

- `object_create` — Create type, attribute definitions, and form (one object per call)
- `object_update` — Link form to type
- `objecttype_describe` — Fetch the finished type for confirmation

## Workflow (5 Steps)

### Step 1: Create the Object Type

Call `object_create` with `objectTypeId: 0`:

```json
{
  "branchId": 2100347,
  "objectTypeId": 0,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"English Name\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"German Name\"},{\"attribute\":1012,\"language\":201,\"variant\":0,\"value\":\"English Description\"},{\"attribute\":1012,\"language\":202,\"variant\":0,\"value\":\"German Description\"}]",
  "comment": "Create type"
}
```

- `objectTypeId: 0` = Object Type
- `values` is a JSON-encoded **string** of a CmsValue array.
- Only include language/description values actually provided.
- Response: `CreateObjectResult` with `createdObjectId` (the new type id).

### Step 2: Create Attribute Definitions (if any)

For each attribute the user wants, call `object_create` with `objectTypeId: 10`:

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"Attr Name EN\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Attr Name DE\"},{\"attribute\":1001,\"language\":0,\"variant\":0,\"value\":\"String\"}]"
}
```

- `objectTypeId: 10` = Attribute Definition
- Loop over attributes; each call returns one `createdObjectId`.
- Collect all new attribute ids for the form step.

#### Attribute Definition Values

| Attribute ID | Field | Type | Notes |
|-------------|-------|------|-------|
| 1000 | Name | String | Language-dependent (201=EN, 202=DE) |
| 1001 | Data type | String | See data types below |
| 1006 | Required/Predefined | Boolean | Whether values are required |
| 1010 | Has local values | Boolean | |
| 1014 | Max values | Integer | 1=single-value, 0=multi-value |
| 1015 | Allowed types | ObjRef | For ObjRef data type — multi-value |

#### Valid Data Types

`String`, `TextRef`, `TextRef.JavaScript`, `TextRef.CSS`, `XmlRef.SimpleHtml`, `XmlRef.VisualDocument`, `Integer`, `Decimal`, `Float`, `Boolean`, `DateTime`, `DateTime.Date`, `ObjRef`, `BinRef`, `BinRef.Icon`, `BinRef.Thumbnail`, `String.DataType`, `String.InheritanceBehavior`, `String.UserName`, `String.RGBColor`

### Step 3: Create a Form

Call `object_create` with `objectTypeId: 50` linking to the attribute definitions. The MCP auto-expands array values for multi-value attributes, so 5053 can be passed in a single entry:

```json
{
  "branchId": 2100347,
  "objectTypeId": 50,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"English Name\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"German Name\"},{\"attribute\":5056,\"language\":0,\"variant\":0,\"value\":false},{\"attribute\":5053,\"language\":0,\"variant\":0,\"value\":[<attrId1>,<attrId2>]}]"
}
```

- `objectTypeId: 50` = Form
- `5056` = isSingleEditor (set to `false` by default)
- `5053` = Form content — the array order IS the display order.
- Response: `createdObjectId` = form id.

### Step 4: Link Form to Type

Call `object_update` to set the form as both create form and detail tab:

```json
{
  "branchId": 2100347,
  "objectId": <typeId>,
  "values": "[{\"attribute\":5001,\"language\":0,\"variant\":0,\"value\":<formId>},{\"attribute\":5002,\"language\":0,\"variant\":0,\"value\":[<formId>]}]"
}
```

- `5001` = Create form (single ObjRef)
- `5002` = Detail forms (multi-value ObjRef; pass as array inside `value`)

### Step 5: Confirm

Call `objecttype_describe` to fetch the finished type and return a structured summary:

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "languages": [201, 202]
}
```

## Minimum Required

Only `nameEn` (attribute 1000, language 201) is required. All other fields — German name, descriptions, attribute definitions — are optional.

## Skip Steps 2–4

If no attribute definitions are requested, skip steps 2–4 entirely. Just create the bare type in step 1 and describe it in step 5.

## Common Patterns

### CmsValue Format (inside the JSON string)
Every value entry follows: `{ attribute, language, variant, value, sortReverse? }`
- `language`: 201=EN, 202=DE, 0=language-independent
- `variant`: 0=default
- `sortReverse`: explicit multi-value ordering (0, 1, 2, …)

### Multi-Value ObjRef
The new MCP accepts either form:
- ✓ Array: `{"attribute":5053,"value":[id1,id2]}` (array order = display order)
- ✓ Entries with sortReverse: `{"attribute":5053,"value":id1,"sortReverse":0}`

### API Response (Create)
Returns `{ createdObjectId, transaction }`. Use the id in subsequent calls.
