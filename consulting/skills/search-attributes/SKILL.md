---
name: search-attributes
description: "Search attribute definitions by name, data type, or flags."
user-invocable: false
allowed-tools: object_query
---

# Search Attributes

## Scope

**This skill ONLY handles:** Searching and filtering attribute definitions (typeRef=10) via `object_query`.

**For fetching a single attribute by ID** → use `get-attribute`

Search attribute definitions by name, data type, or flags.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`object_query`

## Parameters

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "languages": [201, 202],
  "searchPhrase": "name",
  "skip": 0,
  "take": 5
}
```

- `branchId` — Numeric branch ID (int).
- `objectTypeId` — Always `10` (typeRef for attribute definitions).
- `languages` — Language IDs to load (e.g. `[201, 202]`).
- `searchPhrase` — (optional) Full-text search query. Supports wildcards.
- `searchAttributes` — (optional) `attr:lang`-formatted strings to scope the search.
- `filters` — (optional) Array of filter strings (see below).
- `attributes` — (optional) Attribute IDs to include in the result. Omit for defaults.
- `skip` / `take` — Pagination (defaults: skip=0, take=20).

## Attribute Filters

`filters` is an array of strings. The format is: `"<attrId>:<lang>:<variant>:<operator>:<value>"` — e.g. equal-to uses operator `0`.

### Filter by data type

```json
{ "filters": ["1001:0:0:0:ObjRef"] }
```

### Filter by inheritance behavior

```json
{ "filters": ["1013:0:0:0:Inheriting"] }
```

Valid values: `"None"`, `"Inheriting"`, `"InheritingAll"`.

### Filter by boolean flags

Boolean values are encoded as `"true"` / `"false"`:

| Flag | Attr ID |
|------|---------|
| Virtual | 1020 |
| Required | 1018 |
| Language dependent | 1017 |

```json
{ "filters": ["1020:0:0:0:true"] }
```

### Combined example

Search for required ObjRef attributes:

```json
{
  "branchId": 2100347,
  "objectTypeId": 10,
  "languages": [201, 202],
  "searchPhrase": "company",
  "filters": ["1001:0:0:0:ObjRef", "1018:0:0:0:true"],
  "skip": 0,
  "take": 10
}
```

## Response

Returns `ObjectQueryResult` with matching objects plus pagination info.

## Data Type Enum (for filter values)

```
String, TextRef, TextRef.JavaScript, TextRef.CSS,
XmlRef.SimpleHtml, XmlRef.VisualDocument,
Integer, Decimal, Float, Boolean,
DateTime, DateTime.Date,
ObjRef, BinRef, BinRef.Icon, BinRef.Thumbnail,
String.DataType, String.InheritanceBehavior, String.UserName, String.RGBColor
```

## Common Patterns

### API Response
Returns `ObjectQueryResult` — inspect `objects` for results and the pagination fields for more pages.
