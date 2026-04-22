---
name: nova-find-branches
description: "Search and filter branches (work packages)."
user-invocable: false
allowed-tools: work_package_query
---

# Find Branches Reference

Search for branches by name, assigned user, parent, or other criteria.

## Scope

**This skill ONLY handles:** Searching and filtering branches by criteria (name, assignee, parent, state).

**For a quick overview of all branches** → use `list-branches`
**For fetching a single branch by ID** → use `get-branch`

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All numeric IDs in examples are samples — always use real IDs from the target system.
>
> The new C# MCP does not expose the Index API's count / occurrence / facet tools. Only result-list queries are available — if you need counts/facets, fetch pages and aggregate client-side.

## Tool

`work_package_query`

## Search

```json
{
  "languages": [201, 202],
  "searchPhrase": "feature",
  "skip": 0,
  "take": 20
}
```

- `languages` — Languages for display names.
- `searchPhrase` — (optional) Full-text / wildcard search.
- `skip` / `take` — Pagination (defaults: skip=0, take=20).

### Response

```json
{
  "objects": [
    { "meta": { "id": 2100347, "typeRef": 40 }, "values": [ { "attribute": 1000, "language": 201, "value": "My Branch" } ] }
  ]
}
```

Advance `skip` by `take` for the next page when more results remain.

---

## Attribute Filters

`filters` is an array of strings with the shape `"<attrId>:<langId>:<variantId>:<operator>:<value>"`.

### Filter by assigned user

```json
{
  "filters": ["4004:0:0:0:jdoe"]
}
```

Operator `0` = Equal.

### Filter by parent branch

```json
{
  "filters": ["4000:0:0:7:2100347"]
}
```

Operator `7` = ObjRef lookup (follows reference hierarchy).

### Combined example

```json
{
  "languages": [201, 202],
  "searchPhrase": "feature",
  "filters": [
    "4004:0:0:0:jdoe",
    "4000:0:0:7:2100347"
  ],
  "skip": 0,
  "take": 20
}
```

### compareOperator Values

| Value | Meaning |
|-------|---------|
| 0 | Equal |
| 1 | NotEqual |
| 2 | GreaterThan |
| 3 | LessThan |
| 4 | Like |
| 7 | ObjRefLookup |

## Common Patterns

### API Response
Returns `ObjectQueryResult` with `objects[]` and pagination info.

### No Facets / Occurrences
The old Index API facet tools (`object_occurrences`, `suggestions`, `match_strings`) are not available in the new C# MCP. If you need counts or dashboards, page through the result set.
