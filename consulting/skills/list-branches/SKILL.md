---
name: nova-list-branches
description: "List all NovaDB branches (work packages) with resolved references."
user-invocable: false
allowed-tools: work_package_query, object_get
---

# List Branches Reference

## Scope

**This skill is EXCLUSIVELY a reference for:** Listing and identifying available NovaDB branches (work packages).

**For searching/filtering branches by criteria** → see `find-branches` skill
**For fetching a single branch by ID** → see `get-branch` skill

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All numeric IDs in examples are samples — always use real IDs from the target system.

## Tool

`work_package_query` returns work packages (branches) with the default attribute set: Name, ParentBranch, BranchType, WorkflowState, DueDate, BranchAssignedTo. ObjRef resolution for parent can be done via a follow-up `object_get`.

## Fetch branches

```json
{
  "languages": [201, 202],
  "take": 100
}
```

- `languages` — Languages for display names (e.g. `[201, 202]`).
- `attributes` — Omit to get the default attribute set; pass an explicit int array to override.
- `searchPhrase` — Optional wildcard search.
- `filters` — Optional array of `"<attrId>:<lang>:<variant>:<op>:<value>"` filter strings.
- `skip` / `take` — Pagination (defaults: skip=0, take=20). For a full listing, set take to 100 and page if more exist.

### Response shape

```json
{
  "objects": [
    {
      "meta": { "id": 2100347, "typeRef": 40 },
      "values": [
        { "attribute": 1000, "language": 201, "value": "My Branch" },
        { "attribute": 4000, "language": 0, "value": 2100000 },
        { "attribute": 4001, "language": 0, "value": 123 },
        { "attribute": 4002, "language": 0, "value": 456 },
        { "attribute": 4003, "language": 0, "value": "2026-06-01" },
        { "attribute": 4004, "language": 0, "value": "jdoe" }
      ]
    }
  ]
}
```

### Present results

Show a table with columns: **ID, Name, Parent, Type, State, Due Date, Assigned To**.

## Resolving ObjRef display names

Attributes 4000, 4001, 4002 are ObjRefs. Collect the numeric values across all returned branches, deduplicate, and fetch them in one call:

```json
{
  "branchId": 2100347,
  "objectIds": [2100000, 123, 456],
  "languages": [201, 202],
  "attributes": [1000]
}
```

Tool: `object_get`. Match each returned object's `meta.id` to the ObjRef; the display name lives at `attribute: 1000`, `language: 201`.

## Pagination

`work_package_query` uses `skip` / `take`. When more results exist, advance `skip` by `take` for the next page. Response pagination metadata indicates whether more remains.
