# Design guide — servo mount

Generic, reusable knowledge on how to design this category of
piece. No project-specific dimensions here (that's the role of the
per-project spec sheet) — only the method: what geometric approach,
what parameters must always be fixed, what pitfalls are known.

## Use cases covered
Any mount/housing that must accept a hobby-style RC servo (analog
or digital) as an insert — 9g micro (SG90/MG90S-class), mini
(MG92B/DS3218-mini-class), or standard full-size — whether retained
by ear tabs or by screwing into the servo body's own bosses.

## Recommended geometric approach
Model the housing as a **negative** (pocket/cavity) sized to the
servo body envelope plus clearance — never as a positive shape
built around an idealized servo silhouette.

Two mounting families are common, pick one deliberately:
- **Ear-tab mount** — a close-fitting rectangular pocket with a
  shoulder the servo's ear tabs rest on; retained by screws through
  the tabs into the surrounding print/part.
- **Slot/cage mount** — a full 4-wall pocket, servo held by screws
  through the case wall directly into the servo body's own threaded
  bosses (only available on servos with threaded metal bodies).

Order of operations: rough pocket first (sized to envelope +
clearance) → ear-tab shoulders / screw bosses → horn bore and wire
channel last. The horn bore and wire channel are the most
position-sensitive cuts and should reference the already-
consolidated pocket geometry, never be modeled before it exists.

Model the horn/output-shaft clearance as a full cylindrical
through-bore of at least horn diameter + swing clearance, extended
through the pocket face — not just a point. If the horn arm is
offset from the shaft centerline, it sweeps an arc and needs radial
clearance too, not just axial clearance at the shaft.

## Parameters to always fix explicitly
- Servo body envelope (L × W × H) for the specific size class in
  use — never assume one fixed number for a whole class (e.g. "9g")
  without checking the actual unit; body width in particular varies
  across manufacturers even within one nominal size class.
- Clearance around the body (typical 0.3–0.5mm per side for FDM
  print fit; tighter for machined/resin parts).
- Ear tab thickness and screw hole diameter matching the actual
  screw used — never assumed as a round number like "M2" without
  checking the tab.
- Horn diameter and sweep clearance (horn arc radius + margin).
- Wire exit direction relative to final assembly orientation — must
  not be routed through a load path or a zone the horn sweeps
  through.

## Known pitfalls
- Modeling the pocket at exact nominal servo dimensions with zero
  clearance: FDM shrinkage and wall artifacts alone can make a
  "perfect fit" pocket require sanding, or not fit at all. Always
  print a test pocket before committing a full part.
- Assuming all servos in one nominal size class share identical body
  width: clones commonly vary ±0.2–0.5mm even when their ear-tab hole
  spacing is standardized. Budget clearance for cross-manufacturer
  variance, not just one datasheet.
- Assuming the horn bore is concentric with the body centerline:
  some servos are not perfectly symmetric. Verify against the actual
  unit or its datasheet drawing.
- Sizing the ear-tab shoulder to the tab's nominal thickness with no
  relief: tabs are often molded slightly proud or warped and need an
  under-cut to seat flat.

## Category-specific verification
- Dry-fit the actual physical servo before locking geometry — a
  numeric bounding-box overlap check is not enough; thin ear tabs
  and horn clearance are easy to get numerically fine but physically
  tight.
- Rotate the horn through its full range (0–180° typical, or
  continuous) and confirm no collision with housing walls at either
  extreme.

## External references
Standard 9g/micro footprint drawings are widely published by
multiple manufacturers (e.g. common SG90-class datasheets) — useful
as a starting cross-reference, but treat as approximate given the
clone variance noted above.

## History
- 2026-07-02 — created generically (not tied to one project's exact
  servo unit), pending refinement once a first real servo's measured
  dimensions are logged in a project's spec sheet.
