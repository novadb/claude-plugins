---
name: nova-schema
description: "Meta-object CRUD reference — type catalog, schema discovery, create/update/delete workflows, impact analysis, and safety patterns for all objects with typeRef < 8192."
user-invocable: false
allowed-tools: novadb_cms_get_object, novadb_cms_get_objects, novadb_cms_get_typed_objects, novadb_cms_create_objects, novadb_cms_update_objects, novadb_cms_delete_objects, novadb_index_search_objects, novadb_index_count_objects, novadb_index_object_occurrences, novadb_index_object_xml_link_count, novadb_index_suggestions
---

# NovaDB Schema/Meta-Object Management Reference

## Scope

**This skill is EXCLUSIVELY a reference for:** Creating, reading, updating, and deleting NovaDB meta/schema objects (typeRef < 8192) with strict safety checks.

**For business data objects** → see `nova-objects` skill
**For read-only schema browsing** → see `nova-explore` skill
**For form configuration** → see `nova-forms` skill
**For search only** → see `nova-search` skill

> **WARNING:** Schema objects define the entire CMS/PIM structure. A wrong change can break the system for all users. Every write operation requires explicit user confirmation.

---

## Meta-Type Catalog

All meta/schema types have `typeRef < 8192`. Here is the complete catalog:

| typeRef | Name | Protection Level | Description |
|---------|------|-----------------|-------------|
| 0 | Object Type | CRITICAL | Defines data object types. Core types (ID < 500) are read-only. |
| 10 | Attribute Definition | CRITICAL | Defines attributes available on objects. Core attrs (ID < 500) are read-only. |
| 11 | Attribute Group | Normal | Groups related attributes for UI organization. |
| 12 | Attribute Group Item | Normal | Links an attribute to a group. |
| 20 | Language | PROTECTED | Defines available languages. Instances with ID < 500 are read-only. |
| 30 | Unit Category | Normal | Groups units of measure (e.g. "Length", "Weight"). |
| 31 | Unit of Measure | Normal | Specific units (e.g. "cm", "kg"). |
| 50 | Form | CRITICAL | Defines UI forms for object types. Controls what users see and edit. |
| 60 | Application Area | Normal | Groups object types by domain/theme. |
| 70 | UI String | PROTECTED | UI labels and translations. Instances with ID < 500 are read-only. |
| 80 | Workflow State | Normal | Defines workflow states for branches. |
| 90 | Kanban Board | Normal | Kanban board configuration. |
| 100 | Media Type | PROTECTED | Defines allowed media/file types. Instances with ID < 500 are read-only. |
| 120 | Job Definition | Normal | Defines server-side processing jobs. |
| 130 | Package | Normal | Groups related schema objects for export/import. |
| 140 | Tree Definition | Normal | Defines tree navigation structures. |
| 150 | Razor View | Normal | Server-side Razor view templates. |
| 155 | Parameter | Normal | Configuration parameters. |
| 160 | Trigger | Normal | Event-driven automation triggers. |
| 161 | Timer | Normal | Time-based automation triggers. |
| 170 | Condition | Normal | General conditions. |
| 171 | Condition Group | Normal | Groups of conditions. |
| 172 | Field Condition | Normal | Field-level conditions. |
| 173 | State Condition | Normal | Workflow state conditions. |
| 174 | Type Condition | Normal | Object type conditions. |
| 175 | Script Condition | Normal | Script-based conditions. |
| 176 | Branch Condition | Normal | Branch-level conditions. |
| 180 | Branch Type | Normal | Defines branch types with workflow rules. |
| 190 | Presentation Type | Normal | Defines output/presentation formats. |
| 200 | External API | Normal | External API endpoint definitions. |
| 210 | WebHook | Normal | WebHook configuration. |
| 300 | Style Sheet | Normal | CSS styling definitions. |
| 301 | Style Rule | Normal | Individual CSS rules. |
| 302 | Style Variable | Normal | CSS variables/tokens. |
| 303 | Style Set | Normal | Groups of style rules. |
| 304 | Style Theme | Normal | Complete style themes. |
| 400 | Visual Component | Normal | UI component definitions. |
| 401 | Visual Component Variant | Normal | Variants of visual components. |
| 402 | Visual Component Slot | Normal | Slots within visual components. |

**Protection Levels:**
- **CRITICAL**: Core infrastructure. Instances with ID < 500 are absolutely read-only. Changes to higher-ID instances need careful impact analysis.
- **PROTECTED**: Instances with ID < 500 are read-only. Higher-ID instances can be modified with caution.
- **Normal**: Can be created, updated, and deleted with standard safety checks.

**Special range:** Built-in media types (ID 2098252–2098404) are system-provided and should be treated as read-only even though their IDs are high.

---

## Key Attributes per Meta-Type

### Object Type (typeRef=0)

| Attribute | ID | Type | Purpose |
|-----------|------|------|---------|
| Name | 1000 | String | Display name (language-dependent: 201=EN, 202=DE) |
| Description | 1020 | TextRef | Type description |
| Create Form | 5001 | ObjRef | Form used when creating new objects |
| Detail Forms | 5002 | ObjRef (multi) | Forms shown as tabs on existing objects |
| Display Name Attr | 5005 | ObjRef | Which attribute serves as display name |
| Icon | 5010 | String | Icon identifier |
| API Identifier | 5015 | String | Programmatic identifier |

### Attribute Definition (typeRef=10)

| Attribute | ID | Type | Purpose |
|-----------|------|------|---------|
| Name | 1000 | String | Display name (language-dependent) |
| Data Type | 1001 | String.DataType | Value format (String, Integer, ObjRef, etc.) |
| Description | 1002 | String | Attribute description |
| Unit of Measure | 1003 | ObjRef | Unit category reference |
| Allow Multiple | 1004 | Boolean | Multi-value attribute |
| Allowed Types | 1015 | ObjRef (multi) | For ObjRef: valid target types |
| Language Dependent | 1017 | Boolean | Separate values per language |
| Required | 1018 | Boolean | Must have a value |
| Read Only | 1019 | Boolean | Cannot be edited by users |
| Validation Code | 1030 | TextRef.JavaScript | Client-side validation |
| Virtualization Code | 1031 | TextRef.JavaScript | Computed/virtual value |
| API Identifier | 1050 | String | Programmatic identifier |

### Form (typeRef=50)

| Attribute | ID | Type | Purpose |
|-----------|------|------|---------|
| Name | 1000 | String | Display name (language-dependent) |
| Form Fields | 5053 | ObjRef (multi) | Ordered list of attribute definitions shown on this form |
| Conditional Visibility | 5054 | TextRef.JavaScript | JS code controlling field visibility |
| Tab Label | 5055 | String | Label when used as detail tab |

### Application Area (typeRef=60)

| Attribute | ID | Type | Purpose |
|-----------|------|------|---------|
| Name | 1000 | String | Display name (language-dependent) |
| Object Types | 6001 | ObjRef (multi) | Types belonging to this area |
| Icon | 6002 | String | Area icon |

---

## Schema Discovery Workflow

### Discovering Meta-Object Structure

Before modifying any meta-object, discover its structure:

**Step 1: Identify the meta-type**

Determine the `typeRef` of what you want to work with (see Meta-Type Catalog above).

**Step 2: List existing instances**

```json
{
  "branch": "<branch>",
  "type": "<typeRef>",
  "inherited": true,
  "attributes": "1000",
  "take": 20
}
```

Tool: `novadb_cms_get_typed_objects`

**Step 3: Inspect a specific instance**

```json
{
  "branch": "<branch>",
  "id": "<objectId>",
  "inherited": true
}
```

Tool: `novadb_cms_get_object`

### Finding Which Types Use an Attribute

To find which object types reference a given attribute definition:

1. List all forms (typeRef=50) and fetch their field lists (attr 5053)
2. Check which forms include the attribute ID
3. Find which types reference those forms via attrs 5001/5002

Or use the Index API:

```json
{
  "branch": "<numericBranchId>",
  "filter": {
    "objectTypeIds": [50],
    "filters": [{
      "attrId": 5053,
      "value": "<attrDefId>",
      "compareOperator": 0
    }]
  },
  "take": 50
}
```

Tool: `novadb_index_search_objects`

### Finding Forms for an Object Type

```json
{
  "branch": "<branch>",
  "id": "<typeId>",
  "inherited": true,
  "attributes": "1000,5001,5002"
}
```

Tool: `novadb_cms_get_object`

Extract form IDs from attributes 5001 (create form) and 5002 (detail forms).

---

## Impact Analysis Workflows

### Before Modifying an Attribute Definition

1. **Find all forms using this attribute:**

```json
{
  "branch": "<numericBranchId>",
  "filter": {
    "objectTypeIds": [50],
    "filters": [{
      "attrId": 5053,
      "value": "<attrDefId>",
      "compareOperator": 0
    }]
  },
  "take": 50
}
```

Tool: `novadb_index_search_objects`

2. **Find all object types using those forms:**

For each form found, search which types reference it via attrs 5001 or 5002.

3. **Count data objects of affected types:**

```json
{
  "branch": "<numericBranchId>",
  "filter": {
    "objectTypeIds": [<affectedTypeId1>, <affectedTypeId2>]
  }
}
```

Tool: `novadb_index_count_objects`

4. **Report to user:** Show the list of affected forms, types, and data object counts before proceeding.

### Before Deleting a Schema Object

1. **Check XML references:**

```json
{
  "branch": "<numericBranchId>",
  "objectIds": [<objectId>]
}
```

Tool: `novadb_index_object_xml_link_count`

2. **Type-specific checks:**
   - **Deleting an Object Type (0):** Count data objects of this type. Check forms referencing this type.
   - **Deleting an Attribute Definition (10):** Check which forms include this attribute.
   - **Deleting a Form (50):** Check which types reference this form via 5001/5002.
   - **Deleting an Application Area (60):** List all types in this area (attr 6001).

3. **Report all dependencies to the user and get explicit confirmation.**

---

## Create Workflow

### 1. Verify the meta-type exists

Confirm the target `typeRef` is valid and < 8192.

### 2. Build the values array

Each value is a `CmsValue` entry. For language-dependent attributes, send one entry per language. For multi-value attributes, send one entry per value with `sortReverse` ordering.

```json
{
  "branch": "<branch>",
  "objects": [{
    "meta": { "typeRef": <metaTypeRef> },
    "values": [
      { "attribute": 1000, "language": 201, "variant": 0, "value": "Name EN" },
      { "attribute": 1000, "language": 202, "variant": 0, "value": "Name DE" },
      { "attribute": <attrId>, "language": 0, "variant": 0, "value": "<value>" }
    ]
  }]
}
```

Tool: `novadb_cms_create_objects`

### 3. Show the user what will be created and get confirmation

Display: typeRef name, all values, expected impact. Wait for user approval.

### 4. Execute and verify

After creation, fetch the new object and show the result:

```json
{
  "branch": "<branch>",
  "id": "<newId>",
  "inherited": true
}
```

Tool: `novadb_cms_get_object`

---

## Update Workflow

### 1. Read the current state

```json
{
  "branch": "<branch>",
  "id": "<objectId>",
  "inherited": true
}
```

Tool: `novadb_cms_get_object`

### 2. Verify typeRef < 8192

Check the object's `meta.typeRef`. If >= 8192, refuse the operation and direct the user to the nova-objects agent.

### 3. Verify ID >= 500

If the object ID < 500, refuse the operation — these are core system objects.

### 4. Run impact analysis

For attribute definitions and forms, check what depends on this object (see Impact Analysis Workflows above).

### 5. Merge changes

- **Single-value attributes:** Can be sent in isolation.
- **Multi-value attributes (1004=true):** Must send the COMPLETE value set. Omitted entries are deleted.

### Multi-Value Safety Pattern

```
1. Read the object → extract all values for the multi-value attribute
2. Merge: add/remove/reorder values as needed
3. Rebuild sortReverse (0, 1, 2, ...)
4. Send the complete set in the update
```

### 6. Show changes and get confirmation

Display: current values vs. new values, affected dependents. Wait for user approval.

### 7. Execute and verify

```json
{
  "branch": "<branch>",
  "objects": [{
    "meta": { "id": <objectId>, "typeRef": <typeRef> },
    "values": [
      { "attribute": <attrId>, "language": 0, "variant": 0, "value": "<newValue>" }
    ]
  }]
}
```

Tool: `novadb_cms_update_objects`

Re-read and show the result to the user.

---

## Delete Workflow

### 1. Read the target object

Fetch and verify: `typeRef < 8192`, `ID >= 500`.

### 2. Run impact analysis

See "Before Deleting a Schema Object" in Impact Analysis Workflows.

### 3. Check XML references

```json
{
  "branch": "<numericBranchId>",
  "objectIds": [<objectId>]
}
```

Tool: `novadb_index_object_xml_link_count`

### 4. Show full impact report and get confirmation

Display: object name, type, ID, all dependencies, reference counts. Only proceed with explicit user approval.

### 5. Delete (one at a time)

```json
{
  "branch": "<branch>",
  "objectIds": ["<objectId>"]
}
```

Tool: `novadb_cms_delete_objects`

**Maximum 1 object per delete call.** Each deletion must be individually confirmed.

### 6. Verify

Confirm the object is deleted.

---

## Value Format Table by DataType

| DataType | JSON value type | Example | Notes |
|----------|----------------|---------|-------|
| `String` | string | `"Hello World"` | |
| `TextRef` | string | `"Long text content..."` | For longer text |
| `TextRef.JavaScript` | string | `"function() { ... }"` | JavaScript code |
| `TextRef.CSS` | string | `".class { color: red; }"` | CSS code |
| `XmlRef.SimpleHtml` | string | `"<div>HTML content</div>"` | XHTML with `<div>` root |
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

- **CMS API:** Requires a **numeric branch ID** (int32).
- **Index API:** Requires a **numeric branch ID string** (e.g. `"2100347"`). Never use `"branchDefault"`.

> **Recommendation:** Schema changes should be made on a named branch. This allows review and rollback.

---

## Common Operations

### Create a new Attribute Definition

```json
{
  "branch": "<branch>",
  "objects": [{
    "meta": { "typeRef": 10 },
    "values": [
      { "attribute": 1000, "language": 201, "variant": 0, "value": "My Attribute EN" },
      { "attribute": 1000, "language": 202, "variant": 0, "value": "Mein Attribut DE" },
      { "attribute": 1001, "language": 0, "variant": 0, "value": "String" },
      { "attribute": 1004, "language": 0, "variant": 0, "value": false },
      { "attribute": 1017, "language": 0, "variant": 0, "value": true },
      { "attribute": 1018, "language": 0, "variant": 0, "value": false }
    ]
  }]
}
```

### Add an Attribute to a Form

1. Read the form to get current fields (attr 5053)
2. Append the new attribute ID to the list
3. Rebuild sortReverse values (0, 1, 2, ...)
4. Update with the complete field list

### Create a new Object Type with Form

1. Create the object type (typeRef=0) with name
2. Create a form (typeRef=50) with desired fields
3. Link the form to the type via attr 5001 (create form) and/or 5002 (detail forms)
4. Optionally create an Application Area entry or add to existing one

### Create a new Form

```json
{
  "branch": "<branch>",
  "objects": [{
    "meta": { "typeRef": 50 },
    "values": [
      { "attribute": 1000, "language": 201, "variant": 0, "value": "My Form EN" },
      { "attribute": 1000, "language": 202, "variant": 0, "value": "Mein Formular DE" },
      { "attribute": 5053, "language": 0, "variant": 0, "value": <attrDefId1>, "sortReverse": 0 },
      { "attribute": 5053, "language": 0, "variant": 0, "value": <attrDefId2>, "sortReverse": 1 },
      { "attribute": 5053, "language": 0, "variant": 0, "value": <attrDefId3>, "sortReverse": 2 }
    ]
  }]
}
```

---

## Application Area Discovery

To find object types by domain or theme:

1. Search Application Areas: `objectTypeIds: [60]`, `searchPhrase: "<theme>"`
2. Fetch the App Area with `novadb_cms_get_object` (include attr 6001)
3. Extract type IDs from attribute 6001 values
4. Fetch types with `novadb_cms_get_objects`
