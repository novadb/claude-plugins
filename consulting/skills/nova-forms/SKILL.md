---
name: nova-forms
description: "Form architecture reference — attributes, multi-value patterns, form-type linkage, and configuration workflows."
user-invocable: false
allowed-tools: object_get, object_query, object_create, object_update, object_delete, object_count, objecttype_describe
---

# NovaDB Form Configuration Reference

## Scope

**This skill is EXCLUSIVELY a reference for:** Inspecting and configuring NovaDB forms (typeRef=50) — field layouts, conditional visibility, form-type linkage, and form CRUD operations.

**For schema browsing beyond forms** → see `nova-explore` skill
**For searching data objects** → see `nova-search` skill

Complete reference for form architecture, attribute IDs, value format, workflows, and condition system.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All numeric IDs in examples are samples — always use real IDs from the target system.

## Form Basics

Forms (typeRef=50) define which attribute fields are shown when editing objects in the UI. They control layout, field ordering, and conditional visibility.

---

## Form Attribute IDs

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Name (EN) | 1000 | String | `language: 201`, `variant: 0` |
| Name (DE) | 1000 | String | `language: 202`, `variant: 0` |
| Description | 1012 | String | Language-dependent |
| API Identifier | 1021 | String | Language-independent |
| Form content (fields) | 5053 | ObjRef | **Multi-value** — one entry per field (array-in-value also works) |
| Condition attribute | 5054 | ObjRef | Single — form visible only when this attribute matches |
| Condition refs | 5055 | ObjRef | Multi-value — which values trigger visibility |
| Is single editor | 5056 | Boolean | Render as single code editor instead of individual fields |

---

## Multi-Value Pattern (attribute 5053)

Form content is multi-value. The new C# MCP accepts either form — pick the one that's clearer in context:

### As an array inside `value` (recommended)

```json
{ "attribute": 5053, "language": 0, "variant": 0, "value": [2100500, 2100501, 2100502] }
```

Array order = display order. The MCP auto-expands this to individual entries with correct `sortReverse`.

### As separate entries with `sortReverse`

```json
{ "attribute": 5053, "language": 0, "variant": 0, "value": 2100500, "sortReverse": 0 },
{ "attribute": 5053, "language": 0, "variant": 0, "value": 2100501, "sortReverse": 1 },
{ "attribute": 5053, "language": 0, "variant": 0, "value": 2100502, "sortReverse": 2 }
```

The same pattern applies to all multi-value ObjRef attributes: 5002 (detail forms), 5055 (condition refs), 6001 (app area types).

---

## Form ↔ Object Type Linkage

Forms are attached to object types via two attributes on the type (typeRef=0):

| Role | Attribute | ID | Cardinality |
|------|-----------|------|-------------|
| Create form | `ATTR_OBJ_TYPE_CREATE_FORM` | 5001 | Single ObjRef |
| Detail forms (tabs) | `ATTR_OBJ_TYPE_DETAIL_FORMS` | 5002 | Multi-value ObjRef |

> When inspecting form linkage for a single type, prefer `objecttype_describe` — it returns `createFormId` and `detailFormIds` directly.

### Set create form

```json
{
  "branchId": 2100347,
  "objectId": <typeId>,
  "values": "[{\"attribute\":5001,\"language\":0,\"variant\":0,\"value\":<formId>}]"
}
```

Tool: `object_update`.

### Add detail form

Read existing detail form ids first (via `objecttype_describe`), then send the complete list:

```json
{
  "branchId": 2100347,
  "objectId": <typeId>,
  "values": "[{\"attribute\":5002,\"language\":0,\"variant\":0,\"value\":[<existingFormId>,<newFormId>]}]"
}
```

Tool: `object_update`.

---

## Workflows

### Inspect a form

1. Fetch the form via `object_get`:
   ```json
   {
     "branchId": 2100347,
     "objectIds": [<formId>],
     "languages": [201, 202],
     "attributes": []
   }
   ```
2. Extract field IDs from values where `attribute === 5053`. Sort by `sortReverse` ascending.
3. Resolve attribute definitions via `object_get`:
   ```json
   {
     "branchId": 2100347,
     "objectIds": [<attrDefId1>, <attrDefId2>, <attrDefId3>],
     "languages": [201, 202],
     "attributes": [1000, 1001, 1017, 1018]
   }
   ```
4. Present as table: Field Order, Name, Data Type, Required, Language-Dependent.

### Set fields (full replacement)

Send ALL 5053 entries — omitted fields are removed:

```json
{
  "branchId": 2100347,
  "objectId": <formId>,
  "values": "[{\"attribute\":5053,\"language\":0,\"variant\":0,\"value\":[<attrDefId1>,<attrDefId2>]}]"
}
```

Tool: `object_update`. Response: `{ objectId, updatedObjects, createdValues, transaction }`.

### Add a field

1. Read the form via `object_get`.
2. Extract existing 5053 values sorted by `sortReverse`.
3. Append the new field ID to the array.
4. Send the complete array via `object_update`.

### Remove a field

1. Read the form via `object_get`.
2. Extract existing 5053 values sorted by `sortReverse`.
3. Filter out the field to remove.
4. Send the complete shorter array via `object_update`.

### Create a new form

```json
{
  "branchId": 2100347,
  "objectTypeId": 50,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"Form Name EN\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Formularname DE\"},{\"attribute\":5056,\"language\":0,\"variant\":0,\"value\":false},{\"attribute\":5053,\"language\":0,\"variant\":0,\"value\":[<attrDefId1>,<attrDefId2>]}]"
}
```

Tool: `object_create`. Response: `{ createdObjectId, transaction }`.

After creation, fetch with `object_get` to confirm.

### Link form to type

See the Form ↔ Object Type Linkage section above. Use `object_update` to set attribute 5001 (create form) or 5002 (detail forms) on the type.

---

## Safety: Read Before Write

**Omitting 5053 values in an update removes all fields.** Always:

1. Read the current form state before modifying
2. Merge your changes with the existing values
3. Send the complete value set
4. Verify after write — re-fetch and confirm

---

## Condition System

Forms can be conditionally visible based on attribute values. The condition types (typeRef 170–176) define when a form tab appears:

| Attribute | ID | Purpose |
|-----------|------|---------|
| Condition attribute | 5054 | Which attribute to evaluate (single ObjRef) |
| Condition refs | 5055 | Which values trigger visibility (multi-value ObjRef) |

When attribute 5054 is set on a form, the form tab only appears if the object's value for that attribute matches one of the 5055 references.

---

## CmsValue Structure (inside the JSON string)

```typescript
{
  attribute: number,     // Attribute definition ID
  language: number,      // 201=EN, 202=DE, 0=language-independent
  variant: number,       // 0=default
  value: unknown,        // Scalar OR array for multi-value (auto-expanded)
  sortReverse?: number   // Optional explicit multi-value ordering
}
```

---

## API Response Formats

| Method | Response |
|--------|----------|
| `object_create` | `{ createdObjectId, transaction }` |
| `object_update` | `{ objectId, updatedObjects, createdValues, transaction }` |
| `object_delete` | `{ deletedObjects, transaction }` |
| `object_get` | `ObjectQueryResult` wrapping an array of `CmsObject`s |
| `object_query` | `ObjectQueryResult` with pagination |
| `objecttype_describe` | Structured `ObjectTypeDescription` |
