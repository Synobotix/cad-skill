# Design guide — DC motor mount

Generic, reusable knowledge on how to design this category of
piece. No project-specific dimensions here (that's the role of the
per-project spec sheet) — only the method: what geometric approach,
what parameters must always be fixed, what pitfalls are known.

## Use cases covered
Any bracket/housing that must hold a small brushed DC motor by its
cylindrical can (e.g. N20 gearmotor, 130/180/280-size hobby motor),
whether face-mounted via the motor's own front screw bosses or
clamped radially around the can.

## Recommended geometric approach
Two mounting families, pick one deliberately:
- **Face-mount** — the motor has a front face with 2 or 4 threaded
  holes on a bolt circle around the shaft bore; the bracket is a
  flat plate with matching clearance holes plus a central bore for
  shaft/bushing clearance.
- **Can-clamp** — no front mounting holes (common on cheap N20/130-
  size motors); the bracket wraps a close-fitting half-cylinder (or
  two half-shells) around the can body, retained by a strap, screws,
  or a press fit, relying on friction rather than the shaft.

For can-clamp mounts: never rely on the shaft or its bushing to
carry radial or axial load — the clamp must grip the can; the shaft
only transmits torque.

Model the shaft interface last, once the can/bracket geometry is
confirmed — it carries the tightest tolerance (shaft diameter, and
D-flat position if present).

Include a torque-reaction feature (D-flat capture, keyway, or a
clamp profile matched to any asymmetry in the can) whenever the
motor housing itself could spin inside the clamp under starting
torque — a purely circular clamp on a round can can slip.

## Parameters to always fix explicitly
- Can diameter and length — measured on the actual unit, not
  assumed from a generic size name (e.g. "130 motor"); variants
  differ between manufacturers.
- Shaft diameter, usable length outside the bracket face, and
  D-flat presence/position if any.
- Mounting screw pattern (bolt circle diameter + hole diameter) for
  face-mount, or clamp strap width/screw diameter for can-clamp.
- Expected stall torque / current draw — needed to size the
  torque-reaction feature; a purely aesthetic clamp is not enough
  once real load is applied.

## Known pitfalls
- Sizing the can bore for a snug press fit calibrated for machining
  tolerances, then printing it via FDM: printed circles are rarely
  truly round on the inside (often faceted or undersized). Oversize
  slightly and test-fit, or use a two-piece clamp with a tightening
  screw instead of relying on a single-piece press fit.
- Ignoring axial motor float: many cheap DC motors have shaft
  end-play. A bracket that constrains the can too tightly axially
  can bind the rotor.
- Assuming the shaft is perfectly concentric with the can: cheap
  motors can have several tenths of a mm of runout. Downstream
  pieces (gears, tendons) need clearance for this — not a
  zero-tolerance fit.

## Category-specific verification
- Spin test (by hand or under power) after mounting, checking for
  binding, before committing to a batch of brackets.
- If the motor drives a mechanism with a hard stop (e.g. a jaw or
  eyelid limit), verify the clamp doesn't slip under stall torque at
  that limit. This is a repeated-load case: it triggers CAD-05 FEM
  verification on the clamp/bracket itself, not just on the driven
  part.

## External references
Generic N20/130/180/280-size motor dimension drawings are widely
published by multiple manufacturers; treat published can diameter as
nominal, with at least ±0.2mm tolerance expected in practice.

## History
- 2026-07-02 — created generically (not tied to one project's exact
  motor unit), pending refinement once a first real motor's measured
  dimensions are logged in a project's spec sheet.
