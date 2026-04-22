---
name: add-attribute-to-type
description: "Create a new attribute and add it to all forms of an existing type."
user-invocable: false
allowed-tools: objecttype_describe, object_get, object_create, object_update
---

# Add Attribute to Type

Create a new attribute and add it to all forms of an existing type. ONLY for adding attributes to existing types — NOT for creating types, updating type properties, or editing forms directly.

## Scope

**This skill ONLY handles:** Creating a new attribute definition and adding it to all forms of an existing object type.

**For creating a standalone attribute** → use `create-attribute`
**For reading a type's current attributes** → use `get-object-type`

> **Atomicity caveat:** The new C# MCP creates/updates **one object per call**. The attribute creation and each form update run in separate transactions. If one step fails, earlier steps remain committed. Plan accordingly.

## Tools Used

- `objecttype_describe` — Read the target type's form assignments (primary) and current attribute list
- `object_get` — Fetch each form to read existing fields
- `object_create` — Create the new attribute definition
- `object_update` — Update each form with the new field list

## Workflow (5 Steps)

### Step 1: Describe the Object Type

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "languages": [201, 202]
}
```

Tool: `objecttype_describe`. The response lists the default create form, all detail forms, and every attribute currently wired in — use this instead of reading raw attributes 5001/5002.

### Step 2: Identify Forms to Update

Collect the create form id and all detail form ids from the describe result. If the type has no create form, stop with an error — the type must have a create form before attributes can be added.

### Step 3: Read Existing Form Fields

For each form id, call `object_get` and extract existing field IDs from attribute **5053** (form content). Sort by `sortReverse` ascending. Deduplicate across forms.

### Step 4: Create the Attribute Definition

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"Attr Name EN\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Attr Name DE\"},{\"attribute\":1001,\"language\":0,\"variant\":0,\"value\":\"String\"}]"
}
```

Tool: `object_create`. Response: `createdObjectId` = the new attribute definition id.

#### Attribute Definition Values

| Attribute ID | Field | Type | Notes |
|-------------|-------|------|-------|
| 1000 | Name | String | Language-dependent (201=EN, 202=DE) |
| 1001 | Data type | String | See valid data types below |
| 1006 | Required/Predefined | Boolean | Whether values are required |
| 1010 | Has local values | Boolean | |
| 1014 | Max values | Integer | 1=single-value, 0=multi-value |
| 1015 | Allowed types | ObjRef | For ObjRef data type — multi-value |

#### Valid Data Types

`String`, `TextRef`, `TextRef.JavaScript`, `TextRef.CSS`, `XmlRef.SimpleHtml`, `XmlRef.VisualDocument`, `Integer`, `Decimal`, `Float`, `Boolean`, `DateTime`, `DateTime.Date`, `ObjRef`, `BinRef`, `BinRef.Icon`, `BinRef.Thumbnail`, `String.DataType`, `String.InheritanceBehavior`, `String.UserName`, `String.RGBColor`

### Step 5: Update Each Form

For each form identified in step 2, call `object_update` replacing the 5053 list with existing fields + the new one:

```json
{
  "branchId": 2100347,
  "objectId": <formId>,
  "values": "[{\"attribute\":5053,\"language\":0,\"variant\":0,\"value\":[<existingId1>,<existingId2>,<newAttrDefId>]}]"
}
```

The MCP auto-expands the array into ordered `sortReverse` entries.

## Result

Call `objecttype_describe` once more to confirm the attribute is wired through all forms. Return the new attribute id and the updated form ids to the user.

## Common Patterns

### CmsValue Format (inside the JSON string)
Every value entry follows: `{ attribute, language, variant, value, sortReverse? }`
- `language`: 201=EN, 202=DE, 0=language-independent
- `variant`: 0=default

### Multi-Value ObjRef (Form Fields)
Either form works:
- ✓ Array: `{"attribute":5053,"value":[fieldId1,fieldId2]}` (array order = display order)
- ✓ `sortReverse` entries: `{"attribute":5053,"value":fieldId1,"sortReverse":0}`

### Read-Modify-Write Pattern
1. Describe the type to get forms
2. Read each form's current fields (attr 5053)
3. Create the new attribute
4. Send complete field list (existing + new) to each form via `object_update`

### API Response (Create)
Returns `{ createdObjectId, transaction }`.
