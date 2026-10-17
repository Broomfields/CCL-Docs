---
title: Creative Coding Lab
---

*Design Document v0.3 · Living Design Document*

| **Field**        | **Value**                                                                   |
| ---------------- | --------------------------------------------------------------------------- |
| Status           | Living design document                                                      |
| Version          | 0.3                                                                         |
| Initial platform | Lua + LÖVE                                                                  |
| Project type     | Personal creative-coding platform and collection of independent experiments |

# 1. Overview

Creative Coding Lab is a lightweight creative-coding environment for
building visual, interactive programming experiments. It exists to make
mathematical, simulation, procedural-generation, graphics, and other
computational ideas easy to explore through code.

The Lab is not intended to become a general-purpose game engine. Its
purpose is to remove repetitive infrastructure so that a new experiment
can get quickly to the question being explored.

Each experiment is an independent project and portfolio item. The shared
Creative Lab runtime supplies reusable equipment such as application
lifecycle integration, input, timing, rendering helpers, cameras,
experiment UI, and debugging facilities.

Core summary:

A lightweight LÖVE-based creative coding runtime, paired with a standard
experiment template, that makes it easy to create independent visual
programming experiments while keeping shared infrastructure out of the
way of the ideas being explored.

# 2. Goals

- Make it quick to move from an idea or question to a running visual
  experiment.

- Learn programming, mathematics, graphics, simulation, and related
  concepts by implementing and manipulating them.

- Keep shared infrastructure reusable without allowing the Lab to become
  an engine-building project for its own sake.

- Give each experiment enough independence to stand as its own public
  repository and portfolio item.

- Use clean, modern engineering practices for shared infrastructure
  while allowing exploratory experiment code to remain flexible.

- Document the premise, setup, intended learning, and eventual findings
  of each experiment.

- Allow the platform to evolve in response to real experiments rather
  than speculative requirements.

# 3. Core Philosophy

## 3.1 Start with a question

Experiments should normally begin with something to explore rather than
a product requirement: What happens if several bodies interact under
Newtonian gravity? What is a Fourier transform doing visually? How do
boids produce flocking behaviour?

## 3.2 Learn by building

The objective is not merely to reproduce an established result. The
implementation, manipulation of parameters, visualisation, and
observation are part of the learning process.

## 3.3 The Lab provides the equipment

The Lab should understand generic concepts such as rendering, cameras,
input, timing, UI, and debugging. It should not understand planets,
boids, Fourier transforms, terrain, or other experiment-specific
domains.

## 3.4 Earn abstractions

Reusable concepts should normally begin inside an experiment. If
multiple experiments demonstrate that a concept is genuinely general, it
may be promoted into the Lab, cleaned up, documented, and tested.

## 3.5 Prefer obvious code

The project may use data-oriented, functional, object-oriented, or other
approaches where they make the code clearer. No programming paradigm is
imposed as an architectural goal.

## 3.6 Architecture serves experimentation

The architecture serves the experiments—not the other way around.
Infrastructure should reduce friction, not require experiments to
conform to unnecessary ceremony.

# 4. Technology

| **Technology** | **Role**                                                                       |
| -------------- | ------------------------------------------------------------------------------ |
| Lua            | Primary language for the Lab and experiments.                                  |
| LÖVE           | Initial application, graphics, input, and runtime framework.                   |
| POSIX shell    | Bootstrap, dependency-fetching, and development scripts.                       |
| Markdown       | Project documentation and experiment lab-notebook documentation.               |
| Git            | Source control and version/tag foundation.                                     |
| GitHub         | Repository hosting, releases, distribution, and public portfolio presentation. |

VS Code is the intended local editor and may be used with a multi-root
workspace for convenience, but it is not a Creative Lab technology
dependency. The repositories must remain editor-agnostic.

# 5. High-Level Architecture

Creative Lab  
│  
├── Shared runtime / reusable infrastructure  
│  
└── Experiment template  
│  
├── Experiment A  
├── Experiment B  
└── Experiment C  
  
Within an experiment:  
  
Experiment coordinator  
│  
├── Experiment-specific data  
├── Simulation / domain logic  
├── Experiment-defined instrumentation  
└── Lab runtime  
├── lifecycle  
├── rendering  
├── input  
├── camera  
├── timing  
├── UI shell  
└── debugging

# 6. Lab Runtime

Creative Lab is shared infrastructure used by independent experiments.
Its API should remain small and should grow in response to demonstrated
needs.

## 6.1 Intended responsibilities

- LÖVE lifecycle integration and application setup.

- Frame timing and simulation timing support.

- Input handling.

- 2D rendering and drawing helpers.

- Camera facilities.

- A standard collapsible sidebar and UI controls.

- Lab-provided debugging and instrumentation facilities.

- Generic visualisation utilities that have demonstrated reuse across
  experiments.

## 6.2 Non-responsibilities

- Experiment-specific domain models.

- Experiment-specific simulation rules.

- Premature abstractions for concepts not yet shown to be reusable.

- Owning or cataloguing the collection of experiments.

# 7. Experiments

Each experiment is an independent runnable LÖVE application and an
independent public Git repository. It owns its entry point, its domain
vocabulary, its simulation logic, its experiment-specific data, and the
controls it exposes through the Lab's instrumentation UI.

Experiment code is exploratory by design. It may be refactored when
useful, but production-style ceremony must not become a prerequisite for
trying an idea.

# 8. Experiment Entry Point

The experiment owns main.lua. The Lab supplies runtime facilities rather
than acting as the application itself. A possible eventual entry shape
is:

local Lab = require("vendor.lab.runtime")  
local Experiment = require("experiment")  
  
Lab.run(Experiment)

The exact public lifecycle API remains subject to implementation
experience.

# 9. Simulation and Domain Logic

Where useful, the mathematical or simulation core should be separated
from rendering, input, and other side effects. 'Pure-ish' code means
code that primarily accepts data, performs calculations or state
transitions, and produces results without directly depending on LÖVE or
UI concerns.

Purity is a tool rather than a rule. Clarity and ease of experimentation
take priority.

# 10. Experiment Data

Experiment-specific data definitions belong with the experiment. Lua
tables, constructors, and small helper modules should be used
idiomatically rather than reproducing object models from other
languages.

For an orbital experiment, for example, the experiment may define planet
data while separate simulation modules implement gravity and
integration.

# 11. Ownership Rule

| **Concern**                                                | **Owner**              |
| ---------------------------------------------------------- | ---------------------- |
| Generic lifecycle, input, timing, rendering infrastructure | Lab                    |
| Camera infrastructure                                      | Lab                    |
| Sidebar shell and UI widgets                               | Lab                    |
| Experiment instrumentation controls and their meaning      | Experiment             |
| Debugging facilities                                       | Lab                    |
| Domain data and terminology                                | Experiment             |
| Simulation/domain rules                                    | Experiment initially   |
| Proven reusable concepts                                   | May be promoted to Lab |

Promotion into the Lab is a deliberate boundary: exploratory code
becomes shared infrastructure and should then be generalized,
documented, and tested.

# 12. Repository Architecture

creative-coding/ \[local parent; not a Git repo\]  
├── creative-lab/ \[GIT REPO\]  
├── creative-lab-experiment/ \[GIT REPO / GitHub template\]  
├── orbital-mechanics/ \[GIT REPO\]  
├── boids/ \[GIT REPO\]  
├── fourier-transform/ \[GIT REPO\]  
└── creative-coding.code-workspace \[local convenience\]

The local parent folder provides a solution-like view without creating
repository coupling. A VS Code multi-root workspace may be used here,
but is optional.

# 13. Avoiding Git Submodules

Git submodules are intentionally avoided. Requiring a child repository
commit followed by a parent pointer update introduces workflow friction
that conflicts with the Lab's goal of making experiments quick to create
and change.

Repositories remain independent and dependency retrieval is handled by
bootstrap tooling.

# 14. Lab Dependency

An experiment retrieves the Lab into its own runnable tree rather than
relying on a particular sibling-directory layout.

orbital-mechanics/  
├── .lab-version  
├── main.lua  
├── experiment.lua  
├── EXPERIMENT.md  
├── data/  
├── simulation/  
├── scripts/  
│ └── fetch-lab.sh  
└── vendor/  
└── lab/ \[ignored dependency\]

The vendor directory keeps the runnable experiment self-contained while
avoiding Git submodules. vendor/lab is not committed to the experiment
repository.

# 15. Fetch Lab Script

The experiment template includes a POSIX shell bootstrap script that
retrieves the required Creative Lab release into vendor/lab.

The current direction is for the experiment to declare the required Lab
version independently—for example through a small .lab-version file—and
for fetch-lab.sh to consume that declaration rather than hard-code a
version.

The complete versioning and dependency-management policy is deliberately
not finalized in v0.2; see the Design Agenda.

# 16. Versioning and Dependency Management

Versioning is a defined part of the Creative Coding Lab architecture. Three independent version streams exist: the Creative Lab runtime, the experiment template, and each individual experiment.

## 16.1 Creative Lab versioning

Creative Lab uses Semantic Versioning. Major releases may introduce breaking changes and require deliberate migration by experiments. Minor releases add backwards-compatible functionality. Patch releases contain backwards-compatible fixes or small improvements.

An experiment may automatically update to newer minor or patch releases within its current Lab major version. Crossing a Lab major-version boundary is never automatic. The Lab itself only declares its own version and does not need to know which Template or experiment versions consume it.

## 16.2 Experiment Template versioning

The Experiment Template is independently versioned using the same Semantic Versioning compatibility contract. Template major releases may make changes that cannot safely be applied automatically to existing experiments. Template minor and patch releases must be safe, backwards-compatible updates for experiments using the same Template major version.

Template and Lab versions are deliberately independent. A Template release may change scripts, workflows, documentation, or other inherited infrastructure without requiring a Lab release, and vice versa.

## 16.3 Experiment release versioning

Experiment releases use a simpler maturity/revision scheme rather than Semantic Versioning compatibility rules. Experiments begin with releases such as `v0.1`, `v0.2`, and `v0.3` while the exploration is developing. `v1.0` represents the point at which the experiment has achieved the goal described by its premise. Later revisions may use `v1.1`, `v1.2`, and so on.

Experiment versions communicate maturity and revision, not compatibility guarantees.

## 16.4 Experiment manifest

The Experiment Template provides a TOML manifest that becomes the authoritative version metadata for an experiment. It records at least:

```toml
experiment_version = "0.1"
template_version = "1.0.0"
lab_version = "1.0.0"
```

The exact filename and final schema remain implementation details. The Template seeds the manifest; after creation, the experiment owns it. Template and Lab update tooling updates the corresponding values when dependencies change.

The Lab itself remains simpler and only needs to carry its own version metadata.

## 16.5 Compatible dependency updates

Template-provided tooling should inspect the experiment manifest, compare the recorded Template and Lab versions with available releases, and update to the newest compatible minor or patch release within the current major version. A specific compatible version may also be requested explicitly.

Major-version upgrades are excluded from automatic updates because they represent deliberate migration events.

## 16.6 Release provenance

Every published experiment release should record the experiment release version, Template version, and Creative Lab version used to produce it. Release automation should read these from the manifest and include them in the GitHub release metadata or release notes.

## 16.7 Release and deployment infrastructure

The Experiment Template supplies standard release and deployment machinery inherited by newly created experiments. This may include POSIX-shell scripts for bootstrap and dependency updates, GitHub Actions for validation/build/release/deployment, generation of browser-compatible artifacts where supported, and publication to GitHub Pages.

The Template repository also has its own workflows for validating and publishing Template releases. Those workflows are distinct from the experiment workflows that the Template distributes.

The decision to increment a major, minor, or patch Template/Lab version remains deliberate. Automation may publish a chosen version, but should not decide compatibility semantics on behalf of the programmer.

## 16.8 Web release target

Where practical, experiments should be publishable as browser builds using an appropriate LÖVE web compatibility layer/runtime, with GitHub Pages as the intended static hosting target.

The deployed page should correspond to a released experiment version rather than arbitrary development state. A hosted experiment should be suitable for direct use and, where browser and security constraints permit, embedding into another website through an `iframe`.

External sites or CMSs may store the hosted experiment URL as metadata and embed or link to it. That integration is outside Creative Lab's core responsibilities.

# 17. Experiment Template Repository

creative-lab-experiment is a GitHub template repository used to create
new experiment repositories with the standard structure and bootstrap
tooling already in place.

creative-lab-experiment/  
├── .lab-version  
├── .template-version \[proposed provenance marker\]  
├── main.lua  
├── experiment.lua  
├── conf.lua  
├── README.md  
├── EXPERIMENT.md  
├── data/  
├── simulation/  
├── scripts/  
│ └── fetch-lab.sh  
├── vendor/  
│ └── .gitkeep  
└── .gitignore

The exact presence of data/ and simulation/ should remain flexible; they
are organizational defaults rather than mandatory architecture.

The template itself is maintained to production quality, but it
deliberately does not impose a test structure on generated experiments.

# 18. Experiment Documentation

Every experiment includes an EXPERIMENT.md document generated from the
template. It acts as a small lab notebook and captures why the
experiment exists, what is being tested or explored, how the scenario is
set up, and what is learned.

## 18.1 Standard structure

\# Experiment  
  
\## Premise  
What concept or behaviour is this experiment exploring?  
  
\## Question  
What am I trying to understand?  
  
\## Setup  
What scenario am I constructing to explore it?  
  
\## Things to Explore  
What variables, approaches, or behaviours do I want to manipulate?  
  
\## What I Learned  
Filled in as the experiment develops.

README.md and EXPERIMENT.md have distinct roles. README.md explains the
repository and how to run it; EXPERIMENT.md explains the exploration
itself.

# 19. Testing Policy

## 19.1 Creative Lab

Creative Lab is shared infrastructure and should be treated as
production-quality code. Testable behaviour—particularly pure
mathematical utilities, coordinate transforms, timing logic, camera
calculations, and other deterministic infrastructure—should have
automated tests.

Code promoted from an experiment into the Lab should be generalized and
accompanied by appropriate tests before it becomes relied upon as shared
infrastructure.

## 19.2 Experiments

Experiments have no testing requirement. They are intentionally
exploratory and should not accumulate mandatory test ceremony that
discourages rapid experimentation.

## 19.3 Experiment template

The template itself should be maintained carefully and validated as a
production-quality starting point, but generated experiments should not
inherit a mandatory test suite or testing structure.

# 20. Proposed Lab Structure

creative-lab/  
├── lab/  
│ ├── runtime.lua  
│ ├── camera.lua  
│ ├── input.lua  
│ ├── time.lua  
│ ├── drawing.lua  
│ ├── ui/  
│ └── debug/  
├── tests/  
├── README.md  
└── LICENSE

This structure is indicative rather than final. Modules should emerge
from real requirements rather than being created simply to fill an
architectural diagram.

# 21. Proposed Experiment Structure

experiment-name/  
├── .lab-version  
├── main.lua  
├── experiment.lua  
├── conf.lua  
├── README.md  
├── EXPERIMENT.md  
├── data/  
├── simulation/  
├── scripts/  
│ └── fetch-lab.sh  
├── vendor/  
│ └── lab/  
└── .gitignore

# 22. Experiment UI

Creative Lab owns a standard application UI shell while the experiment
remains visually dominant. The normal application view is the experiment
rendering area with a small menu button in the top-left corner.

Activating the menu toggles a sidebar. The sidebar is Lab infrastructure
and contains two tabs: Instrumentation and Debugging.

## 22.1 Instrumentation tab

Instrumentation is rendered by the Lab but defined by the experiment.
The Lab supplies reusable widgets such as sliders, checkboxes, buttons,
labels, and other controls; the experiment chooses which controls exist
and what they affect.

function experiment.instrument(ui)  
ui.slider("Gravity", ...)  
ui.slider("Time Scale", ...)  
ui.checkbox("Show Trails", ...)  
ui.button("Reset", ...)  
end

Instrumentation is part of the experiment's normal interactive
experience, not merely developer debugging.

## 22.2 Debugging tab

The Debugging tab is both supplied and defined by the Lab. It provides
common development and inspection facilities consistently across
experiments.

- FPS and frame-time information.

- Pause and resume.

- Single-step simulation where supported by the runtime model.

- Simulation-speed controls.

- Coordinate and grid overlays.

- Vector visualisation.

- Debug drawing and bounds.

- Object/selection inspection where meaningful.

- Logging facilities.

## 22.3 Sidebar behaviour

The initial preference is for the sidebar to overlay the experiment
rather than resize the experiment viewport. This avoids changing
simulation presentation or rendering dimensions merely because controls
were opened. The behaviour may be revisited if real experiments
demonstrate a better approach.

# 23. Rendering and Camera Direction

## 23.1 Initial 2D implementation

Creative Lab will initially implement 2D rendering and camera
facilities. The first implementation should remain straightforward and
should not build speculative 3D abstractions before they are needed.

## 23.2 Anticipated 3D capability

3D rendering is an expected future capability because some experiments
will benefit substantially from three-dimensional visualisation. It will
be designed and implemented when a concrete experiment provides a real
use case.

Introducing 3D may require a significant redesign of rendering and
camera systems. When that happens, preserving experiment-facing APIs and
minimizing migration work for existing experiments should be preferred
where reasonably possible.

Initial 2D architecture should therefore avoid unnecessarily encoding
assumptions that would make future 3D support prohibitively difficult,
without attempting to pre-design the future 3D system.

# 24. State Management and Data Flow

State management is intentionally unresolved in v0.2. The issue extends
beyond state machines and includes ownership, lifetime, mutation, reset
behaviour, data flow, and the boundaries between Lab state and
experiment state.

A representative experiment should be used to design this rather than
choosing a pattern in the abstract. A gravitational simulation with
simulation state, camera state, instrumentation values, selection state,
and Lab debug state is a suitable candidate.

## 24.1 Questions for the next iteration

- Which state is owned by the Lab and which by the experiment?

- How is simulation state represented and passed to simulation
  functions?

- How do instrumentation controls read and mutate experiment state?

- What does pause stop, and what continues to update while paused?

- How are reset and restart semantics defined?

- Should update functions mutate state, return state, or use a mixed
  approach?

- How are camera, UI, debug, and selected-object state separated from
  simulation state?

This is the second major architectural topic scheduled for the next
design iteration.

# 25. Experiment Evolution and Related Experiments

An experiment repository represents an exploration rather than a permanently frozen scenario. Closely related developments should normally remain within the same experiment when they are natural extensions of the same underlying question.

For example, a Newtonian-gravity experiment may evolve from two bodies to three or more bodies without requiring a separate repository if the underlying exploration remains the same. The number of bodies may become an interactive parameter where appropriate.

A separate experiment repository should be created when the new work represents a meaningfully different question or exploration. Forking is not the default mechanism. Small amounts of experiment-specific code may be copied when beginning a related experiment.

If the same underlying code is repeatedly useful across experiments, that is evidence that the concept may have earned promotion into Creative Lab. Promotion should generalize, document, and test the reusable implementation.

# 26. Playground / Catalogue

A future public playground or catalogue repository may act as a front
door to the wider Creative Coding Lab collection. It can link to the
Lab, template, and individual experiments; present screenshots,
animations, and descriptions; and optionally provide bootstrap scripts
for cloning a curated or complete collection.

The catalogue must reference projects rather than own them. Neither the
Lab nor any experiment should require the catalogue to function.

# 27. Packaging and Distribution

An experiment should ultimately be capable of being packaged with its
required Lab dependency present in vendor/lab. Exact packaging mechanics
will be determined after the basic development workflow is proven.

GitHub releases are expected to be part of the distribution model for
the Lab and template, with experiment release practices allowed to
evolve according to need.

# 28. Architectural Principles

- Optimize for experimentation.

- Prefer obvious code.

- Keep the Lab generic.

- Keep experiments expressive in their own domain vocabulary.

- Separate concept from presentation where useful.

- Push side effects toward the edges where doing so improves clarity.

- Do not impose a programming paradigm unnecessarily.

- Earn abstractions through repeated use.

- Treat promoted shared infrastructure as production-quality code.

- Avoid premature engine development.

- Design for 2D now without closing the door on 3D.

- Keep development tools optional; the project should remain
  editor-agnostic.

The Lab provides the equipment. The experiment provides the question.

# 29. Initial Development Strategy

1. Design state ownership and data flow against a representative experiment.
2. Define the initial branching strategy and commit/workflow conventions.
3. Create the Creative Lab repository and establish its Semantic Versioning and release metadata.
4. Implement the minimum viable 2D runtime and UI shell.
5. Create the Experiment Template, TOML manifest, dependency-update scripts, and inherited GitHub Actions.
6. Create the first real experiment.
7. Establish the experiment release and GitHub Pages deployment path.
8. Observe friction, missing abstractions, and unnecessary abstractions.
9. Promote genuinely reusable concepts into the Lab with documentation and tests.
10. Create another experiment and repeat.

# 30. Remaining Open Questions

The following questions remain intentionally open and should be resolved through focused design discussion or implementation experience.

- **State management and data flow:** ownership, mutation, lifecycle, pause/reset semantics, UI interaction, and simulation boundaries.
- **Branching strategy:** a lightweight model for the Lab, Template, and experiments that allows unfinished work to be committed and pushed without treating it as release-ready.
- **Commit metadata and workflow directives:** whether standardized commit-message markers should influence GitHub Actions, such as allowing tests while suppressing builds/deployments, suppressing release behaviour, or expressing version-increment/release intent.
- Exact shape and naming of the public Lab API.
- How directly experiments may call LÖVE APIs versus using Lab wrappers.
- Exact experiment lifecycle callbacks.
- Detailed camera API for the initial 2D implementation.
- Exact UI widget API and binding model.
- Exact debugging/inspection capabilities and their rollout order.
- Final browser-build tooling and packaging mechanics.
- When the first 3D experiment warrants the rendering overhaul and what that architecture becomes.

Versioning and dependency management is no longer a principal open architectural question in v0.3. Its compatibility model, independent version streams, manifest direction, and release provenance are now established, although implementation details may still be refined.

# 31. Non-Goals — For Now

- Building a general-purpose game engine.

- Designing a complete 3D renderer before a real experiment needs one.

- Creating a universal plugin/package ecosystem.

- Enforcing production-quality structure or tests inside exploratory
  experiments.

- Abstracting every LÖVE API pre-emptively.

- Solving hypothetical future requirements without evidence from
  experiments.

# 32. Definition of Success

Creative Coding Lab succeeds when an idea such as 'what happens if I
simulate several bodies under Newtonian gravity?' quickly becomes work
on gravity, integration, visualisation, and experimentation rather than
an evening spent rebuilding window setup, input handling, camera
controls, sliders, and debugging infrastructure.

A successful experiment should also leave behind a comprehensible record
of what was being explored and what was learned.

# 33. Current Architectural Summary

| **Area**          | **v0.2 direction**                                                                               |
| ----------------- | ------------------------------------------------------------------------------------------------ |
| Runtime           | Independent Creative Lab repository; reusable shared infrastructure.                             |
| Experiments       | Independent public repositories and runnable LÖVE applications.                                  |
| Template          | Independent GitHub template repository; production-quality starting point.                       |
| Dependencies      | Lab fetched into vendor/lab; version declaration required; full policy pending.                  |
| Documentation     | README.md plus structured EXPERIMENT.md lab notebook.                                            |
| Testing           | Required/expected for Lab infrastructure; not imposed on experiments.                            |
| UI                | Lab-owned collapsible sidebar with experiment-defined Instrumentation and Lab-defined Debugging. |
| Rendering         | 2D first; 3D anticipated and implemented only when justified by an experiment.                   |
| State             | Dedicated design topic for v0.4.                                                                 |
| Development tools | Editor-agnostic; VS Code workspace is optional convenience only.                                 |

# 34. Design Agenda for v0.4

Versioning and dependency management is now substantially defined. The next design iteration should focus primarily on state management while also resolving source-control and workflow conventions that affect day-to-day development.

## 34.1 State Management & Data Flow

Define ownership, mutation, lifecycle, pause/reset semantics, UI interaction, and simulation boundaries using a representative experiment.

## 34.2 Branching Strategy

Define a lightweight branching model for the Lab, Template, and experiments that allows work-in-progress code to be committed and pushed safely without requiring the default branch to contain broken or incomplete work.

Topics include feature branches, expectations for the default branch, merge criteria, whether the three repository types share the same rules, and how releases interact with branches.

## 34.3 Commit Metadata and Workflow Directives

Explore whether standardized markers in commit messages should influence GitHub Actions and other automation.

Possible uses include allowing tests while suppressing builds or deployments, suppressing release behaviour, explicitly requesting release-related behaviour, expressing version-increment intent for SemVer repositories, and distinguishing ordinary development commits from release-triggering commits.

The exact syntax and semantics are intentionally unresolved. The eventual design should avoid making ordinary commits fragile or imposing excessive commit-message ceremony.

# 35. Document Status

Version 0.2 incorporates the first review of the original architecture.
It establishes experiment documentation, the testing boundary, the Lab
UI model, the 2D-first/future-3D policy, POSIX-shell bootstrap tooling,
and a stronger repository/template structure. It deliberately leaves
versioning/dependency management and state/data flow as the two
principal design decisions for the next iteration.

This remains a living document. Its purpose is to record current intent
and decisions while allowing real experiments to challenge and improve
the architecture.
