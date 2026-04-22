---
name: update-object-type
description: "Update name or description of an existing object type."
user-invocable: false
allowed-tools: object_update, objecttype_describe
---

# Update Object Type

Update name or description of an existing object type. ONLY for modifying type properties — NOT for creating types, adding attributes, or editing forms.

## Scope

**This skill ONLY handles:** Updating the name or description of an existing object type (typeRef=0).

**For adding attributes to a type** → use `add-attribute-to-type`
**For reading a type's full definition** → use `get-object-type`

## Tools

- `objecttype_describe` — (optional) Read the current type metadata to confirm the target before updating.
- `object_update` — Apply the update.

## Parameters

Provide only the fields you want to change.

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Name (EN) | 1000 | String | `language: 201`, `variant: 0` |
| Name (DE) | 1000 | String | `language: 202`, `variant: 0` |
| Description (EN) | 1012 | String | `language: 201`, `variant: 0` |
| Description (DE) | 1012 | String | `language: 202`, `variant: 0` |

## Example Call

```json
{
  "branchId": 2100347,
  "objectId": <typeId>,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"New English Name\"},{\"attribute\":1000,\"language\":202,\"variant\":0,\"value\":\"Neuer deutscher Name\"},{\"attribute\":1012,\"language\":201,\"variant\":0,\"value\":\"New description\"},{\"attribute\":1012,\"language\":202,\"variant\":0,\"value\":\"Neue Beschreibung\"}]",
  "comment": "Renamed type"
}
```

- `values` is a JSON-encoded **string** of a CmsValue array.
- Only include fields you want to change.
- At least one field must be provided.

## Response

Returns `UpdateObjectResult` with `objectId`, `updatedObjects`, `createdValues`, `transaction`.

## Common Patterns

### CmsValue Format
Every value entry in the JSON string follows: `{ attribute, language, variant, value }`
- `language`: 201=EN, 202=DE, 0=language-independent
- `variant`: 0=default
- Only include fields being changed; omitted fields remain unchanged.

### API Response (PATCH/Update)
Returns `{ transaction }`. Call `objecttype_describe` afterward to confirm changes.
