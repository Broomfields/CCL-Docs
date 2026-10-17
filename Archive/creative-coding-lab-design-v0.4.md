---
title: Creative Coding Lab
---

*Design Document v0.4 · Living Design Document*

  ---------------------------------------------------------------------
  **Field**     **Value**
  ------------- -------------------------------------------------------
  Status        Living design document

  Version       0.4

  Initial       Lua + LÖVE
  platform      

  Project type  Personal creative-coding platform and collection of
                independent experiments
  ---------------------------------------------------------------------

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

-   Make it quick to move from an idea or question to a running visual
    experiment.

-   Learn programming, mathematics, graphics, simulation, and related
    concepts by implementing and manipulating them.

-   Keep shared infrastructure reusable without allowing the Lab to
    become an engine-building project for its own sake.

-   Give each experiment enough independence to stand as its own public
    repository and portfolio item.

-   Use clean, modern engineering practices for shared infrastructure
    while allowing exploratory experiment code to remain flexible.

-   Document the premise, setup, intended learning, and eventual
    findings of each experiment.

-   Allow the platform to evolve in response to real experiments rather
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

The architecture serves the experiments---not the other way around.
Infrastructure should reduce friction, not require experiments to
conform to unnecessary ceremony.

# 4. Technology

  ---------------------------------------------------------------------
  **Technology**   **Role**
  ---------------- ----------------------------------------------------
  Lua              Primary language for the Lab and experiments.

  LÖVE             Initial application, graphics, input, and runtime
                   framework.

  POSIX shell      Bootstrap, dependency-fetching, and development
                   scripts.

  Markdown         Project documentation and experiment lab-notebook
                   documentation.

  Git              Source control and version/tag foundation.

  GitHub           Repository hosting, releases, distribution, and
                   public portfolio presentation.
  ---------------------------------------------------------------------

VS Code is the intended local editor and may be used with a multi-root
workspace for convenience, but it is not a Creative Lab technology
dependency. The repositories must remain editor-agnostic.

# 5. High-Level Architecture

Creative Lab\
│\
├── Shared runtime / reusable infrastructure\
│\
└── Experiment template\
│\
├── Experiment A\
├── Experiment B\
└── Experiment C

Within an experiment:

Experiment coordinator\
│\
├── Experiment-specific data\
├── Simulation / domain logic\
├── Experiment-defined instrumentation\
└── Lab runtime\
├── lifecycle\
├── rendering\
├── input\
├── camera\
├── timing\
├── UI shell\
└── debugging

# 6. Lab Runtime

Creative Lab is shared infrastructure used by independent experiments.
Its API should remain small and should grow in response to demonstrated
needs.

## 6.1 Intended responsibilities

-   LÖVE lifecycle integration and application setup.

-   Frame timing and simulation timing support.

-   Input handling.

-   2D rendering and drawing helpers.

-   Camera facilities.

-   A standard collapsible sidebar and UI controls.

-   Lab-provided debugging and instrumentation facilities.

-   Generic visualisation utilities that have demonstrated reuse across
    experiments.

## 6.2 Non-responsibilities

-   Experiment-specific domain models.

-   Experiment-specific simulation rules.

-   Premature abstractions for concepts not yet shown to be reusable.

-   Owning or cataloguing the collection of experiments.

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

local Lab = require("vendor.lab.runtime")\
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

  ---------------------------------------------------------------------
  **Concern**                                      **Owner**
  ------------------------------------------------ --------------------
  Generic lifecycle, input, timing, rendering      Lab
  infrastructure                                   

  Camera infrastructure                            Lab

  Sidebar shell and UI widgets                     Lab

  Experiment instrumentation controls and their    Experiment
  meaning                                          

  Debugging facilities                             Lab

  Domain data and terminology                      Experiment

  Simulation/domain rules                          Experiment initially

  Proven reusable concepts                         May be promoted to
                                                   Lab
  ---------------------------------------------------------------------

Promotion into the Lab is a deliberate boundary: exploratory code
becomes shared infrastructure and should then be generalized,
documented, and tested.

# 12. Repository Architecture

creative-coding/ \[local parent; not a Git repo\]\
├── creative-lab/ \[GIT REPO\]\
├── creative-lab-experiment/ \[GIT REPO / GitHub template\]\
├── orbital-mechanics/ \[GIT REPO\]\
├── boids/ \[GIT REPO\]\
├── fourier-transform/ \[GIT REPO\]\
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

orbital-mechanics/\
├── .lab-version\
├── main.lua\
├── experiment.lua\
├── EXPERIMENT.md\
├── data/\
├── simulation/\
├── scripts/\
│ └── fetch-lab.sh\
└── vendor/\
└── lab/ \[ignored dependency\]

The vendor directory keeps the runnable experiment self-contained while
avoiding Git submodules. vendor/lab is not committed to the experiment
repository.

# 15. Fetch Lab Script

The experiment template includes a POSIX shell bootstrap script that
retrieves the required Creative Lab release into vendor/lab.

The current direction is for the experiment to declare the required Lab
version independently---for example through a small .lab-version
file---and for fetch-lab.sh to consume that declaration rather than
hard-code a version.

The complete versioning and dependency-management policy is deliberately
not finalized in v0.2; see the Design Agenda.

# 16. Versioning and Dependency Management

Three independent version streams exist: **Lab**, **Template**, and each
**Experiment**. Version metadata committed in the repository is
authoritative. Git tags and GitHub Releases record published history;
GitHub repository variables are configuration, not version state.

## 16.1 Creative Lab versioning

Creative Lab uses Semantic Versioning. Major releases are breaking or
require deliberate migration; minor releases add backwards-compatible
functionality; patch releases contain backwards-compatible fixes or
small corrections.

Consumers may sync to newer minor and patch releases within their
current Lab major. Crossing a Lab major boundary is never automatic.

## 16.2 Experiment Template versioning

The Experiment Template is independently Semantic Versioned using the
same compatibility contract. Template and Lab versions are deliberately
independent. Existing experiments may update within their current
Template major; crossing a Template major is a deliberate migration.

## 16.3 Experiment release versioning and Cases

Experiment versions are **not Semantic Versioning**. They describe
progress through a sequence of **Cases** within the experiment.

The format is `whole.fraction`.

-   The whole-number component records the number of completed Cases.
-   The fractional component records released progress through the
    currently active Case.
-   A normal progress release increments the fractional component:
    `0.4 → 0.5` or `1.2 → 1.3`.
-   Completing a Case advances to the next whole version: `0.x → 1.0`,
    `1.x → 2.0`, and so on.
-   Starting a new Case does not itself change the version.

For example, a Newtonian Orbits experiment may develop its two-body Case
through `0.1`, `0.2`, and later complete it as `1.0`. Months later, Case
2 may begin as a three-body exploration while the repository remains at
`1.0`; its first progress release becomes `1.1`. Completing that Case
produces `2.0`.

A Case does not need to be planned in advance. An experiment may remain
complete at a whole version indefinitely and later be reopened with a
new Case.

## 16.4 Experiment manifest

The Template provides `experiment.toml`, which is authoritative for at
least:

``` toml
experiment_version = "0.1"
template_version = "1.0.0"
lab_version = "1.0.0"
```

The exact final schema remains an implementation detail. Template and
Lab tooling update the corresponding values when dependencies change.

## 16.5 Compatible dependency sync

Template and Experiment repositories provide a manually triggered **Sync
Lab** operation backed by locally runnable POSIX tooling.

Sync Lab:

1.  Reads the current Lab version from repository metadata.
2.  Finds the newest released Lab minor/patch version within the same
    major.
3.  Exits successfully if already current.
4.  Fetches the selected Lab release.
5.  Updates the manifest or dependency metadata.
6.  Validates the resulting repository.
7.  Commits the successful sync directly through trusted automation.

It must never cross a Lab major boundary. A major change is an explicit
upgrade/migration rather than a sync.

Where practical, GitHub workflows should call the same repository
scripts that a developer can run locally rather than maintaining
separate CI-only implementations.

## 16.6 Release provenance

Every published experiment release records:

-   experiment release version;
-   exact Template version;
-   exact Lab version.

The source tag, browser deployment, and packaged artifacts for a release
must correspond to the same exact source state.

## 16.7 Release targets

Where practical, an experiment release should produce:

-   a browser build for GitHub Pages;
-   Windows x64 distribution;
-   macOS distribution;
-   Linux distribution;
-   optionally a plain `.love` package.

The exact Linux packaging format and web compatibility tooling remain
implementation details.

GitHub Pages represents the latest **released** experiment rather than
arbitrary `main` state. Browser builds should support responsive sizing
and fullscreen where practical and should remain suitable for embedding
by an external site or CMS. External CMS integration itself is outside
Lab scope.

Lab and Template do not require experiment-style desktop or browser
targets. They publish only the artifacts needed by their own
update/fetch mechanisms.

# 17. Experiment Template Repository

creative-lab-experiment is a GitHub template repository used to create
new experiment repositories with the standard structure and bootstrap
tooling already in place.

creative-lab-experiment/\
├── .lab-version\
├── .template-version \[proposed provenance marker\]\
├── main.lua\
├── experiment.lua\
├── conf.lua\
├── README.md\
├── EXPERIMENT.md\
├── data/\
├── simulation/\
├── scripts/\
│ └── fetch-lab.sh\
├── vendor/\
│ └── .gitkeep\
└── .gitignore

The exact presence of data/ and simulation/ should remain flexible; they
are organizational defaults rather than mandatory architecture.

The template itself is maintained to production quality, but it
deliberately does not impose a test structure on generated experiments.

# 18. Experiment Documentation

Every experiment includes a single living `EXPERIMENT.md`. `README.md`
explains the repository and how to run it; `EXPERIMENT.md` explains the
exploration itself.

The document records the experiment's overall premise, question/domain,
setup, things worth exploring, and accumulated learning. It also
contains the history of **Cases** explored within the same experiment.

## 18.1 Case structure

A Case is a distinct scenario or question within the overarching
experiment. Each Case should use a consistent Markdown structure
containing, at minimum:

-   Case number/title;
-   objective or question;
-   setup/scenario;
-   status;
-   things to explore;
-   observations/findings;
-   completion version when complete.

The Case template should exist as maintainable repository content rather
than a large Markdown literal embedded in workflow YAML.

## 18.2 Start New Case

**Start New Case** is manually triggered. It verifies that the
experiment is at a whole-version boundary such as `1.0` or `2.0`,
determines the next Case number, appends a fresh Case template to
`EXPERIMENT.md`, and marks that Case active/current. It does **not**
change the experiment version.

If the fractional component is non-zero---for example `1.4`---the
operation fails with a useful message directing the developer to
complete the current Case first. Starting a new Case must never silently
complete an existing Case.

## 18.3 Complete Case

**Complete Case** marks the active Case complete in `EXPERIMENT.md`,
advances to the next whole experiment version, validates, and publishes
the completed Case using the normal experiment artifact matrix.

For example, completing Case 2 may move `1.4 → 2.0`.

## 18.4 Release Experiment

**Release Experiment** is the normal manually triggered progress
release. It increments the fractional component---for example
`1.2 → 1.3`---and publishes the experiment without declaring the active
Case complete.

# 19. Testing Policy

**Busted** is the standard/default Lua testing framework for Creative
Coding Lab.

## 19.1 Creative Lab

Lab is shared infrastructure and is treated as production-quality code.
Testable behaviour---particularly simulation-independent mathematics,
coordinate transforms, timing logic, state transitions, utilities, and
other deterministic infrastructure---should have Busted coverage.

Lab may use a minimal fixture/dummy experiment to exercise its
experiment-facing API and verify that LÖVE can load the Lab without
startup errors.

Code promoted from an experiment into Lab should be generalized,
documented, and accompanied by appropriate tests.

## 19.2 Experiments

Experiments do not have a mandatory testing requirement. They remain
exploratory.

The Template nevertheless provides a standard place and configuration
for Busted tests so an experiment can test simulation logic,
mathematics, state transitions, or utilities when useful. Experiment
validation detects whether tests exist: if they do, it runs Busted; if
not, the test stage skips successfully.

Rendering and interactive behaviour should not be distorted merely to
make everything unit-testable.

## 19.3 Experiment Template

Template is maintained as production-quality infrastructure. Its
validation must test the Template as a **real experiment bootstrap**,
not only with isolated test doubles.

A clean validation should:

1.  instantiate a fresh temporary experiment from the Template;
2.  avoid relying on a pre-vendored Lab;
3.  run the same user-facing Lab-fetch tooling distributed to
    experiments;
4.  fetch the Lab version declared by Template metadata;
5.  validate the resulting structure and configuration;
6.  run applicable Busted tests; and
7.  perform a LÖVE load/smoke check.

Test doubles remain useful for isolation, but critical
Lab--Template--Experiment integration paths should be exercised for
real.

# 20. Proposed Lab Structure

creative-lab/\
├── lab/\
│ ├── runtime.lua\
│ ├── camera.lua\
│ ├── input.lua\
│ ├── time.lua\
│ ├── drawing.lua\
│ ├── ui/\
│ └── debug/\
├── tests/\
├── README.md\
└── LICENSE

This structure is indicative rather than final. Modules should emerge
from real requirements rather than being created simply to fill an
architectural diagram.

# 21. Proposed Experiment Structure

experiment-name/\
├── .lab-version\
├── main.lua\
├── experiment.lua\
├── conf.lua\
├── README.md\
├── EXPERIMENT.md\
├── data/\
├── simulation/\
├── scripts/\
│ └── fetch-lab.sh\
├── vendor/\
│ └── lab/\
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

function experiment.instrument(ui)\
ui.slider("Gravity", ...)\
ui.slider("Time Scale", ...)\
ui.checkbox("Show Trails", ...)\
ui.button("Reset", ...)\
end

Instrumentation is part of the experiment's normal interactive
experience, not merely developer debugging.

## 22.2 Debugging tab

The Debugging tab is both supplied and defined by the Lab. It provides
common development and inspection facilities consistently across
experiments.

-   FPS and frame-time information.

-   Pause and resume.

-   Single-step simulation where supported by the runtime model.

-   Simulation-speed controls.

-   Coordinate and grid overlays.

-   Vector visualisation.

-   Debug drawing and bounds.

-   Object/selection inspection where meaningful.

-   Logging facilities.

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

State management remains intentionally unresolved in v0.4. The issue
includes ownership, lifetime, mutation, reset behaviour, data flow, and
the boundaries between Lab state and experiment state.

A representative experiment should be used to design this rather than
selecting a pattern in the abstract. A gravitational simulation with
simulation state, camera state, instrumentation values, selection state,
and Lab debug state remains a suitable candidate.

Lua's first-class functions make callback-based experiment behaviour
practical---for example, a Lab-owned button may receive an
experiment-defined callback---but the exact state ownership and mutation
model has not yet been chosen.

## 24.1 Questions for the next iteration

-   Which state is owned by Lab and which by the experiment?
-   How is simulation state represented and passed to simulation
    functions?
-   How do instrumentation controls read and mutate experiment state?
-   What does pause stop, and what continues to update while paused?
-   How are reset and restart semantics defined?
-   Should update functions mutate state, return state, or use a mixed
    approach?
-   How are camera, UI, debug, and selected-object state separated from
    simulation state?
-   What callback/binding model should Lab UI widgets expose?

This is the principal unresolved architectural topic after v0.4.

# 26. Experiment Evolution and Related Experiments

An experiment repository represents an exploration rather than a
permanently frozen scenario. Closely related developments should
normally remain within the same experiment when they are natural
extensions of the same underlying question.

For example, a Newtonian-gravity experiment may evolve from two bodies
to three or more bodies without requiring a separate repository if the
underlying exploration remains the same. The number of bodies may become
an interactive parameter where appropriate.

A separate experiment repository should be created when the new work
represents a meaningfully different question or exploration. Forking is
not the default mechanism. Small amounts of experiment-specific code may
be copied when beginning a related experiment.

If the same underlying code is repeatedly useful across experiments,
that is evidence that the concept may have earned promotion into
Creative Lab. Promotion should generalize, document, and test the
reusable implementation.

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

-   Optimize for experimentation.

-   Prefer obvious code.

-   Keep the Lab generic.

-   Keep experiments expressive in their own domain vocabulary.

-   Separate concept from presentation where useful.

-   Push side effects toward the edges where doing so improves clarity.

-   Do not impose a programming paradigm unnecessarily.

-   Earn abstractions through repeated use.

-   Treat promoted shared infrastructure as production-quality code.

-   Avoid premature engine development.

-   Design for 2D now without closing the door on 3D.

-   Keep development tools optional; the project should remain
    editor-agnostic.

The Lab provides the equipment. The experiment provides the question.

# 29. Development and Git Practices

## 29.1 Commits and branches

Commits should normally be small, focused development checkpoints.
Prefer incremental commits with a clear purpose over accumulating a
large body of work and separating it retrospectively. A development
commit may be incomplete or temporarily broken.

The hierarchy is: **commit = checkpoint; branch = ongoing work; pull
request = complete proposed change; `main` = trusted stable state;
release = published version.**

Lab and Template use stable `main` branches. Meaningful development
normally occurs on short-lived branches from `main`, followed by
validation and self-review through a pull request. There is no
long-lived `develop`, integration, or major-version branch unless a
concrete future need justifies one. Major release work still uses an
ordinary change branch: branches describe where work happens; versions
describe compatibility; releases describe published states.

Experiments are more relaxed. Branches and pull requests are supported
for substantial work but direct exploratory development on `main` is
permitted.

## 29.2 Issues and project tracking

GitHub Issues are used as work items where tracking is useful. Lab and
Template may share one Creative Coding Lab GitHub Project with
lightweight states such as Backlog, Ready, In Progress, and Done. Issues
can link to branches and pull requests. Tiny changes do not require an
Issue, and experiments remain outside the shared Project by default
unless they become substantial.

## 29.3 Coding principles

-   Prefer domain-specific, intention-revealing names. Names should
    communicate what a value or function represents, not merely its
    programming role.
-   Avoid single-letter identifiers, acronym-heavy names, and generic
    placeholders such as `data`, `item`, `value`, or `index` when a more
    meaningful domain name exists.
-   Prefer self-explanatory code over comments. Comments should be
    uncommon and explain non-obvious reasoning, constraints, deliberate
    compromises, or temporary context rather than narrating readable
    code.
-   Prefer guard clauses and early returns where they reduce nesting.
-   Keep functions and modules focused on coherent responsibilities
    without arbitrary line-count limits.
-   Keep nesting shallow where practical.
-   Avoid premature abstractions while promoting abstractions that have
    demonstrated genuine reuse.
-   Make state, mutation, side effects, and failures understandable.
-   Refactor continuously when emerging structure is clearly poor.
-   Follow established Lua/LÖVE conventions rather than importing
    conventions from other languages without a reason.

These are human readability conventions, not mechanically enforced size
or complexity metrics.

# 30. Pull Requests, CI, and Release Automation

Lab and Template pull requests must select exactly one release-intent
label: `release:none`, `release:patch`, `release:minor`, or
`release:major`. Absence of a label is not treated as `release:none`.

Release intent belongs to the pull request because the PR is the
complete unit of change. Development commits do not carry release
directives.

The standard PR template contains **Summary**, **Release Notes**, and
**Migration** sections. Patch/minor releases require meaningful release
notes. A `release:major` PR must contain meaningful migration guidance
rather than `N/A`.

PR Validation runs before merge and may include Lua syntax/static
checks, shell/configuration checks, version/manifest/structure
validation, Busted, LÖVE load/smoke checks, exactly-one-label
validation, release-note validation, and major-migration validation.

After a release-labelled Lab or Template PR is merged, Release
automation:

1.  reads the authoritative current version;
2.  applies the requested SemVer bump;
3.  extracts the reviewed Release Notes;
4.  extracts reviewed Migration content for a major release;
5.  updates authoritative version metadata;
6.  appends `CHANGELOG.md`;
7.  materializes major migration text into a dedicated migration
    document;
8.  commits those release changes atomically to `main` using trusted
    GitHub automation;
9.  tags that automated release commit;
10. packages from the tag; and
11. creates the GitHub Release.

The PR description is the human-authored source for release
documentation. Automation materializes it; it does not invent migration
advice.

The same Release workflow supports manual dispatch for recovery.
Recovery should be idempotent where practical. Trusted release/sync
automation may commit directly to protected `main` while ordinary human
changes remain PR-only.

# 31. Workflow Inventory

## 31.1 Lab

**PR Validation:** syntax/configuration, Busted, structure/version
metadata, release label/documentation, major migration requirements, and
a minimal fixture/dummy-experiment LÖVE load/smoke path.

**Release:** automatic after a merged release-labelled PR, with manual
recovery as a second entry point into the same workflow.

## 31.2 Template

**PR Validation:** the relevant Lab-style checks plus a real clean
experiment bootstrap. Validation creates a temporary experiment, relies
on no pre-vendored Lab, invokes the same user-facing Lab-fetch tooling
distributed to experiments, fetches the Lab declared by Template
metadata, and tests/builds/smoke-loads the resulting environment.

**Release:** the same PR-label-driven SemVer release model as Lab.

**Sync Lab:** manually triggered. Uses the shared POSIX sync tooling to
move Template to the newest compatible Lab minor/patch within its
current Lab major, validates, and commits the successful sync through
trusted automation.

Experiment workflows stored inside Template are Template content rather
than additional Template-release workflows.

## 31.3 Experiment

**Validation:** validates `experiment.toml`, structure/configuration,
the declared Lab dependency/fetch path, optional Busted tests, and
LÖVE/package/load behaviour.

**Sync Lab:** manually triggered compatible same-major Lab sync,
followed by validation and a trusted automated commit.

**Start New Case:** manually triggered. Only succeeds at a whole-version
boundary and appends the next Case template without changing the
version.

**Release Experiment:** manually triggered normal progress release. It
increments the fractional experiment version, validates, commits/tags,
builds the release artifact matrix, deploys the web build to GitHub
Pages, and creates the GitHub Release.

**Complete Case:** manually triggered completion release. It marks the
active Case complete, advances to the next whole experiment version, and
publishes the same artifact matrix.

# 32. Initial Development Strategy

1.  Design state ownership and data flow against a representative
    experiment.
2.  Define the initial branching strategy and commit/workflow
    conventions.
3.  Create the Creative Lab repository and establish its Semantic
    Versioning and release metadata.
4.  Implement the minimum viable 2D runtime and UI shell.
5.  Create the Experiment Template, TOML manifest, dependency-update
    scripts, and inherited GitHub Actions.
6.  Create the first real experiment.
7.  Establish the experiment release and GitHub Pages deployment path.
8.  Observe friction, missing abstractions, and unnecessary
    abstractions.
9.  Promote genuinely reusable concepts into the Lab with documentation
    and tests.
10. Create another experiment and repeat.

# 33. Remaining Open Questions

The principal unresolved architectural topic after v0.4 is **State
Management & Data Flow**: ownership, mutation, lifetime, lifecycle,
pause/reset semantics, UI interaction, callback/binding behaviour, and
simulation boundaries.

Other implementation/API details remain deliberately open: exact public
Lab API naming, direct LÖVE access versus wrappers, lifecycle callbacks,
initial 2D camera API, UI widget/binding API, debugging rollout,
browser-build tooling, Linux distribution format, exact version
filenames/script names, and eventual 3D architecture.

Branching strategy, release intent, CI/release automation, testing
framework, compatible Lab syncing, and the experiment Case lifecycle are
no longer principal open architectural questions in v0.4.

# 34. Non-Goals --- For Now

-   Building a general-purpose game engine.

-   Designing a complete 3D renderer before a real experiment needs one.

-   Creating a universal plugin/package ecosystem.

-   Enforcing production-quality structure or tests inside exploratory
    experiments.

-   Abstracting every LÖVE API pre-emptively.

-   Solving hypothetical future requirements without evidence from
    experiments.

# 35. Definition of Success

Creative Coding Lab succeeds when an idea such as 'what happens if I
simulate several bodies under Newtonian gravity?' quickly becomes work
on gravity, integration, visualisation, and experimentation rather than
an evening spent rebuilding window setup, input handling, camera
controls, sliders, and debugging infrastructure.

A successful experiment should also leave behind a comprehensible record
of what was being explored and what was learned.

# 36. Current Architectural Summary

  ---------------------------------------------------------------------
  **Area**                           **v0.4 direction**
  ---------------------------------- ----------------------------------
  Runtime                            Independent Creative Lab
                                     repository; reusable shared
                                     infrastructure.

  Experiments                        Independent public LÖVE
                                     repositories with a structured
                                     Case lifecycle.

  Template                           Independent SemVer'd GitHub
                                     template; production-quality
                                     infrastructure and experiment
                                     tooling.

  Dependencies                       Lab fetched into `vendor/lab`;
                                     committed metadata is
                                     authoritative; same-major Sync Lab
                                     supported.

  Versioning                         Lab/Template use independent
                                     SemVer; experiments use
                                     completed-Case/progress numbering.

  Documentation                      `README.md` plus living
                                     `EXPERIMENT.md` with Case history.

  Testing                            Busted standard; expected for
                                     Lab/Template; optional but
                                     supported for experiments.

  Git                                Stable `main` for Lab/Template
                                     with short-lived branches and PR
                                     validation; experiments more
                                     relaxed.

  Releases                           PR-label-driven Lab/Template
                                     releases; manual experiment
                                     progress and Case-completion
                                     releases.

  Distribution                       Experiment releases target
                                     web/GitHub Pages, Windows, macOS,
                                     Linux, and optionally `.love`.

  UI                                 Lab-owned sidebar with
                                     experiment-defined Instrumentation
                                     and Lab-defined Debugging.

  Rendering                          2D first; 3D only when justified
                                     by a real experiment.

  State                              Principal unresolved architectural
                                     topic for v0.5.

  Development tools                  Editor-agnostic; VS Code workspace
                                     is optional convenience.
  ---------------------------------------------------------------------

# 37. Design Agenda for v0.5

The next focused design iteration should address **State Management &
Data Flow** using a representative experiment rather than selecting a
pattern in the abstract.

The discussion should resolve state ownership, mutation, lifecycle,
pause/reset semantics, instrumentation interaction, callback/binding
behaviour, and separation of simulation, camera, UI, debugging, and
selection state.

# 38. Document Status

Version 0.4 is a consolidation checkpoint. It incorporates the resolved
versioning model, experiment Case lifecycle, Busted testing policy, real
Template bootstrap validation, Git/branching practices, GitHub
Issue/Project usage, PR release metadata, changelog and migration
generation, per-repository workflow inventories, compatible Lab syncing,
release artifact direction, and general development/coding principles.

State Management & Data Flow remains intentionally unresolved and is the
principal design topic for v0.5.

This remains a living document. Its purpose is to record current intent
and decisions while allowing implementation experience and real
experiments to challenge and improve the architecture.
