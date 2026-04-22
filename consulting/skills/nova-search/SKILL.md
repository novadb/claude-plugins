---
name: nova-search
description: "Object search workflow and reference — parameters, filters, pagination, and ObjRef resolution."
user-invocable: false
allowed-tools: object_query, object_count, object_get, objecttype_describe
---

# NovaDB Search Reference

## Scope

**This skill is EXCLUSIVELY a reference for:** Searching, filtering, and counting NovaDB data objects via `object_query` with detail resolution via `object_get`.

**For schema browsing** → see `nova-explore` skill
**For form configuration** → see `nova-forms` skill
**For branch listing** → see `nova-list-branches` skill

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All numeric IDs in examples are samples — always use real IDs from the target system.

## Search Workflow

### 1. Count first (optional but recommended)

Get the total count before fetching results:

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "searchPhrase": "company"
}
```

Tool: `object_count`. Response: `{ count }`.

### 2. Search

Fetch matching objects:

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "languages": [201, 202],
  "searchPhrase": "company",
  "skip": 0,
  "take": 20
}
```

Tool: `object_query`.

### 3. Resolve ObjRef values

For results that contain ObjRef values worth displaying, batch-fetch the referenced objects:

```json
{
  "branchId": 2100347,
  "objectIds": [2100500, 2100501, 2100502],
  "languages": [201, 202],
  "attributes": [1000]
}
```

Tool: `object_get`. Match `meta.id` to the ObjRef values.

### 4. Present as table

Show results as a readable markdown table. Include ID, display name, and relevant resolved attributes.

---

## `object_query` — Full Reference

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "languages": [201, 202],
  "searchPhrase": "name",
  "searchAttributes": ["1000:201"],
  "filters": ["1001:0:0:0:ObjRef"],
  "attributes": [],
  "skip": 0,
  "take": 20
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `branchId` | int | yes | Branch identifier |
| `objectTypeId` | int | yes | typeRef to list (e.g. `10` for attributes) |
| `languages` | int[] | yes | Languages for display names |
| `searchPhrase` | string | no | Full-text / wildcard search query |
| `searchAttributes` | string[] | no | Restrict phrase search to `attr:lang` pairs |
| `filters` | string[] | no | Attribute-level filters (see below) |
| `attributes` | int[] | no | Which attributes to load (empty → defaults) |
| `skip` | int | no | Offset for pagination (default 0) |
| `take` | int | no | Max results per page (default 20) |

### Filter String Syntax

Each filter is `"<attrId>:<langId>:<variantId>:<operator>:<value>"`.

| Field | Description |
|-------|-------------|
| `attrId` | Attribute definition id |
| `langId` | Language id (0 for language-independent, 201 for EN, 202 for DE) |
| `variantId` | Variant id (usually 0) |
| `operator` | Comparison operator (see table) |
| `value` | String value to compare (booleans as `"true"` / `"false"`) |

### compareOperator Values

| Value | Meaning | Example use |
|-------|---------|-------------|
| 0 | Equal | Exact match on data type, boolean flags |
| 1 | NotEqual | Exclude specific values |
| 2 | GreaterThan | Date/number comparisons |
| 3 | LessThan | Date/number comparisons |
| 4 | Like | Partial string matching (with wildcards) |
| 7 | ObjRefLookup | Find objects referencing a specific id |

### Response

```json
{
  "objects": [
    {
      "meta": { "id": 2100500, "typeRef": 10 },
      "values": [
        { "attribute": 1000, "language": 201, "value": "Company Name" }
      ]
    }
  ]
}
```

Pagination info is returned alongside (consult the result — page again with `skip += take` when more results remain).

---

## `object_count` — Reference

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "searchPhrase": "name"
}
```

Same filter options as `object_query` (minus `attributes`, `skip`, `take`). Returns `{ count }`.

---

## `object_xmllinkcount` — Reference

```json
{
  "branchId": 2100347,
  "objectIds": [2100500, 2100501]
}
```

Returns `{ count }` of objects whose XML attributes reference the given ids. Useful pre-deletion check.

---

## Faceted / Occurrence Queries

Not available in the current C# MCP. If you need counts per user / per type / per date bucket, run `object_count` once per filter value and aggregate client-side, or page through `object_query` results and aggregate.

---

## ObjRef Resolution

When search results contain ObjRef attributes you need to display:

1. Fetch the full object via `object_get` with appropriate `attributes`.
2. Collect ObjRef IDs from the values.
3. Batch-resolve via `object_get` with `attributes: [1000]`.
4. Match `meta.id` to the ObjRef values and display names.

```json
{
  "branchId": 2100347,
  "objectIds": [2100500, 2100501],
  "languages": [201, 202],
  "attributes": [1000]
}
```

### Binary References (BinRef)

If an object contains BinRef attributes (images, documents), read Attr **11000** (identifier) and **11005** (extension) from the referenced binary object. To download → use `get-file` skill (currently unsupported in the C# MCP — listed for completeness).

---

## Application Area Discovery

To find object types by domain/theme, do NOT search object types directly — they have generic names. Instead:

1. `object_query` with `objectTypeId: 60`, `searchPhrase: "<theme>"`
2. `object_get` on the App Area with `attributes: [6001]`
3. Extract type IDs from 6001 values
4. `objecttype_describe` for each

---

## Branch Parameter

All object tools take `branchId` as an **int**. No named branch identifiers. Query results are scoped to the specified branch — objects from other branches will not appear.
