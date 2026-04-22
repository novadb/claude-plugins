---
name: create-branch
description: "Create a new branch (work package) in NovaDB."
user-invocable: false
allowed-tools: work_package_create
---

# Create Branch

Create a new branch (work package) in NovaDB.

## Scope

**This skill ONLY handles:** Creating a new branch (work package) in NovaDB.

**For updating existing branches** → use `update-branch`

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`work_package_create`

## Parameters

The new MCP takes typed named parameters — no raw `values` array.

```json
{
  "name": "My Branch",
  "parentBranchId": 2100347,
  "branchTypeId": 123,
  "assignedTo": "jdoe",
  "comment": "Created via AI assistant"
}
```

- `name` — Branch name (string, required).
- `parentBranchId` — Parent branch ID (int, required).
- `branchTypeId` — (optional) Branch type (ObjRef int).
- `assignedTo` — (optional) Username.
- `comment` — (optional) Audit trail.

### What about the other attributes (description, workflow state, due date)?

The typed create tool only accepts the five parameters above. To set workflow state (attr 4002), due date (attr 4003), or additional language names (attr 1000 @ lang 202), call `update-branch` immediately after creation — the create tool returns the new branch id.

## Response

Returns the created branch as a `CmsObject` with `meta` (id, guid, typeRef) and `values`.

## Common Patterns

### Follow-up updates
If additional metadata is needed beyond the five typed parameters, chain `create-branch` → `update-branch` in one flow.

### API Response (Create Branch)
Returns the created branch as a `CmsObject` with `meta` and `values`.
