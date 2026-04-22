---
name: nova-schema
description: "Meta-object CRUD reference — type catalog, schema discovery, create/update/delete workflows, impact analysis, and safety patterns for all objects with typeRef < 8192."
user-invocable: false
allowed-tools: object_get, object_query, object_create, object_update, object_delete, object_count, object_xmllinkcount, objecttype_describe, objecttype_query
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
| Validation Code | 1008 | TextRef.JavaScript | Client-side validation |
| Virtualization Code | 1009 | TextRef.JavaScript | Computed/virtual value |
| API Identifier | 1021 | String | Programmatic identifier |

### Form (typeRef=50)

| Attribute | ID | Type | Purpose |
|-----------|------|------|---------|
| Name | 1000 | String | Display name (language-dependent) |
| Form Fields | 5053 | ObjRef (multi) | Ordered list of attribute definitions shown on this form |
| Condition attribute | 5054 | ObjRef | Controls conditional visibility |
| Condition refs | 5055 | ObjRef (multi) | Values that trigger visibility |
| Is single editor | 5056 | Boolean | Single-editor mode |

### Application Area (typeRef=60)

| Attribute | ID | Type | Purpose |
|-----------|------|------|---------|
| Name | 1000 | String | Display name (language-dependent) |
| Object Types | 6001 | ObjRef (multi) | Types belonging to this area |
| Icon | 6002 | String | Area icon |

---

## Schema Discovery Workflow

Before modifying any meta-object, discover its structure.

### For an Object Type → prefer `objecttype_describe`

```json
{
  "branchId": 2100347,
  "objectTypeId": <typeId>,
  "languages": [201, 202]
}
```

Returns the full structured schema in one call.

### For other meta-types → use `object_query` + `object_get`

List instances:

```json
{
  "branchId": 2100347,
  "objectTypeId": <metaTypeRef>,
  "languages": [201, 202],
  "take": 20
}
```

Tool: `object_query`.

Inspect one instance via `object_get` with the id.

### Finding which types/forms use an attribute

```json
{
  "branchId": 2100347,
  "objectTypeId": 50,
  "languages": [201, 202],
  "filters": ["5053:0:0:0:<attrDefId>"],
  "take": 50
}
```

Tool: `object_query`. Returns all forms that list the attribute. Then for each form, query which types reference it on 5001/5002.

---

## Impact Analysis Workflows

### Before modifying an Attribute Definition

1. Find all forms using the attribute (see above).
2. For each form, find all types that reference it via 5001 or 5002 (`object_query` with `"5001:0:0:0:<formId>"` / `"5002:0:0:0:<formId>"`).
3. Count data objects of affected types with `object_count` — one call per type id.
4. Report to user: affected forms, types, and data object counts before proceeding.

### Before deleting a Schema Object

1. Check XML references via `object_xmllinkcount`:
   ```json
   { "branchId": 2100347, "objectIds": [<objectId>] }
   ```
2. Type-specific checks:
   - **Object Type (0):** Count data objects of this type. Check forms referencing this type.
   - **Attribute Definition (10):** Find forms listing it in 5053.
   - **Form (50):** Find types referencing it via 5001/5002.
   - **Application Area (60):** List all types in 6001.
3. Report all dependencies and get explicit confirmation.

---

## Create Workflow

### 1. Verify the meta-type

Confirm the target `typeRef` is valid and < 8192.

### 2. Build the values payload (JSON-encoded string)

```json
{
  "branchId": 2100347,
  "objectTypeId": <metaTypeRef>,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"Name EN\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Name DE\"},{\"attribute\":<attrId>,\"language\":0,\"variant\":0,\"value\":\"<value>\"}]"
}
```

Tool: `object_create`. Response: `{ createdObjectId, transaction }`.

### 3. Confirm with user and execute

Show the user what will be created, wait for approval, then call.

### 4. Verify

`object_get` (or `objecttype_describe` for a new type) with the new id.

---

## Update Workflow

### 1. Read the current state

- For object types: `objecttype_describe`.
- For other meta-objects: `object_get`.

### 2. Verify typeRef < 8192 and ID >= 500

Refuse if either check fails.

### 3. Run impact analysis

See Impact Analysis Workflows.

### 4. Merge changes

- **Single-value attributes:** Can be sent in isolation (read-then-merge).
- **Multi-value attributes (1004=true):** Must send the COMPLETE value set — prefer array-in-value.

### 5. Show changes and get confirmation

Display: current values vs. new values, affected dependents.

### 6. Execute

```json
{
  "branchId": 2100347,
  "objectId": <objectId>,
  "values": "[{\"attribute\":<attrId>,\"language\":0,\"variant\":0,\"value\":\"<newValue>\"}]"
}
```

Tool: `object_update`. Re-read the object and show the result.

---

## Delete Workflow

### 1. Read the target object

Fetch and verify `typeRef < 8192`, `ID >= 500`.

### 2. Impact analysis + XML refs

See above. Use `object_xmllinkcount`.

### 3. Show full impact report and get confirmation

Only proceed with explicit approval.

### 4. Delete (one at a time)

```json
{
  "branchId": 2100347,
  "objectIds": [<objectId>],
  "comment": "reason"
}
```

Tool: `object_delete`. Maximum 1 schema object per call.

### 5. Verify

Query for the object with `isDeleted` filter or confirm `deletedObjects: 1` in the response.

---

## Value Format Table by DataType

| DataType | JSON value type | Example | Notes |
|----------|----------------|---------|-------|
| `String` | string | `"Hello World"` | |
| `TextRef` | string | `"Long text content..."` | |
| `TextRef.JavaScript` | string | `"function() { ... }"` | |
| `TextRef.CSS` | string | `".class { color: red; }"` | |
| `XmlRef.SimpleHtml` | string | `"<div>HTML content</div>"` | XHTML with `<div>` root |
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
| `objecttype_describe` | `ObjectTypeDescription` |
| `objecttype_query` | `ObjectQueryResult` |

---

## Branch Parameter

- All tools take `branchId` as an **int**. No named branch identifiers.

> **Recommendation:** Schema changes should be made on a named branch (work package). This allows review and rollback.

---

## Common Operations

### Create a new Attribute Definition

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"My Attribute EN\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Mein Attribut DE\"},{\"attribute\":1001,\"language\":0,\"variant\":0,\"value\":\"String\"},{\"attribute\":1017,\"language\":0,\"variant\":0,\"value\":true},{\"attribute\":1018,\"language\":0,\"variant\":0,\"value\":false}]"
}
```

### Add an Attribute to a Form

1. Read the form (`object_get`) — extract current fields from 5053.
2. Append the new attribute id to the array.
3. Update via `object_update` with the complete array inside `value`.

### Create a new Object Type with Form

1. Create the object type (`object_create`, `objectTypeId: 0`) with name.
2. Create a form (`object_create`, `objectTypeId: 50`) with desired fields.
3. Link the form to the type via `object_update` setting 5001 (create form) and/or 5002 (detail forms).
4. Optionally add to an Application Area.

> Each step runs in a separate transaction. Partial failures leave partial state.

### Create a new Form

```json
{
  "branchId": 2100347,
  "objectTypeId": 50,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"My Form EN\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Mein Formular DE\"},{\"attribute\":5053,\"language\":0,\"variant\":0,\"value\":[<attrDefId1>,<attrDefId2>,<attrDefId3>]}]"
}
```

---

## Application Area Discovery

1. `object_query` with `objectTypeId: 60`, `searchPhrase: "<theme>"`
2. `object_get` on the App Area with `attributes: [6001]`
3. Extract type IDs from 6001
4. `objecttype_describe` for each
