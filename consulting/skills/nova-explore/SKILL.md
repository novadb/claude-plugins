---
name: nova-explore
description: "Schema browsing API reference — tools, parameters, attribute ID tables, pagination, and type discovery."
user-invocable: false
allowed-tools: object_get, object_query, object_count, objecttype_describe, objecttype_query, work_package_query, language_query
---

# NovaDB Schema Exploration Reference

## Scope

**This skill is EXCLUSIVELY a reference for:** Read-only NovaDB schema browsing — object types, attribute definitions, forms, application areas, and configuration discovery.

**For searching data objects** → see `nova-search` skill
**For form configuration** → see `nova-forms` skill
**For branch listing** → see `nova-list-branches` skill

Full API reference for read-only schema browsing. All tools, parameters, attribute tables, and discovery strategies.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All numeric IDs in examples are samples — always use real IDs from the target system.

## Available Tools

| Tool | Purpose |
|------|---------|
| `object_get` | Fetch one or more objects by numeric id(s) |
| `object_query` | Typed listing and full-text search (with optional filters / sorting / pagination) |
| `object_count` | Count matching objects without fetching |
| `object_xmllinkcount` | Count objects whose XML attributes reference given ids |
| `objecttype_describe` | Full structured schema of one object type (attributes, forms, flags) |
| `objecttype_query` | List object types (defaults to customer types; set `includeSystemTypes: true` for system types) |
| `work_package_query` | List and filter branches (work packages) |
| `language_query` | Enumerate available languages with their numeric IDs |

> The old Index API extras — `suggestions`, `match_strings`, `object_occurrences`, `work_item_occurrences` — are **not** available in the current C# MCP. If faceted counts are needed, aggregate from pages of `object_query`/`object_count` instead.

---

## Tool Parameters

### `object_get`

```json
{
  "branchId": 2100347,
  "objectIds": [2100500, 2100501, 2100502],
  "languages": [201, 202],
  "attributes": []
}
```

- `branchId` — int.
- `objectIds` — int[] (one or many).
- `languages` — int[] (e.g. `[201, 202]`).
- `attributes` — int[] (empty → all).

### `object_query`

```json
{
  "branchId": 2100347,
  "objectTypeId": 0,
  "languages": [201, 202],
  "searchPhrase": "Company",
  "filters": ["1001:0:0:0:ObjRef"],
  "attributes": [],
  "skip": 0,
  "take": 20
}
```

- `branchId` — int.
- `objectTypeId` — int (the typeRef to list). Use `objectTypeId: 10` for attribute definitions, `0` for object types, etc.
- `languages` — int[].
- `searchPhrase` — optional wildcard search (e.g. `"Com*"`).
- `searchAttributes` — optional `attr:lang` strings to scope the search.
- `filters` — optional array of `"<attrId>:<lang>:<variant>:<operator>:<value>"` strings (see §Filters).
- `attributes` — optional int[] to restrict returned attributes.
- `skip` / `take` — pagination (defaults: 0 / 20).

### `object_count`

Same params as `object_query` (minus `attributes`, `skip`, `take`). Returns `{ count }`.

### `objecttype_describe`

```json
{
  "branchId": 2100347,
  "objectTypeId": 2100000,
  "languages": [201, 202]
}
```

Returns `{ id, apiIdentifier, name, description, displayNameAttrId, createFormId, detailFormIds, attributes[] }`. Each attribute carries `{ dataType, multiValue, languageDependent, variantDependent, defaultForm, apiIdentifier }`.

### `objecttype_query`

```json
{
  "branchId": 2100347,
  "languages": [201, 202],
  "includeSystemTypes": false,
  "searchPhrase": "Company",
  "skip": 0,
  "take": 20
}
```

Defaults to customer types only. Set `includeSystemTypes: true` to also get built-in types (Attribute Definition, Form, etc.).

### `work_package_query`

```json
{
  "languages": [201, 202],
  "take": 100
}
```

No `branchId` parameter — lists all work packages. Filter by id/parent/assignee via `filters`.

### `language_query`

No parameters. Returns `{ languages: [ { id, name, isoCode } ] }`. Useful when you don't yet know the numeric language IDs for the user's locales.

---

## Filters (string syntax)

`filters` is an array of strings with shape `"<attrId>:<langId>:<variantId>:<operator>:<value>"`.

| Operator | Meaning |
|----------|---------|
| 0 | Equal |
| 1 | NotEqual |
| 2 | GreaterThan |
| 3 | LessThan |
| 4 | Like |
| 7 | ObjRefLookup |

Examples:

```
"1001:0:0:0:ObjRef"       // data type equals ObjRef
"1018:0:0:0:true"         // required = true (booleans encoded as "true"/"false")
"1000:201:0:4:Comp*"      // name EN LIKE "Comp*"
"4000:0:0:7:2100347"      // parent branch (ObjRef lookup) = 2100347
```

---

## CmsValue Structure

Every object value follows:

```typescript
{
  attribute: number,     // Attribute definition ID
  language: number,      // 201=EN, 202=DE, 0=language-independent
  variant: number,       // 0=default
  value: unknown,        // The actual value
  sortReverse?: number   // Multi-value ordering (optional — array-in-value is auto-expanded)
}
```

When writing (`object_create`/`object_update`), `values` is passed as a JSON-encoded **string** containing an array of CmsValues. For multi-value attributes either use explicit `sortReverse` entries OR put an array inside `value` — the MCP expands it.

---

## typeRef Constants

| typeRef | Description |
|---------|-------------|
| 0 | Object types |
| 10 | Attribute definitions |
| 40 | Branches / work packages |
| 50 | Forms |
| 60 | Application areas |

---

## Universal Attributes

Apply to all object types.

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Name | 1000 | String | Language-dependent (201=EN, 202=DE) |
| Description | 1012 | String | Language-dependent |
| API Identifier | 1021 | String | Language-independent |

---

## Object Type Attributes (typeRef=0)

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Create form | 5001 | ObjRef | Single form reference |
| Detail forms | 5002 | ObjRef | Multi-value |
| Display name attribute | 5005 | ObjRef | Which attribute serves as display name |
| Binary proxy | 5009 | Boolean | |

> Prefer `objecttype_describe` over reading these attributes directly.

---

## Attribute Definition Attributes (typeRef=10)

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Data type | 1001 | String.DataType | See DATA_TYPE_ENUM |
| Variant axis | 1002 | ObjRef | |
| Unit of measure | 1003 | ObjRef | NOT attribute definitions |
| Allow multiple | 1004 | Boolean | |
| Max values | 1014 | Integer | 1=single, 0=unlimited |
| Allowed types | 1015 | ObjRef | Multi-value, for ObjRef data type |
| Language dependent | 1017 | Boolean | |
| Required | 1018 | Boolean | |
| Virtual | 1020 | Boolean | Computed attribute |
| Reverse relation name | 1051 | String | Language-dependent |

**DATA_TYPE_ENUM values:**
```
String, TextRef, TextRef.JavaScript, TextRef.CSS,
XmlRef.SimpleHtml, XmlRef.VisualDocument,
Integer, Decimal, Float, Boolean,
DateTime, DateTime.Date,
ObjRef, BinRef, BinRef.Icon, BinRef.Thumbnail,
String.DataType, String.InheritanceBehavior, String.UserName, String.RGBColor
```

---

## Form Attributes (typeRef=50)

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Form content (fields) | 5053 | ObjRef | Multi-value |
| Condition attribute | 5054 | ObjRef | Single |
| Condition refs | 5055 | ObjRef | Multi-value |
| Is single editor | 5056 | Boolean | |

---

## Branch Attributes (typeRef=40)

| Attribute | ID | Type |
|-----------|------|------|
| Parent branch | 4000 | ObjRef |
| Branch type | 4001 | ObjRef |
| Workflow state | 4002 | ObjRef |
| Due date | 4003 | DateTime.Date |
| Assigned to | 4004 | String.UserName |

---

## Application Area Attributes (typeRef=60)

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Object types | 6001 | ObjRef | Multi-value — the types in this area |
| Sort key | 6003 | Integer | Display ordering |

---

## Type → Form → Attribute Chain

Use `objecttype_describe` as the one-shot way to resolve this chain. The raw chain is still:

```
Object Type (typeRef=0)
  → Create Form (attr 5001, single ObjRef)
  → Detail Forms (attr 5002, multi-value ObjRef)
    → Form Content (attr 5053, multi-value ObjRef)
      → Attribute Definitions (typeRef=10)
```

---

## Application Area Discovery Strategy

To find object types by domain or theme:

1. `object_query` with `objectTypeId: 60` and `searchPhrase: "<theme>"`
2. `object_get` on the matching App Area with `attributes: [6001]`
3. Extract type IDs from 6001 values
4. `objecttype_describe` for each, or `object_get` for raw `CmsObject`s

Do NOT search object types (typeRef=0) by theme name — types have generic names. Application Areas provide the thematic grouping.

---

## ObjRef Resolution Pattern

When values contain numeric ObjRef references:

1. Collect all unique ObjRef IDs across the objects
2. Batch-fetch with `object_get`:
   ```json
   {
     "branchId": 2100347,
     "objectIds": [2100500, 2100501, 2100502],
     "languages": [201, 202],
     "attributes": [1000]
   }
   ```
3. Match each resolved object's `meta.id` to the original ObjRef values
4. Display the name (attribute 1000, language 201) instead of the bare numeric ID

---

## Pagination

All listing tools (`object_query`, `objecttype_query`, `work_package_query`) use `skip` / `take`. When the response indicates more results remain, advance `skip` by `take` for the next page.

`comment_query` / `job_query` use an opaque `continueToken` — pass it back in the next call.
