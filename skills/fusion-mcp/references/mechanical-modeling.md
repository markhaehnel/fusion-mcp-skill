# Mechanical modeling best practices

Read this reference for mechanical geometry, parameters, assemblies, or modeling scripts executed through Fusion MCP.

## Preserve design intent

- Prefer parametric, timeline-native features when the design is parametric and future edits matter. Do not switch modeling mode merely to simplify a script; changing or removing design history can be destructive.
- Represent important dimensions and relationships with clearly named user parameters and expressions. Reuse existing parameters and local naming conventions rather than introducing parallel controls.
- Prefer standard Fusion features—sketch, extrude, revolve, hole, shell, pattern, mirror, fillet, chamfer, and construction geometry—over finalizing geometry as transient or directly edited B-Rep when a native editable feature expresses the intent.
- Use direct modeling only when the user asks for it, the source has no useful history, or a parametric feature would add no meaningful editability. Preserve the active design's existing modeling style.

A robust default feature order is primary parameters, component structure, construction geometry, simple driving sketches, primary form features, repeated features, then edge details. Depart from it when dependencies or the user's intent require another order.

## Components and assemblies

- Use the existing default/root component for a true single-part design when that matches the document's structure. Use a separate component for each independently manufactured, documented, reused, or moving part. Bodies are appropriate for intermediate geometry or inseparable multi-body construction.
- When a separate component is needed, create, name, and activate it before creating its geometry, and create sketches/features in that component's context. Do not create root geometry and move it into a component afterward unless that restructuring is the task.
- Use descriptive names for components, bodies, sketches, construction geometry, parameters, and important features. Preserve an established naming scheme.
- Distinguish native objects from assembly-context proxies. Use `nativeObject` and `createForAssemblyContext(occurrence)` only after checking the live API documentation for the concrete entity type. Do not combine selections from incompatible component contexts.
- Apply transforms and joints at the occurrence/assembly level; do not bake placement into part geometry when motion or reuse is intended.

## Units and parameters

- Fusion Design API database units are centimeters for length and radians for angles. API categories such as CAM can use different conventions; verify the relevant live documentation before mixing Design and CAM values.
- Prefer `ValueInput.createByString` with an explicit unit such as `"12 mm"`, or a named parameter expression, for values that become editable parameters. A unitless string follows the active document units and is therefore ambiguous.
- Use `ValueInput.createByReal` only for already calculated values in the API's database units. Convert deliberately with `UnitsManager` when values enter or leave the API.
- Give user parameters unique, stable names, compatible unit types, and useful comments when the API supports them. Avoid silently replacing an existing parameter with a different meaning.

## Sketches and references

- Keep driving sketches small and purpose-specific. Fully constrain the geometry that controls the result, using dimensions and geometric constraints rather than fixed coordinates where practical.
- Prefer origin planes, axes, construction planes, construction lines, and named parameters as references. Referencing generated faces or edge indices creates fragile downstream dependencies when topology changes.
- Project only geometry that is necessary for design intent; excessive projected geometry and cross-component dependencies make recomputation brittle.
- Before consuming a profile, verify that the sketch produced the expected closed regions. Do not choose `profiles.item(0)` or an arbitrary face/edge solely because it is first; identify the intended entity by geometry, location, area, or another stable invariant.
- Entity tokens are not permanent IDs. Resolve and validate them in the current design before use, and do not assume the returned token string stays identical over time.

## Feature construction

- Use feature input objects so extents, operations, directions, participant bodies, and creation context are explicit. Verify the current signatures with `apiDocumentation` before scripting.
- Prefer extent definitions tied to design intent—such as through-all, to-object, symmetric, or parameter expressions—over guessed fixed distances when they better describe the requested result.
- Specify whether an operation creates a new body/component, joins, cuts, or intersects. Never rely on an implicit default that could affect unintended bodies.
- Add fillets and chamfers late when possible, and identify their edges by geometric criteria. Edge collection order is not a stable design reference.
- Use `TemporaryBRepManager` for calculation, preview-like construction, or validation. Copy transient geometry into the design only when direct geometry is actually the desired deliverable.

## Preflight and verification

Before the first mutation, check the expected design type, target component, existing names/parameters, required profiles or bodies, and all numeric inputs. If any selector is ambiguous, stop rather than editing an arbitrary entity.

After the change, use documented APIs to verify the invariants relevant to the task, for example:

- component, occurrence, body, sketch, profile, and feature counts;
- user-parameter expressions and evaluated values;
- solid versus surface body type, volume, bounding box, or principal dimensions;
- feature health state and warning/error messages;
- placement, joint, or transform values in the correct assembly context.

Call the appropriate recompute API when required by the documented workflow, then inspect health again. When geometry or placement is visually relevant, pair numeric/API checks with a screenshot that exposes the change. Prefer the current camera when sufficient; use an additional orthographic or isometric direction only when depth or alignment would otherwise remain ambiguous.

## Autodesk references

- [Components and recommended workflow](https://help.autodesk.com/view/fusion360/ENU/?contextId=ASM-COMPONENTS)
- [Modeling modes and the timeline](https://help.autodesk.com/view/fusion360/ENU/?contextId=DESIGN_HISTORY)
- [Sketches and constraints](https://help.autodesk.com/view/fusion360/ENU/?contextId=SKT-3D-SKETCH)
- [Fusion API units](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Units_UM.htm)

Use Fusion MCP's live `apiDocumentation` query for class and member details; these links provide design principles, not a substitute for current signatures.
