---
name: get-branch
description: "Fetch a single branch (work package) by ID."
user-invocable: false
allowed-tools: work_package_query
---

# Get Branch

Fetch a single branch (work package) with its attributes.

## Scope

**This skill ONLY handles:** Fetching a single branch by its ID.

**For listing all branches** → use `list-branches`
**For searching/filtering branches** → use `find-branches`

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.
>
> The new C# MCP has no dedicated single-branch getter. Use `work_package_query` with a filter on the branch id.

## Steps

### 1. Query by branch id

```json
{
  "languages": [201, 202],
  "filters": ["1:0:0:0:2100347"],
  "take": 1
}
```

Tool: `work_package_query`.

- `filters` — an array of `"<attrId>:<lang>:<variant>:<op>:<value>"` strings. Attribute `1` is the built-in object id; use it to pin to a single branch.
- `languages` — load display names in the requested languages.
- Omit `attributes` to use the tool defaults (Name, ParentBranch, BranchType, WorkflowState, DueDate, BranchAssignedTo).

### 2. Extract values

Parse `values` using these attribute IDs:

| Attribute | ID | Language | Description |
|-----------|------|----------|-------------|
| Name (EN) | 1000 | 201 | English display name |
| Name (DE) | 1000 | 202 | German display name |
| Parent | 4000 | 0 | Parent branch ID (ObjRef) |
| Type | 4001 | 0 | Branch type ID (ObjRef) |
| Workflow state | 4002 | 0 | State ID (ObjRef) |
| Due date | 4003 | 0 | ISO date string |
| Assigned to | 4004 | 0 | Username |

### 3. Resolve ObjRef names (optional)

Attributes 4000, 4001, 4002 are ObjRefs. `work_package_query` already resolves parent-branch display names when present in the result set; otherwise fetch the referenced objects via `object_get`.

### 4. Present enriched result

```
Branch: My Feature Branch (ID: 2100500)
Parent: Main Branch (ID: 2100347)
Type: Feature (ID: 123)
State: In Progress (ID: 456)
Due: 2026-06-01
Assigned to: jdoe
```

## Common Patterns

### ObjRef Resolution
Resolve numeric ObjRef values via `object_get` (attribute 1000 for display names).

### API Response (Work Package Query)
Returns `ObjectQueryResult` with `objects[]` and pagination info.
