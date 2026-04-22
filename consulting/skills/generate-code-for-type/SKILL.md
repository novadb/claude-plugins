---
name: generate-code-for-type
description: "[UNSUPPORTED in current MCP] Generate C# code for a single object type."
user-invocable: false
allowed-tools: novadb_cms_get_code_generator_type
---

# Generate Code for Type

> ⚠️ **UNSUPPORTED IN THE CURRENT C# MCP.**
> This skill relies on `novadb_cms_get_code_generator_type`, which is not exposed by the current NovaDB MCP (Noxum.Nova.AI.Mcp). Calling it will fail. Until code-generation tools are added back to the C# MCP, generate C# classes outside Claude (e.g. via the CLI/tooling that the old MCP wrapped). The content below is preserved for that future state; do not attempt to invoke.

Generate C# code for a single object type. ONLY for single-type code generation — NOT for multi-type generation or any other operations.

## Scope

**This skill ONLY handles:** Generating C# source code for a single object type.

**For generating code for multiple/all types** → use `generate-code-for-types`

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tools

1. `novadb_cms_get_code_generator_type` — Generate code for one type

## Parameters

```json
{
  "branch": "2100347",
  "language": "csharp",
  "type": "12345"
}
```

- `branch` — Numeric branch ID (int32, required). Always use the branch the user is currently working on.
- `language` — **Always use `"csharp"`**. This is the only supported language. (string, required)
- `type` — Type ID, GUID, or ApiIdentifier (string, required)

## Important

- The `language` parameter **must** be `"csharp"` — no other language is supported.
- The `type` parameter accepts an ID (numeric string), GUID, or ApiIdentifier.

## Response

Returns the generated C# source code as text.

## Common Patterns

### Language Parameter
Only `"csharp"` is supported. No other languages are available.

### Type Identifier
The `type` parameter accepts an ID (numeric), GUID, or ApiIdentifier string.

### API Response
Returns the generated C# source code as text.
