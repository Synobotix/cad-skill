## CAD (FreeCAD via MCP)

This project uses the `cad-modeler` / `cad-autopilot` skill
(submodule at `skills/cad`) for any mechanical design work, plus the
`cad-agents` submodule (`agents/cad`) for the architecture/execution
model split.

- Before any CAD task: read `skills/cad/cad-modeler.md` and
  `skills/cad/cad-autopilot.md` in full — they define the method,
  the folder layout (`notes.md`, `specs/`, `parts/`), and the
  non-negotiable invariants (CAD-01 to CAD-07).
- **CAD-07 in practice**: never invent a design method for a piece
  category not already covered in `skills/cad/resources/`. If no
  guide exists for the requested category, stop and ask to co-write
  one before generating anything in FreeCAD.
- **Model split is mandatory, not advisory**: delegate the
  architecture phase (breakdown, dependency graph, guide
  authoring/gap resolution, ambiguity resolution) to the
  `cad-architect` agent, and the execution phase (FreeCAD generation
  once a piece's spec is locked) to the `cad-builder` agent. Do not
  do either phase directly in the main session, and do not ask the
  user to manually switch models — that's only a fallback if the
  agents aren't set up yet (see `cad-agents`' README for the
  `.claude/agents/` symlink setup).
- Every assumption made on a missing dimension must be logged in the
  project's `assumptions.md` before use (CAD-02) — never a silent
  estimate.
