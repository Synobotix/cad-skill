<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<a id="readme-top"></a>



<!-- PROJECT SHIELDS -->
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![project_license][license-shield]][license-url]


<!-- PROJECT LOGO -->
<br />
<div align="center">
<h3 align="center">cad-skill</h3>

  <p align="center">
    A reusable CAD modeling method for FreeCAD via MCP — breakdown into pieces, spec sheets, generic per-category design guides, and an auditable-assumption autopilot mode. Contains no project-specific data.
    <br />
    <a href="cad-modeler.md"><strong>Read the method »</strong></a>
    <br />
    <br />
    <a href="https://github.com/Synobotix/cad-skill/issues/new?labels=bug">Report Bug</a>
    &middot;
    <a href="https://github.com/Synobotix/cad-skill/issues/new?labels=enhancement">Request Feature</a>
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li>
      <a href="#usage">Usage</a>
      <ul>
        <li><a href="#how-it-works">How it works</a></li>
        <li><a href="#creating-a-spec-sheet-for-a-new-piece">Creating a spec sheet for a new piece</a></li>
        <li><a href="#creating-a-design-guide-for-a-new-piece-category">Creating a design guide for a new piece category</a></li>
      </ul>
    </li>
    <li><a href="#versioning">Versioning</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

Without a shared method, every CAD session tends to reinvent its own
approach: dimensions get guessed silently instead of logged, each
piece category is designed ad hoc with no memory of what worked or
failed last time, and there's no forced separation between "generic
engineering knowledge" (how do you design a servo mount in general)
and "this specific project's data" (this project's servo is a
MG92B, mounted at these coordinates). That leads to unreviewable
assumptions baked directly into geometry, and design methods that
have to be re-derived from scratch on every new project.

This skill fixes that by splitting CAD work into two layers:

- **Generic method + design guides** (this repo, `skills/cad/`) —
  how to approach a breakdown, how to write a spec sheet, and
  reusable per-category design guides (`resources/`) capturing
  known-good geometric approaches and pitfalls. Versioned and shared
  across every project that uses this skill.
- **Project-specific data** (in the consuming project, not here) —
  actual dimensions, actual hardware, actual spec sheets, actual
  FreeCAD outputs.

On top of that, `cad-autopilot` adds a set of non-negotiable
invariants (CAD-01 to CAD-07) so that autonomous work — filling in a
missing dimension without asking the user every time — stays
auditable: every assumption is logged with its method and confidence
level in `assumptions.md`, never silently baked in.

**Contents of this repo:**

- `cad-modeler.md` — breakdown into pieces, spec sheets, FreeCAD
  prompt template
- `cad-autopilot.md` — invariants CAD-01 to CAD-07, generator/
  critic/gate loop, systematic blocking cases
- `_spec-template.md` — template to duplicate in each consuming
  project, once per piece
- `resources/` — generic design guides by piece category (CAD-07),
  reusable across projects; `_template.md` is the template for a new
  guide. Currently covers: `servo-mount.md`, `dc-motor-mount.md`.
- `claude-md-snippet.md` — block to paste into a consuming project's
  `CLAUDE.md` (not `CLAUDE.local.md` — it must apply to every
  session on the project, not just a local override) so any session
  in that project knows to use this skill, respect CAD-07, and
  delegate to `cad-agents` automatically

Model pinning per phase (`cad-architect` = opus, `cad-builder` =
sonnet) lives in a separate repo, [`cad-agents`](../cad-agents) —
kept apart because agent definitions must sit under a project's
`.claude/agents/` to be discovered by the harness, which is a
different consumption path than this skill's `skills/cad/`
submodule.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

- A project repo with **Git submodules** support
- A **FreeCAD MCP** server reachable from the Claude session (this
  skill assumes CAD tool calls — document creation, sketch/pad,
  scene info, FEM — are available via MCP, not a manual FreeCAD UI)
- *Optional but recommended*: the [`cad-agents`](../cad-agents) repo,
  if you want the architecture/execution phase split enforced with a
  hard model pin rather than left to the current session's model

### Installation

1. Add this repo as a submodule in the consuming project, plus
   `cad-agents` if you're using the hard model pin:
   ```bash
   git submodule add <cad-skill-repo-url> skills/cad
   git submodule add <cad-agents-repo-url> agents/cad
   git submodule update --init --recursive
   ```
2. Follow `cad-agents`' own README to symlink its agent definitions
   into `.claude/agents/` — a submodule path alone isn't enough for
   the harness to discover `cad-architect`/`cad-builder` as
   invocable agents.
3. Paste the contents of `claude-md-snippet.md` into the project's
   `CLAUDE.md` (the shared, versioned one — not `CLAUDE.local.md`)
   so the method, CAD-07, and the agent delegation are picked up
   automatically in every session on that project, rather than
   having to be re-explained each time.
4. Create the project-side folders that `cad-modeler.md` expects:
   ```
   cad-projects/<project-name>/
     notes.md
     specs/
     parts/
   ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- USAGE EXAMPLES -->
## Usage

### How it works

1. **`notes.md`** — one project-wide brief fixed before the first
   piece: scale, material/manufacturing method, coordinate
   convention, tolerances, manufacturing constraints.
2. **Breakdown** — list subsystems, their geometric dependencies,
   and the resulting design order.
3. **Design guide check (CAD-07)** — for each piece category, verify
   a guide exists in `resources/`. If not, stop and co-write one
   (see below) before designing anything in that category.
4. **Spec sheet per piece** — one `specs/<piece>.md` per piece,
   duplicated from `_spec-template.md` (see below).
5. **FreeCAD generation** — built from the locked spec sheet, one
   piece at a time, respecting dependency order.
6. **Verification** — scene checks, interference checks, FEM where
   relevant (CAD-05), dry-fit before anything irreversible.

In autopilot mode, missing values go through a generator → critic →
gate loop instead of blocking on every gap, but every value used is
still logged in `assumptions.md` with its method and confidence
level — see `cad-autopilot.md` for the full invariant list and the
cases where the gate always blocks regardless of mode.

Architecture decisions (breakdown, dependency graph, guide
authoring) and execution (FreeCAD generation once a spec is locked)
are split across two agents (`cad-architect` / `cad-builder`, see
`cad-agents`) rather than done in one session — see `cad-modeler.md`
for why.

### Creating a spec sheet for a new piece

A spec sheet is **project-specific** — it lives in the consuming
project, not in this skill.

1. Copy `_spec-template.md` to `<project>/specs/<piece-name>.md`.
2. Fill in function, dependencies (which pieces must be locked
   before/after this one), and known dimensions with their source
   (measurement / datasheet / estimate — never blank).
3. Fill in the mechanical verification status (CAD-05): whether the
   piece is load-bearing or repeatedly actuated, and its FEM status
   — or an explicit justification if it's exempt.
4. Attach whatever reference material exists: annotated diagram/
   photo, an off-the-shelf component's STL/STEP, or a manufacturer
   datasheet. Prefer STEP over STL for off-the-shelf parts (see
   `cad-modeler.md`, "Using external resources") — STL can be off by
   a tenth of a mm on mounting holes.
5. Any dimension you don't have yet stays a gap to resolve — either
   ask the user, or, in autopilot mode, run it through the
   generator → critic → gate loop and log the result in the
   project's `assumptions.md` before it's used in FreeCAD.

A spec sheet is considered locked once every dimension has a source,
dependencies are satisfied, and (for CAD-05-relevant pieces)
mechanical verification is done or explicitly not applicable — only
then should FreeCAD generation for that piece start.

### Creating a design guide for a new piece category

A design guide is **generic and reusable** — it lives in this skill
repo (`resources/`), not in a project. Create one whenever CAD-07
blocks because no existing guide covers the requested piece
category — never improvise a design method for the gap instead.

1. Copy `resources/_template.md` to `resources/<category>.md`
   (category name, not project or piece-instance name — e.g.
   `servo-mount.md`, not `left-arm-servo.md`).
2. Fill in **use cases covered** precisely enough that it's obvious
   whether a new request falls under this guide or needs its own.
3. Fill in the **recommended geometric approach**: modeling strategy
   (e.g. negative/pocket vs. positive shape), operation order, and
   why that order matters.
4. Fill in **parameters to always fix explicitly** — the checklist
   of values that must never be silently assumed for this category,
   each with a one-line reason. This is what CAD-01/the generator
   checks against in autopilot mode.
5. Fill in **known pitfalls** and **category-specific verification**
   from real experience — this is the part that actually saves
   rework on the next project, so favor concrete failure modes
   (e.g. "assuming X is concentric with Y — verify against the
   actual unit") over generic advice.
6. Add an **external references** entry if a relevant standard or
   published footprint exists.
7. Log the **history** entry (date + originating context).
8. Co-write and validate the guide with the user before using it to
   generate any FreeCAD geometry (CAD-07) — this step isn't
   optional, even under autopilot.
9. Commit the guide to this repo (bump the version — see below), not
   just to the current project, so every consuming project benefits
   from it on their next submodule update.

`resources/servo-mount.md` and `resources/dc-motor-mount.md` are
good references for the level of detail expected — concrete
parameters and pitfalls, no project-specific dimensions.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- VERSIONING -->
## Versioning

Semantic tags (`v0.1.0`, `v0.2.0`...). A consuming project pins the
submodule to a specific tag rather than the main branch — a change
to an invariant here (e.g. widening CAD-05) must never silently
change the behavior of an ongoing project. Updating the submodule =
an explicit decision, not a side effect of a `git pull`. The same
applies to adding a new `resources/` guide: it only reaches a
consuming project once that project bumps its pinned tag.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTRIBUTING -->
## Contributing

The main way this skill grows is through new `resources/` design
guides — co-written and validated against a real piece (see
[Creating a design guide](#creating-a-design-guide-for-a-new-piece-category)
above), then committed here so every consuming project benefits. Add
new guides as new piece categories come up (e.g. bearing housings,
linkage arms, enclosure shells).

1. Fork the project
2. Create your branch (`git checkout -b feat/your-guide`)
3. Add or update a guide under `resources/`, following the
   `_template.md` structure
4. Bump the version (see [Versioning](#versioning)) if the change
   affects consuming projects
5. Open a Pull Request targeting `main`

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Top contributors:

<a href="https://github.com/Synobotix/cad-skill/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=Synobotix/cad-skill" alt="contrib.rocks image" />
</a>



<!-- LICENSE -->
## License

Distributed under the Creative Commons Attribution-ShareAlike 4.0
International License (CC BY-SA 4.0). See `LICENSE` for more
information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Rémi Boivin - remiboivin021@gmail.com

Project Link: [https://github.com/Synobotix/cad-skill](https://github.com/Synobotix/cad-skill)

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

* [Best-README-Template](https://github.com/othneildrew/Best-README-Template)
* [FreeCAD](https://www.freecad.org/) and its MCP integration
* [`cad-agents`](../cad-agents) — the companion repo enforcing the
  architecture/execution model split

<p align="right">(<a href="#readme-top">back to top</a>)</p>


<!-- MARKDOWN LINKS & IMAGES -->
[stars-shield]: https://img.shields.io/github/stars/Synobotix/cad-skill.svg?style=for-the-badge
[stars-url]: https://github.com/Synobotix/cad-skill/stargazers
[issues-shield]: https://img.shields.io/github/issues/Synobotix/cad-skill.svg?style=for-the-badge
[issues-url]: https://github.com/Synobotix/cad-skill/issues
[license-shield]: https://img.shields.io/github/license/Synobotix/cad-skill.svg?style=for-the-badge
[license-url]: https://github.com/Synobotix/cad-skill/blob/main/LICENSE
