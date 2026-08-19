---
name: fusion-mcp
description: Work safely in Autodesk Fusion through the Fusion MCP server for mechanical CAD, document lifecycle, viewport inspection, and Electronics data queries. Use when reading, creating, or modifying a Fusion design through MCP; do not use for standalone CAD files outside an active Fusion session.
license: MIT
compatibility: Requires an Agent Skills-compatible client with access to the Fusion MCP server tools and an active Autodesk Fusion session; Electronics schema discovery also requires MCP resource access.
---

# Fusion MCP

Use the Fusion MCP server capabilities to inspect the active Fusion session, make deliberate model changes, and verify the result. Preserve the user's design intent and document state; do not treat a successful API call as proof that the model is correct.

AI clients may namespace or display MCP tools differently. Resolve tools by their exposed schemas and capabilities rather than assuming a vendor-specific prefix:

- The Fusion read capability, commonly named `read`, covers projects, documents, API documentation, screenshots, and the active command.
- The Fusion execution capability, commonly named `execute`, runs scripts and document open, close, or save operations.
- The Fusion update capability, commonly named `update`, performs undo and redo.
- The Electronics read capability, commonly named `electronics_read`, queries Electronics entities.
- The client's MCP resource operations provide Electronics entity-type and per-class schema resources.

If a required capability is unavailable, state the limitation. Do not substitute an unrelated tool or infer a mutation capability that the server does not expose.

## Route the task

- For mechanical modeling, geometry, parameters, or assemblies, read [references/mechanical-modeling.md](references/mechanical-modeling.md).
- For schematics, PCB boards, libraries, or connectivity analysis, read [references/electronics.md](references/electronics.md).
- For document search/open/close, screenshots, undo/redo, or API discovery alone, use the core workflow below.

## Establish the Fusion context

1. Read the open-document list before a change. Confirm the active document name and whether it is already modified. If the user names another document, search for it and do not guess among ambiguous matches.
2. Read `activeCommand` before a mutation or document operation. A session default with `isDefaultCommand: true` is not an interactive blocker. If a non-default command or preview is active, do not run a mutating script, undo, redo, save, close, or open another document; ask the user to finish or cancel it first.
3. Capture a screenshot only when visual state materially helps inspection or verification. Prefer the current camera when it exposes the relevant geometry. A named direction changes the viewport, so use one only when the task needs that view; do not rotate the view merely to satisfy a checklist.
4. In scripts, validate `Application.get().activeProduct` and the required product/document type before the first mutation. Fail clearly when the active product is not suitable.

Opening a document changes UI state, so open only the intended result. Saving is a separate action: never save unless the user expressly asks. When closing a modified document, obtain the user's save-or-discard choice and pass the matching confirmation flag.

## Discover APIs before using them

Before scripting, use `read(queryType="apiDocumentation")` for classes and members that control a mutation, are unfamiliar, or have uncertain signatures or semantics. Reuse documentation already verified in the same task and search narrowly:

- Query the class name to learn its collections, inputs, and relevant members.
- Query important member names for full signatures, return values, version notes, and object-context requirements.
- Limit with `filter` such as `adsk.fusion.ExtrudeFeatures` or `adsk.cam` when possible.

Prefer this live documentation over remembered API details. A read-only inspection may use `execute(featureType="script")`, but it must still follow the script contract.

## Write safe Fusion scripts

Every executed script must:

- Define exactly `def run(_context: str):` as its entry point.
- Import only the required `adsk` namespaces and obtain the application object inside `run`; obtain document and product objects only when the operation needs them.
- Validate the prerequisites relevant to the operation before creating or editing anything.
- Use a small, coherent scope and print concise facts needed for verification: affected names, counts, parameter expressions, dimensions, or health states.
- Let exceptions propagate. Do not wrap `run` in `try/except`; the MCP error is needed to diagnose the real failure.

Make retries safe. Before rerunning a mutating script, inspect whether it partially succeeded and compare the actual state with the intended objects, parameters, and invariants. Reuse an intended name or other existing identifier only when it is unambiguous; do not add artificial marker geometry or metadata solely to make retries detectable. If the partial state is uncertain, stop and inspect rather than duplicating work. Never delete or overwrite same-named user geometry merely to force idempotence.

## Change and verify

1. Make the smallest coherent change that satisfies the request. Preserve existing components, parameters, feature order, naming conventions, appearances, materials, and manufacturing intent unless the user asks to change them.
2. Inspect the execution result for exceptions and printed invariants.
3. Verify through a second read-only inspection. When the result has visually relevant geometry or placement, also capture a screenshot from the current view or a deliberately chosen direction. Check model facts, not only appearance: expected objects exist, parameter expressions are correct, body/profile counts make sense, and affected features are healthy.
4. If verification fails, inspect the actual state before deciding whether to revise or undo. Do not blindly rerun. One undo reverses one committed Fusion transaction and may not revert an entire multi-feature script; use repeated undo only when the exact transaction scope is known.
5. Leave the document unsaved unless the user explicitly requested a save.

Report the document acted on, the components/features/parameters changed, how the result was verified, and whether the document remains modified or was explicitly saved.
