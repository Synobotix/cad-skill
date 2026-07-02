---
name: cad-modeler
description: General skill for turning a mechanical idea into modeled parts in FreeCAD via MCP, with resource management (diagrams, STL/STEP, images) separated by piece and by project.
trigger: Any request to design, model, or manufacture a mechanical/physical object.
version: 0.1
---

# $cad-modeler — General CAD modeling skill

## Principle

The skill is the method, reusable from one project to another.
Resources (dimensions, diagrams, off-the-shelf component STLs) are
specific to each project and live alongside it, not inside the
skill.

## Recommended folder structure

```
cad-projects/
  <project-name>/
    notes.md              # project-wide brief
    specs/
      <piece-1>.md         # brief + references for this piece
      <piece-1>.stl/.step  # off-the-shelf component to integrate
      <piece-1>.png/.jpg   # annotated reference diagram or photo
    parts/                 # FreeCAD outputs (.FCStd, exported .stl)

skills/cad/                # this skill, as a submodule
  resources/
    <piece-category>.md   # generic design guide (CAD-07),
                            # reusable across projects
```

## Step 0 — Project brief (notes.md)

To be fixed once for the whole project, before the first piece:

- Scale / target overall dimensions
- Material and manufacturing method (3D printing, machining, cutting)
- Coordinate convention: origin, X/Y/Z axis orientation
- Target assembly tolerances
- Manufacturing constraints (printer bed size, minimum wall thickness)

## Step 1 — Breakdown into pieces

For any non-trivial object: list the subsystems, their mutual
geometric dependencies, and the resulting design order (pieces that
others depend on are designed first). One piece = one spec file.

## Step 1bis — Design guide (see cad-autopilot, CAD-07)

Before designing a piece, check that a generic guide exists in
`skills/cad/resources/` for its category. If no guide covers this
category: do not improvise — raise the gap to the user and co-write
the guide before continuing.

## Step 2 — Spec sheet per piece (see separate template)

Each `specs/<piece>.md` file documents:
- Function of the piece and constraints (load, clearance, interface
  with neighboring pieces)
- Known dimensions, with their source (measurement, datasheet,
  estimate)
- Attached files: annotated diagram, STL/STEP of component to
  integrate
- Dependencies: which other pieces must be locked before this one

## Step 3 — FreeCAD generation

Prompt template per piece, built from the spec sheet:

```
Create a new document [piece_name].
If specs/<piece>.step exists: import it as a geometric reference
  before designing adjacent pieces.
Add [primitive or sketch+pad] with [dimensions in mm, taken from
  the spec sheet].
Position according to the project's coordinate convention (notes.md).
Apply the specified fillets/chamfers.
Export as STL/STEP to parts/.
```

## Step 4 — Verification

- `get_scene_info` to validate relative positions between
  already-designed pieces
- If off-the-shelf components are imported (servo, bearing): check
  for geometric interference with `run_script` (bounding box
  overlap) before printing
- `run_fem_analysis` if available, for pieces under significant
  mechanical load (supports, linkage arms)
- STL export and dry-fit assembly test before any irreversible step
  (molding, gluing, final machining)

## Using external resources

### Images and diagrams
Useful for communicating a shape or layout, but no real-world scale
can be inferred without an explicit reference. Always provide at
least one reference dimension (in the image or in text) before
translating a diagram into FreeCAD dimensions.

### STL vs STEP for off-the-shelf components
Prefer STEP when available: exact geometry (parametric surfaces)
rather than a faceted mesh. STL is fine for a visual clearance
check, but can introduce errors on the order of a tenth of a mm on
mounting holes — critical for pieces that must receive a precise
screw or shaft. If the manufacturer provides neither, a datasheet
PDF with dimensions is more reliable than a third-party-generated
STL.

### Model choice by phase — hard-pinned via the cad-agents repo
- **Architecture phase** (Step 0, Step 1, Step 1bis, ambiguity
  resolution): delegate to the **`cad-architect`** agent (`model:
  opus` in its frontmatter — pinned by the harness, not a runtime
  choice). Deeper multi-step spatial reasoning is required here — an
  error at this stage propagates to every downstream piece. This
  agent never generates or exports FreeCAD geometry.
- **Execution phase** (Step 3, Step 4, on a piece whose spec sheet
  is already locked): delegate to the **`cad-builder`** agent
  (`model: sonnet` in its frontmatter). Faster and cheaper, and
  sufficient once the plan is fixed. This agent never makes an
  architectural decision — if the spec sheet or design guide is
  incomplete, it stops and reports back rather than improvising.
- **Setup**: these two agents are defined in the separate
  [`cad-agents`](../cad-agents) repo, not in this skill — added as
  its own submodule and symlinked into the consuming project's
  `.claude/agents/` (a submodule path alone isn't enough for the
  harness to discover them). See `cad-agents`' README for the exact
  commands. This skill only documents the phase split; it doesn't
  pin the model itself.
- Do not ask the user to manually switch models (`/model opus` /
  `/model sonnet`) — that's only a fallback if `cad-agents` isn't set
  up yet in the current project.
