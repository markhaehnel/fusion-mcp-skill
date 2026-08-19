# Fusion Electronics queries

Read this reference for schematics, PCB boards, libraries, packages, connectivity, variants, or ERC/DRC analysis.

## Confirm context and schema

`electronics_read` requires an active Electronics document. Confirm the active document before querying, and do not assume a mechanical Design document contains Electronics entities.

Before querying an entity type for the first time in a task:

1. Read `resource://mcp.electronics_entity_types` to choose the entity that represents the question.
2. Read its exact registered per-class schema resource for the available fields, filterable properties, and supported operators. Schema resource suffixes are lowercase and may use snake case: `electronics.Net` maps to `resource://mcp.electronics_schema_net`, `electronics.PinRef` to `resource://mcp.electronics_schema_pin_ref`, `electronics.DeviceSet` to `resource://mcp.electronics_schema_device_set`, and `electronics.Package3D` to `resource://mcp.electronics_schema_package3d`. List the server resources when the mapping is uncertain rather than guessing a URI.
3. Request only the fields needed. A query with no `object` returns every available column and can be unnecessarily large.

Use `eq` for strings. Use `lt` or `gt` only when that numeric property explicitly exposes the operator in its schema.

## Query deliberately

- Start with a narrow page and filters based on verified properties. The default limit is 100 and the maximum is 1000.
- When `pagination.hasMore` is true, continue using the returned `pagination.nextOffset`; do not invent offsets or silently analyze only the first page.
- Keep entity types distinct: schematic `Net`/`Part`/`Instance`/`PinRef` relationships are not interchangeable with board `Signal`/`Element`/`ContactRef`/`Via` data.
- Follow explicit reference fields to establish connectivity. Do not infer that matching display names alone prove an electrical connection.
- For design checks, query `electronics.Error` and preserve its severity, message, and object references in the result. Distinguish an empty query result from a failure to access the expected document.
- For large designs, request only needed fields, cover every required page, and then aggregate client-side rather than returning raw rows the user did not request.

The Electronics MCP operation is read-only. If the user requests a schematic or board mutation, explain that the exposed Electronics operation cannot perform it. Do not route the change through a mechanical Fusion script unless the live Fusion API documentation explicitly supports the exact Electronics mutation and the user authorizes that approach.

Report the active document, entity types queried, filters and pagination coverage, and any limitations that affect the conclusion.
