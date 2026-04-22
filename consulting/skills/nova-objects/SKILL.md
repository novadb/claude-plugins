---
name: nova-objects
description: "Data object CRUD reference — schema discovery, create/update/delete workflows, value formats, and multi-value safety patterns."
user-invocable: false
allowed-tools: object_get, object_query, object_create, object_update, object_delete, object_count, object_xmllinkcount, objecttype_describe
---

# NovaDB Data Object Management Reference

## Scope

**This skill is EXCLUSIVELY a reference for:** Creating, reading, updating, and deleting NovaDB business data objects (typeRef >= 2097152) with schema-aware safety checks.

**For meta-/schema objects (typeRef < 8192)** → see `nova-schema` skill
**For schema browsing** → see `nova-explore` skill
**For form configuration** → see `nova-forms` skill
**For search only** → see `nova-search` skill

Complete reference for data object CRUD operations, schema discovery, value formats, and safety patterns.

> **IMPORTANT:** This skill is for business data objects only. Before every write operation, verify the target object has `typeRef >= 2097152`. If the typeRef is < 8192, the object is a meta/schema object — refuse the operation and direct the user to the nova-schema agent.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All numeric IDs in examples are samples — always use real IDs from the target system.

---

## Schema Discovery Workflow

**Before creating or updating any object, discover the type structure.** Never guess attribute IDs or data types.

### One-shot: `objecttype_describe`

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "languages": [201, 202]
}
```

Returns `{ id, apiIdentifier, name, description, displayNameAttrId, createFormId, detailFormIds, attributes[] }`. Each attribute carries `{ dataType, multiValue, languageDependent, variantDependent, defaultForm, apiIdentifier }`.

That is usually enough. Only fall back to the raw chain if you need something not in the description.

### Raw chain (fallback)

```
Object Type (typeRef=0)
  → Create Form (attr 5001) → Form Content (attr 5053) → Attribute Definitions
  → Detail Forms (attr 5002) → Form Content (attr 5053) → Attribute Definitions
```

Use `object_get` hops. Key attribute ids:

| Attribute | ID | Type | Purpose |
|-----------|------|------|---------|
| Name | 1000 | String | Display name (language-dependent: 201=EN, 202=DE) |
| Data type | 1001 | String.DataType | Value format |
| Allow multiple | 1004 | Boolean | `true` = multi-value attribute |
| Allowed types | 1015 | ObjRef | For ObjRef attrs: valid target types |
| Language dependent | 1017 | Boolean | Separate values per language |
| Required | 1018 | Boolean | Must have a value |

---

## Create Workflow

### 1. Discover the schema (see above)

### 2. Build the values payload (JSON-encoded string)

Each entry is a `CmsValue`. For language-dependent attributes, send one entry per language. For multi-value attributes, either pass an array inside `value` (preferred) or individual entries with `sortReverse`.

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"Name EN\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Name DE\"},{\"attribute\":<attrDefId>,\"language\":0,\"variant\":0,\"value\":\"<value>\"}]"
}
```

Tool: `object_create`. Response: `{ createdObjectId, transaction }`.

> `object_create` creates **one** object per call. Sequential creates are not atomic — plan for partial state.

### 3. Verify

```json
{
  "branchId": 2100347,
  "objectIds": [<newId>],
  "languages": [201, 202],
  "attributes": []
}
```

Tool: `object_get`.

---

## Value Format Table by DataType

| DataType | JSON value type | Example | Notes |
|----------|----------------|---------|-------|
| `String` | string | `"Hello World"` | |
| `TextRef` | string | `"Long text content..."` | |
| `TextRef.JavaScript` | string | `"function() { ... }"` | |
| `TextRef.CSS` | string | `".class { color: red; }"` | |
| `XmlRef.SimpleHtml` | string | `"<div>HTML content</div>"` | XHTML with `<div>` root |
| `XmlRef.VisualDocument` | string | `"<div>...</div>"` | Rich document XHTML |
| `Integer` | number | `42` | |
| `Decimal` | number | `3.14` | |
| `Float` | number | `1.5` | |
| `Boolean` | boolean | `true` | |
| `DateTime` | string | `"2025-06-15T10:30:00Z"` | ISO 8601 |
| `DateTime.Date` | string | `"2025-06-15"` | Date only |
| `ObjRef` | number | `2100500` | Object ID |
| `BinRef` | number | `2100600` | Binary object ID |
| `String.DataType` | string | `"String"` | One of the DataType enum values |
| `String.UserName` | string | `"admin"` | NovaDB username |
| `String.RGBColor` | string | `"#FF0000"` | Hex color |

---

## Update Workflow

### 1. Read the current object

```json
{
  "branchId": 2100347,
  "objectIds": [<objectId>],
  "languages": [201, 202],
  "attributes": []
}
```

Tool: `object_get`.

### 2. Merge changes

- **Single-value attributes:** Can be sent in isolation — only the specified attributes are changed (read-then-merge semantics).
- **Multi-value attributes (1004=true):** Must send the COMPLETE value set. Omitted entries are deleted.

### Multi-Value Safety Pattern

```
1. Read the object → extract all values for the multi-value attribute
2. Merge: add/remove/reorder values as needed
3. Send the complete set (array-in-value is cleanest)
```

### 3. Send the update

```json
{
  "branchId": 2100347,
  "objectId": <objectId>,
  "values": "[{\"attribute\":<attrDefId>,\"language\":0,\"variant\":0,\"value\":\"<newValue>\"}]"
}
```

Tool: `object_update`. Response: `{ objectId, updatedObjects, createdValues, transaction }`.

### 4. Verify

Re-read the object and confirm the changes.

### Multi-Value ObjRef Example

Updating a multi-value ObjRef attribute:

```json
{
  "branchId": 2100347,
  "objectId": <objectId>,
  "values": "[{\"attribute\":<attrDefId>,\"language\":0,\"variant\":0,\"value\":[2100500,2100501,2100502]}]"
}
```

Array order = display order. The MCP auto-expands this to sorted entries.

---

## Delete Workflow

### 1. Check references

```json
{
  "branchId": 2100347,
  "objectIds": [<objectId>]
}
```

Tool: `object_xmllinkcount`.

If the count is > 0, warn the user about existing references.

### 2. Confirm with user

Show the object that will be deleted (name, type, ID) and any reference counts. Ask the user to confirm.

### 3. Delete

```json
{
  "branchId": 2100347,
  "objectIds": [<objectId>],
  "comment": "reason"
}
```

Tool: `object_delete`. Response: `{ deletedObjects, transaction }`.

> `objectIds` is an int[], but per the safety rules, confirm each schema-level deletion individually.

### 4. Verify

Confirm deletion by querying for the object with `isDeleted` filter.

---

## Read / Inspect

### Fetch a single object

Use `object_get` with a one-element `objectIds` array (see §Update step 1).

### Fetch multiple objects

Pass the full id list in one `object_get` call.

### ObjRef Resolution Pattern

1. Collect all unique ObjRef IDs
2. Batch-fetch with `object_get`, `attributes: [1000]`, `languages: [201, 202]`
3. Match `meta.id` to the ObjRef values
4. Display resolved names instead of bare IDs

---

## Finding Objects via `object_query`

### Search

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "languages": [201, 202],
  "searchPhrase": "<query>",
  "skip": 0,
  "take": 20
}
```

### Count

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "searchPhrase": "<query>"
}
```

Tool: `object_count`.

### Attribute-Level Filter

`filters` is an array of `"<attrId>:<langId>:<variantId>:<operator>:<value>"` strings.

| Operator | Meaning |
|----------|---------|
| 0 | Equal |
| 1 | NotEqual |
| 2 | GreaterThan |
| 3 | LessThan |
| 4 | Like |
| 7 | ObjRefLookup |

Example — all objects of a type with `required = true`:

```json
{ "filters": ["1018:0:0:0:true"] }
```

> The old Index API extras (`suggestions`, `match_strings`, `object_occurrences`) are not available in the C# MCP. Aggregate client-side if you need counts-per-something.

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

| Operation | Response |
|-----------|----------|
| `object_create` | `{ createdObjectId, transaction }` |
| `object_update` | `{ objectId, updatedObjects, createdValues, transaction }` |
| `object_delete` | `{ deletedObjects, transaction }` |
| `object_get` | `ObjectQueryResult` |
| `object_query` | `ObjectQueryResult` with pagination |
| `object_count` | `{ count }` |
| `object_xmllinkcount` | `{ count }` |
| `objecttype_describe` | `ObjectTypeDescription` |

---

## Branch Parameter

- All object tools take `branchId` as an **int**. No named identifiers.

---

## Application Area Discovery

To find object types by domain or theme:

1. `object_query` with `objectTypeId: 60`, `searchPhrase: "<theme>"`
2. `object_get` on the matching App Area with `attributes: [6001]`
3. Extract type IDs from 6001 values
4. `objecttype_describe` for each
