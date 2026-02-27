---
name: nova-objects
description: "Data object CRUD reference — schema discovery, create/update/delete workflows, value formats, and multi-value safety patterns."
user-invocable: false
allowed-tools: novadb_cms_get_object, novadb_cms_get_objects, novadb_cms_get_typed_objects, novadb_cms_create_objects, novadb_cms_update_objects, novadb_cms_delete_objects, novadb_index_search_objects, novadb_index_count_objects, novadb_index_object_occurrences, novadb_index_object_xml_link_count, novadb_index_suggestions
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

### Step 1: Get the Object Type

Fetch the type to find its forms:

```json
{
  "branch": "<branch>",
  "id": "<typeId>",
  "inherited": true,
  "attributes": "1000,5001,5002,5005"
}
```

Tool: `novadb_cms_get_object`

Key attributes on the type:
- **5001** — Create Form (single ObjRef → form ID)
- **5002** — Detail Forms (multi-value ObjRef → form IDs)
- **5005** — Display Name Attribute (which attribute serves as the object's display name)

### Step 2: Get the Form(s)

Fetch the create form (and optionally detail forms) to find the fields:

```json
{
  "branch": "<branch>",
  "ids": "<createFormId>,<detailFormId1>,<detailFormId2>",
  "inherited": true,
  "attributes": "1000,5053"
}
```

Tool: `novadb_cms_get_objects`

Extract attribute definition IDs from **5053** values (sorted by `sortReverse`).

### Step 3: Get the Attribute Definitions

Fetch all attribute definitions to learn their data types, constraints, and flags:

```json
{
  "branch": "<branch>",
  "ids": "<attrDefId1>,<attrDefId2>,<attrDefId3>",
  "inherited": true,
  "attributes": "1000,1001,1004,1015,1017,1018"
}
```

Tool: `novadb_cms_get_objects`

### Key Attribute Definition Attributes

| Attribute | ID | Type | Purpose |
|-----------|------|------|---------|
| Name | 1000 | String | Display name (language-dependent: 201=EN, 202=DE) |
| Data type | 1001 | String.DataType | The value format (see Value Format Table) |
| Allow multiple | 1004 | Boolean | `true` = multi-value attribute |
| Allowed types | 1015 | ObjRef | For ObjRef attrs: which object types are valid targets (multi-value) |
| Language dependent | 1017 | Boolean | `true` = separate values per language |
| Required | 1018 | Boolean | `true` = must have a value |

### Complete Discovery Chain

```
Object Type (typeRef=0)
  → Create Form (attr 5001) → Form Content (attr 5053) → Attribute Definitions
  → Detail Forms (attr 5002) → Form Content (attr 5053) → Attribute Definitions
```

---

## Create Workflow

### 1. Discover the schema (see above)

### 2. Build the values array

Each value is a `CmsValue` entry. For language-dependent attributes, send one entry per language. For multi-value attributes, send one entry per value with `sortReverse` ordering.

```json
{
  "branch": "<branch>",
  "objects": [{
    "meta": { "typeRef": "<typeId>" },
    "values": [
      { "attribute": 1000, "language": 201, "variant": 0, "value": "Name EN" },
      { "attribute": 1000, "language": 202, "variant": 0, "value": "Name DE" },
      { "attribute": "<attrDefId>", "language": 0, "variant": 0, "value": "<value>" }
    ]
  }]
}
```

Tool: `novadb_cms_create_objects`

Response: `{ transaction, createdObjectIds: [<newId>] }`

### 3. Verify

Fetch the created object to confirm:

```json
{
  "branch": "<branch>",
  "id": "<newId>",
  "inherited": true
}
```

Tool: `novadb_cms_get_object`

---

## Value Format Table by DataType

| DataType | JSON value type | Example | Notes |
|----------|----------------|---------|-------|
| `String` | string | `"Hello World"` | |
| `TextRef` | string | `"Long text content..."` | For longer text |
| `TextRef.JavaScript` | string | `"function() { ... }"` | JavaScript code |
| `TextRef.CSS` | string | `".class { color: red; }"` | CSS code |
| `XmlRef.SimpleHtml` | string | `"<div>HTML content</div>"` | XHTML with `<div>` root |
| `XmlRef.VisualDocument` | string | `"<div>...</div>"` | Rich document XHTML |
| `Integer` | number | `42` | Whole numbers |
| `Decimal` | number | `3.14` | Decimal numbers |
| `Float` | number | `1.5` | Floating point |
| `Boolean` | boolean | `true` | `true` or `false` |
| `DateTime` | string | `"2025-06-15T10:30:00Z"` | ISO 8601 format |
| `DateTime.Date` | string | `"2025-06-15"` | Date only |
| `ObjRef` | number | `2100500` | Object ID reference |
| `BinRef` | number | `2100600` | Binary object ID reference |
| `String.DataType` | string | `"String"` | One of the DataType enum values |
| `String.UserName` | string | `"admin"` | NovaDB username |
| `String.RGBColor` | string | `"#FF0000"` | Hex color code |

---

## Update Workflow

### 1. Read the current object

**Always read before writing** to get current values and avoid accidental data loss:

```json
{
  "branch": "<branch>",
  "id": "<objectId>",
  "inherited": true
}
```

Tool: `novadb_cms_get_object`

### 2. Merge changes

- **Single-value attributes:** Can be sent in isolation — only the specified attributes are changed.
- **Multi-value attributes (1004=true):** Must send the COMPLETE value set. Omitted entries are deleted.

### Multi-Value Safety Pattern

```
1. Read the object → extract all values for the multi-value attribute
2. Merge: add/remove/reorder values as needed
3. Rebuild sortReverse (0, 1, 2, ...)
4. Send the complete set in the update
```

### 3. Send the update

```json
{
  "branch": "<branch>",
  "objects": [{
    "meta": { "id": "<objectId>", "typeRef": "<typeId>" },
    "values": [
      { "attribute": "<attrDefId>", "language": 0, "variant": 0, "value": "<newValue>" }
    ]
  }]
}
```

Tool: `novadb_cms_update_objects`

Response: `{ updatedObjects, createdValues, transaction }`

### 4. Verify

Re-read the object and confirm the changes:

```json
{
  "branch": "<branch>",
  "id": "<objectId>",
  "inherited": true
}
```

Tool: `novadb_cms_get_object`

### Multi-Value ObjRef Example

Updating a multi-value ObjRef attribute (e.g. adding a reference):

```json
{
  "branch": "<branch>",
  "objects": [{
    "meta": { "id": "<objectId>", "typeRef": "<typeId>" },
    "values": [
      { "attribute": "<attrDefId>", "language": 0, "variant": 0, "value": 2100500, "sortReverse": 0 },
      { "attribute": "<attrDefId>", "language": 0, "variant": 0, "value": 2100501, "sortReverse": 1 },
      { "attribute": "<attrDefId>", "language": 0, "variant": 0, "value": 2100502, "sortReverse": 2 }
    ]
  }]
}
```

Each value gets its own CmsValue entry with `sortReverse` for ordering. Never use arrays as values.

---

## Delete Workflow

### 1. Check references

Before deleting, check if other objects reference this object via XML attributes:

```json
{
  "branch": "<branchNumericId>",
  "objectIds": [<objectId>]
}
```

Tool: `novadb_index_object_xml_link_count`

If the count is > 0, warn the user about existing references.

### 2. Confirm with user

Show the object that will be deleted (name, type, ID) and any reference counts. Ask the user to confirm.

### 3. Delete (soft-delete)

```json
{
  "branch": "<branch>",
  "objectIds": ["<objectId>"]
}
```

Tool: `novadb_cms_delete_objects`

Response: `{ deletedObjects, transaction }`

**Note:** This is a soft-delete — objects can be restored. The `objectIds` parameter accepts strings (IDs, GUIDs, or ApiIdentifiers).

### 4. Verify

Confirm deletion by searching for the object or fetching with `deleted` filter.

---

## Read / Inspect Object

### Fetch a single object

```json
{
  "branch": "<branch>",
  "id": "<objectId>",
  "inherited": true
}
```

Tool: `novadb_cms_get_object`

### Fetch multiple objects

```json
{
  "branch": "<branch>",
  "ids": "<id1>,<id2>,<id3>",
  "inherited": true
}
```

Tool: `novadb_cms_get_objects`

### ObjRef Resolution Pattern

When values contain numeric ObjRef references:

1. Collect all unique ObjRef IDs
2. Batch-fetch: `novadb_cms_get_objects` with `ids` and `attributes: "1000"`, `inherited: true`
3. Match `meta.id` to the ObjRef values
4. Display resolved names instead of bare IDs

---

## Finding Objects via Index API

### Search

```json
{
  "branch": "<numericBranchId>",
  "filter": {
    "objectTypeIds": [<typeId>],
    "searchPhrase": "<query>"
  },
  "sortBy": [{ "sortBy": 3 }],
  "take": 20
}
```

Tool: `novadb_index_search_objects`

### Count

```json
{
  "branch": "<numericBranchId>",
  "filter": {
    "objectTypeIds": [<typeId>]
  }
}
```

Tool: `novadb_index_count_objects`

### Suggestions (autocomplete)

```json
{
  "branch": "<numericBranchId>",
  "pattern": "<partial>",
  "filter": {
    "objectTypeIds": [<typeId>]
  },
  "suggestDisplayName": true,
  "take": 10
}
```

Tool: `novadb_index_suggestions`

### Attribute-Level Filter

```json
{
  "filter": {
    "objectTypeIds": [<typeId>],
    "filters": [{
      "attrId": <attrDefId>,
      "langId": 0,
      "variantId": 0,
      "value": "<filterValue>",
      "compareOperator": 0
    }]
  }
}
```

**compareOperator values:** 0=Equal, 1=NotEqual, 2=GreaterThan, 3=LessThan, 4=Like, 7=ObjRefLookup

---

## CmsValue Structure

```typescript
{
  attribute: number,   // Attribute definition ID
  language: number,    // 201=EN, 202=DE, 0=language-independent
  variant: number,     // 0=default
  value: unknown,      // The actual value (type depends on DataType)
  sortReverse?: number // Multi-value ordering (0, 1, 2, ...)
}
```

---

## API Response Formats

| Operation | Response |
|-----------|----------|
| Create (`cms_create_objects`) | `{ transaction, createdObjectIds: number[] }` |
| Update (`cms_update_objects`) | `{ updatedObjects, createdValues, transaction }` |
| Delete (`cms_delete_objects`) | `{ deletedObjects, transaction }` |
| Get single (`cms_get_object`) | `CmsObject` with `meta` and `values` |
| Get multiple (`cms_get_objects`) | `{ objects: CmsObject[] }` |
| Get typed (`cms_get_typed_objects`) | `{ objects: CmsObject[], continue? }` |
| Index search | `{ objects: [...], more, changeTrackingVersion }` |
| Index count | `{ count }` |

---

## Branch Parameter

- **CMS API:** Accepts branch ID or `"draft"`.
- **Index API:** Requires a **numeric branch ID string** (e.g. `"2100347"`). Never use `"draft"` or `"branchDefault"`.

---

## Application Area Discovery

To find object types by domain or theme:

1. Search Application Areas: `objectTypeIds: [60]`, `searchPhrase: "<theme>"`
2. Fetch the App Area with `novadb_cms_get_object` (include attr 6001)
3. Extract type IDs from attribute 6001 values
4. Fetch types with `novadb_cms_get_objects`
