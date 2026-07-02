---
name: cad-autopilot
description: Extension of the cad-modeler skill allowing Claude to fill in missing dimensions and advance modeling without systematic validation, subject to strict invariants and an auditable assumption log.
trigger: Any cad-modeler session where the user wants to delegate decisions on unspecified values rather than being interrupted at every missing dimension.
version: 0.1
depends_on: cad-modeler
---

# $cad-autopilot — Autonomous mode for cad-modeler

## Principle

Never block on a missing dimension if it can be reasonably derived.
Never guess silently either. Any value set without explicit user
instruction follows the generator → critic → gate loop, and is
logged before being used.

## Invariants (non-negotiable, even in autonomous mode)

- **CAD-01** — Any value not provided and not derivable from an
  existing spec sheet must be chosen by an explicit method
  (documented ratio, standard, sizing calculation) — never a silent
  estimate.
- **CAD-02** — Every assumption is logged in the project's
  `assumptions.md` before use: value, method/source, confidence
  level, affected piece.
- **CAD-03** — The pieces' dependency order (from the project
  breakdown) is strictly respected: never generate a piece whose
  upstream dependency isn't locked and validated.
- **CAD-04** — Before exporting a piece: pass through the critic
  checklist (next section), comparing the produced geometry against
  the spec sheet's dimensions and the project's constraints.
- **CAD-05** — Any piece under mechanical load — load-bearing, but
  also any repeatedly actuated piece (fatigue at servo mounting
  points, linkage/tendon flex zones, rotation axes) — goes through
  FEM verification if the tool is available, or gets flagged `NOT
  VERIFIED — validate before manufacturing` if the tool is absent. A
  purely cosmetic or cover piece with no mechanical stress is
  exempt, but this status must be explicitly justified in its spec
  sheet rather than assumed by default.
- **CAD-06** — No destructive action (overwriting an existing
  .FCStd, deleting a file) without explicit confirmation.
- **CAD-07** — No piece is generated without a corresponding design
  guide in the skill's `resources/` (not to be confused with the
  project's `specs/` — see `spec-template.md` for the per-piece
  instance sheet). If no guide covers the requested piece category:
  STOP, never improvise an undocumented design method from general
  knowledge. Propose a draft guide (using the `resources/_template.md`
  structure, pre-filled as best as possible) and ask the user to
  co-write/validate it before any FreeCAD generation for this
  category. The validated guide is added to the skill (so it's
  versioned and reusable on future projects), not just to the
  current project.

## Decision loop (generator → critic → gate)

**1. $generator** — proposes a value or geometry to fill the gap,
with the method used and a confidence level (high/medium/low).

**2. $critic** — reviews the proposal against: the piece's spec
sheet, the constraints in `notes.md`, and invariants CAD-01 through
CAD-07. Lists any discrepancies or inconsistencies found.

**3. $gate** — decides:
- **GO**: assumption logged in `assumptions.md`, FreeCAD generation
  launched, export performed
- **STOP**: back to the user (see blocking cases below), no
  generation until a response is received

## Cases where the gate always blocks (even in autonomous mode)

- Low confidence on an assumption that impacts an already-
  manufactured or locked piece
- Two reasonable methods produce incompatible results (genuine
  ambiguity, not simple lack of info)
- The piece carries a mechanical risk to a user or bystander (pinch
  point, jaw force near a face during demonstrations) — always
  flagged explicitly, never resolved silently
- Before any export intended for final manufacturing (as opposed to
  a test/prototype)

## assumptions.md format

One file per project, one line per assumption:

```markdown
## <date> — <affected piece>
- Value used: <value>
- Method: <anatomical ratio / standard / calculation / other>
- Confidence: high / medium / low
- Impact if wrong: <cascading affected pieces>
- Status: to validate / implicitly validated (no feedback) / explicitly validated
```

## What this changes in FreeCAD prompts

Block 2 of the prompt template (cad-modeler, "precise geometry") can
now contain a value set by Claude rather than waiting on the user,
provided block 1 of the reply message always starts with:
"Assumption made: [value] — [method] — confidence [level]. Logged in
assumptions.md." The user sees the decision without having had to
provide it, and can correct it after the fact without it blocking
progress.
