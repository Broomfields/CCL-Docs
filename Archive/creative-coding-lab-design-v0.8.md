---
title: Creative Coding Lab
---

# Creative Coding Lab

Design document v0.8

| Field | Value |
|---|---|
| Status | Consolidated design draft for review |
| Version | 0.8 |
| Initial platform | Lua and LÖVE |
| Project type | Personal creative-coding platform and independent experiments |

This document brings the existing design into one place. It restores requirements omitted from earlier revisions, groups related topics, and simplifies the wording. It does not introduce new architectural choices. Unresolved behaviour is identified where it affects a topic and collected in Part F.

Examples show intended relationships. File layouts, API names, and code examples are illustrative unless a section explicitly defines them as a requirement. The document records design intent, not a claim that the features have been implemented.

## Contents

- [Part A. Project purpose and scope](#part-a-project-purpose-and-scope)
- [Part B. Repositories and project delivery](#part-b-repositories-and-project-delivery)
- [Part C. Runtime architecture](#part-c-runtime-architecture)
- [Part D. Interface and interaction](#part-d-interface-and-interaction)
- [Part E. Run data and observations](#part-e-run-data-and-observations)
- [Part F. Implementation, open questions, and roadmap](#part-f-implementation-open-questions-and-roadmap)

# Part A. Project purpose and scope

## 1. Purpose

Creative Coding Lab is a lightweight environment for building visual, interactive programming experiments. It supports exploration of mathematics, simulation, procedural generation, graphics, and other computational ideas through code.

The project has three parts:

```text
Creative Coding Lab project
├── Lab: shared runtime and reusable infrastructure
├── Template: starting structure and development tooling
└── Experiments: independent applications and portfolio projects
```

The Lab provides equipment such as timing, cameras, rendering, input, controls, and debugging. Each experiment supplies the question, domain model, and simulation rules.

### Goals

- Move quickly from an idea to a running visual experiment.
- Learn by implementing concepts, changing parameters, and observing results.
- Reuse infrastructure without turning the project into engine development for its own sake.
- Let each experiment stand as an independent public repository and portfolio item.
- Maintain shared infrastructure carefully while keeping exploratory code flexible.
- Record the premise, setup, intended learning, and findings of each experiment.
- Let real experiments guide the platform's growth.

### Design decisions

Experiments normally begin with a question, such as how bodies interact under gravity or how boids produce flocking. Reproducing a known result is only part of the work. Implementation and observation are also learning activities.

Success means spending time on the question rather than rebuilding window setup, camera controls, sliders, timing, picking, and debugging. An experiment should also leave a clear record of what was explored and learned.

## 2. Principles and boundaries

The architecture serves experimentation. Prefer obvious code, explicit ownership, and the smallest useful shared API.

| Principle | Application |
|---|---|
| Keep the Lab generic | It understands cameras, timing, geometry, UI, and inspection, not planets, boids, or Fourier transforms. |
| Earn abstractions | Begin domain-specific ideas inside experiments. Promote them after repeated use shows they are general. |
| Keep dependencies clear | Pass the state or service a function needs. Separate calculations from side effects where useful. |
| Use practical programming styles | Functional, data-oriented, and object-oriented approaches are tools, not architectural goals. |
| Treat shared code carefully | Promoted code is generalised, documented, and tested before others rely on it. |
| Keep experiments flexible | Production-style structure and tests are not prerequisites for exploring an idea. |
| Build 2D first | Avoid assumptions that make future 3D unnecessarily difficult, without designing a 3D engine now. |
| Keep tools optional | Repositories remain editor-agnostic. |

### Outside the current scope

- A general-purpose game engine or universal plugin/package ecosystem.
- A full entity component system, entity factories, pooling, or manager hierarchy without demonstrated need.
- A physics engine or generic collision-response system before reuse justifies one.
- Spatial trees, BVHs, quadtrees, or octrees before experiments need them.
- Mandatory Lab base classes for experiment objects.
- Wrapping every LÖVE API in advance.
- Elaborate lifecycle hooks or formal read-only/writable context hierarchies.
- Automatic deep-copy reset, arbitrary mid-run save-state serialisation, or undo architecture.
- Persistent browser run storage or server-side run storage.
- Built-in charting in the first working versions.
- Speculative 3D infrastructure or other hypothetical requirements.

## 3. Technology and terminology

| Technology | Role |
|---|---|
| Lua | Lab and experiment programming language. |
| LÖVE | Initial application, graphics, input, and runtime framework. |
| POSIX shell | Bootstrap, dependency-fetching, and development scripts. |
| Markdown | Project documentation and experiment notebooks. |
| TOML | Experiment manifest and committed version/provenance metadata. |
| Busted | Standard Lua testing framework. |
| Git | Source control and release tags. |
| GitHub | Hosting, Issues, Projects, Actions, releases, distribution, and portfolio presentation. |

VS Code is the intended local editor. A multi-root workspace is an optional convenience, not a dependency.

| Term | Meaning |
|---|---|
| Lab | Shared runtime and reusable infrastructure. |
| Template | Independently maintained GitHub template used to create experiments. |
| Experiment | Independent application and repository exploring a domain or question. |
| Case | A distinct scenario or question within an experiment's development history. |
| Run | One execution from a defined set of initial conditions. |
| Session | One launch of an experiment application, which may contain multiple runs. |
| Release | A published source state and its corresponding artifacts. |

Completing a run is a runtime event. Completing a Case is a development and release operation.

# Part B. Repositories and project delivery

## 4. Repository organisation

The local workspace groups independent repositories. The parent folder is not itself a Git repository.

```text
creative-coding/                    [local workspace]
├── creative-lab/                   [Git repository]
├── creative-lab-experiment/        [GitHub template repository]
├── experiments/
│   ├── orbital-mechanics/          [Git repository]
│   ├── boids/                      [Git repository]
│   └── fourier-transform/          [Git repository]
└── creative-coding.code-workspace  [optional local convenience]
```

The dependency and creation relationships are separate:

```text
Template ── creates starting files ──> Experiment repository
Lab release ── fetched into ─────────> Experiment's vendor/lab
Experiment + fetched Lab ────────────> Independent application
```

Each experiment runs with its own fetched Lab dependency. There is no persistent Lab process that switches between experiments.

### Design decisions

Independent repositories let experiments stand alone. The shared parent folder provides a convenient local view without repository coupling.

Git submodules are intentionally avoided. Committing a child change and then updating a parent pointer adds friction. Bootstrap tooling retrieves dependencies instead, without requiring a particular sibling-directory layout.

### Related experiments

Keep closely related developments in one experiment when they extend the same question. A Newtonian-gravity experiment can grow from two bodies to three or more, with body count becoming an interactive parameter if useful.

Create a separate repository when the question changes meaningfully. Forking is not the default. Small amounts of experiment-specific code may be copied when starting related work. Repeated reuse is evidence that code may belong in the Lab.

## 5. Template and source layout

The Template supplies the standard structure, bootstrap tooling, experiment workflows, and documentation. It is production-quality infrastructure, but its generated experiments remain flexible.

An experiment's intended structure combines its application files with the fetched dependency:

```text
experiment-name/
├── experiment.toml
├── main.lua
├── conf.lua
├── experiment.lua
├── simulation.lua
├── controls.lua
├── definitions.lua       [optional]
├── shaders/              [may be empty]
├── README.md
├── EXPERIMENT.md
├── scripts/
│   └── fetch-lab.sh
├── vendor/
│   └── lab/              [fetched dependency, ignored by Git]
└── .gitignore
```

| File | Responsibility |
|---|---|
| `main.lua` | Experiment-owned application entry point. |
| `conf.lua` | Application configuration. |
| `experiment.lua` | Domain state, lifecycle coordination, and callbacks. |
| `simulation.lua` | Mathematical/model functions. |
| `controls.lua` | Experiment UI declarations. |
| `definitions.lua` | Optional constructors and plain domain-data definitions. |
| `shaders/` | Experiment-owned shader code. |

Additional data or simulation folders remain organisational choices. The layout is a starting point, not a requirement to create unused modules. The Template can keep an empty vendor directory with `.gitkeep` before fetching the Lab.

The Lab's indicative internal structure is:

```text
creative-lab/
├── lab/
│   ├── runtime.lua
│   ├── camera.lua
│   ├── input.lua
│   ├── time.lua
│   ├── drawing.lua
│   ├── ui/
│   └── debug/
├── tests/
├── README.md
└── LICENSE
```

### Design decisions

Plain Lua tables and small modules are the default. Domain definitions do not introduce a framework type system or require class hierarchies from other languages. Lab modules should emerge from real needs rather than from filling every box in a diagram.

Experiment workflows distributed by the Template are Template content. They are separate from the Template repository's own validation and release workflows.

## 6. Dependencies and versioning

### Version authority

Lab, Template, and each experiment have independent version streams. Committed metadata is authoritative. Tags and GitHub Releases record published history. GitHub repository variables hold configuration, not version state.

The Template seeds `experiment.toml`, which the experiment owns after creation:

```toml
experiment_version = "0.1"
template_version = "1.0.0"
lab_version = "1.0.0"
```

The manifest records at least these values. The final schema and supporting metadata filenames remain implementation details. Update tooling changes the corresponding values when dependencies change. The Lab only needs its own version metadata and does not track its consumers' versions.

### Lab and Template versions

Both use Semantic Versioning independently:

| Change | Meaning |
|---|---|
| Major | Breaking change or deliberate migration. |
| Minor | Backwards-compatible functionality. |
| Patch | Backwards-compatible fixes or small corrections. |

Consumers may update within their current major version. Crossing a Lab or Template major boundary is a deliberate migration and is never automatic. Same-major Template updates must remain compatible with existing experiments. Changes to Template scripts, workflows, or documentation need not require a Lab release, and vice versa.

Compatibility intent is chosen by the programmer. Automation applies the chosen version change rather than deciding its meaning.

### Fetching and syncing the Lab

`fetch-lab.sh` is POSIX shell tooling that reads the declared Lab version and retrieves that release into `vendor/lab`. The dependency is not committed to the experiment repository. It is included in the runnable tree and packaged application.

Template and experiment repositories provide a manually triggered **Sync Lab** operation:

```text
Read committed Lab version
          ↓
Find newest released minor/patch in the same major
          ↓
Already current? ── yes ──> Exit successfully
          │ no
          ↓
Fetch release → Update metadata → Validate
                                      ↓ success
                            Commit through trusted automation
```

A major upgrade is not a sync. GitHub workflows should call the same locally runnable scripts where practical.

Template-provided tooling is intended to compare recorded Template and Lab versions with available compatible releases. Compatible Template updates are part of this direction, but their file-update and conflict-handling mechanism is not defined. The clean v0.4 copy also proposed selecting a specific compatible version explicitly. That option is retained as an unresolved tooling detail, not an extra established workflow.

### Design decisions

Fetching an exact declared version makes an experiment runnable without a particular workspace layout. Independent versions separate runtime compatibility from inherited tooling changes. Manual initiation gives the developer control over when compatible updates are applied.

## 7. Experiment notebooks, Cases, and versions

Every experiment has a living `EXPERIMENT.md`. `README.md` explains the repository and how to run it. The notebook explains the exploration.

```text
EXPERIMENT.md
├── Premise and domain
├── Overall question
├── Setup
├── Things to explore
├── Accumulated learning
└── Case history
    └── Each Case
        ├── Number and title
        ├── Objective or question
        ├── Setup or scenario
        ├── Status
        ├── Things to explore
        ├── Observations and findings
        └── Completion version, when complete
```

Keep the Case template as maintainable repository content rather than a large Markdown string in workflow YAML.

### Experiment versions

Experiment releases use completed-Case/progress numbering, not Semantic Versioning. The first component counts completed Cases. The second records released progress through the active Case.

```text
Case 1 progress:  0.1 → 0.2 → 0.3
Complete Case 1:  0.x → 1.0
Start Case 2:     remains 1.0
Case 2 progress:  1.1 → 1.2 → 1.3
Complete Case 2:  1.x → 2.0
```

A Case need not be planned in advance. An experiment may remain at a completed whole version indefinitely and later resume with another Case.

### Case operations

| Operation | Behaviour |
|---|---|
| Start New Case | Manual. Verify a whole-version boundary such as `1.0`, determine the next Case number, append its template, and mark it active/current. Do not change the version. |
| Release Experiment | Manual progress release. Increment the second component, validate, commit/tag, and publish without declaring the Case complete. |
| Complete Case | Manual completion release. Mark the active Case complete in the notebook, advance to the next whole version, validate, and publish the normal artifact matrix. |

Start New Case fails with a useful instruction to complete the current Case when the second component is non-zero. It must never silently complete an existing Case. For example, completing Case 2 can advance `1.4` to `2.0`.

Repeated Start New Case calls at the same unchanged whole version, initial Case creation, and progress-number handling beyond the examples remain details to clarify. This edition does not add new workflow rules to resolve them.

### Design decisions

Cases record extensions of one exploration without forcing a new repository for each scenario. Versions communicate progress and completion rather than API compatibility.

## 8. Git, tracking, and coding practices

### Commits and branches

| Unit | Meaning |
|---|---|
| Commit | Focused development checkpoint. |
| Branch | Ongoing work. |
| Pull request | Complete proposed change. |
| `main` | Trusted stable state for Lab and Template. |
| Release | Published version. |

Prefer small, focused commits during development rather than dividing a large change retrospectively. Intermediate commits may be incomplete or temporarily broken.

Lab and Template normally use short-lived branches from `main`, followed by validation and self-review through a pull request. There is no standing `develop`, integration, or major-version branch without a concrete need. Major releases still use ordinary change branches.

Experiments may develop directly on `main`. Branches and pull requests remain useful for substantial changes.

### Issues and project tracking

Use GitHub Issues when tracking is useful. Lab and Template may share one Creative Coding Lab GitHub Project with lightweight states such as Backlog, Ready, In Progress, and Done. Issues may link to branches and pull requests.

Tiny changes do not require an Issue. Experiments remain outside the shared Project by default unless they become substantial.

### Coding practices

- Use domain-specific names that explain what a value or function means.
- Avoid single-letter names, unexplained acronyms, and placeholders such as `data`, `item`, `value`, or `index` when a clearer name exists.
- Prefer readable structure and naming over explanatory comments. Use comments for non-obvious reasoning, constraints, compromises, or temporary context.
- Use guard clauses and early returns where they reduce nesting.
- Keep functions and modules focused, with shallow nesting where practical.
- Make state changes, dependencies, side effects, and failures understandable.
- Refactor when emerging structure is clearly poor. Avoid premature abstractions and promote proven reusable code.
- Follow established Lua and LÖVE conventions rather than importing another language's conventions without a reason.

These are readability practices, not mechanical line-count or complexity limits.

### Design decisions

Validation belongs at the integration boundary. Development checkpoints can remain flexible while Lab and Template `main` stay stable. Release intent belongs to the complete change represented by a pull request.

## 9. Testing and validation

Busted is the standard Lua test framework.

| Repository | Policy |
|---|---|
| Lab | Production-quality shared infrastructure. Test deterministic behaviour such as mathematics, transforms, timing, state transitions, camera calculations, and utilities. |
| Template | Production-quality starting point. Validate a real experiment bootstrap as well as its own tooling. |
| Experiment | Tests are optional. Provide a standard place and Busted configuration; run tests when present and skip successfully when absent. |

Code promoted into the Lab is generalised, documented, and tested. Rendering and interactive behaviour should not be distorted merely to make everything unit-testable.

Lab may use a minimal fixture experiment to exercise its public behaviour and verify that LÖVE loads it without startup errors.

### Template bootstrap validation

```text
Create fresh temporary experiment from Template
                      ↓
Start without a pre-vendored Lab
                      ↓
Run the distributed user-facing fetch tooling
                      ↓
Fetch the Lab version declared by Template metadata
                      ↓
Validate structure and configuration
                      ↓
Run applicable Busted tests and LÖVE load/smoke checks
```

### Design decisions

Test doubles are useful for isolation. They do not replace checking the real Lab, Template, and experiment integration path. Optional experiment tests support useful checking without adding mandatory ceremony.

## 10. Pull requests and release automation

### Release intent

Every Lab and Template pull request selects exactly one label:

```text
release:none | release:patch | release:minor | release:major
```

No label is not equivalent to `release:none`. Development commits do not carry release directives.

The PR template contains Summary, Release Notes, and Migration sections. Patch and minor releases require meaningful release notes. Major releases require meaningful migration guidance, not `N/A`.

PR validation runs before merge. Checks may include Lua syntax/static checks, shell/configuration checks, structure and version metadata, Busted, LÖVE load/smoke checks, exactly-one-label validation, release notes, and required migration guidance.

### Lab and Template release sequence

After a PR requesting a version bump is merged, release automation performs this sequence:

1. Read the authoritative current version.
2. Apply the requested Semantic Versioning bump.
3. Extract the reviewed release notes and, for a major release, migration guidance.
4. Update authoritative version metadata.
5. Append `CHANGELOG.md` and write major migration guidance to a dedicated document.
6. Commit the release changes atomically to `main` through trusted GitHub automation.
7. Tag that automated release commit.
8. Package from the tag.
9. Create the GitHub Release.

The same release workflow supports manual dispatch for recovery. Recovery should be idempotent where practical. Trusted release/sync automation may commit directly to protected `main`; ordinary human Lab and Template changes remain PR-only.

### Design decisions

The PR description is the human-authored source for release documentation. Automation transfers reviewed content into published files and releases. It does not invent migration advice. Tagging the release commit keeps metadata and packaged source aligned.

## 11. Workflow inventory

This table identifies each repository's workflows. Detailed rules are defined in the preceding sections of this document.

| Repository | Workflow | Responsibility |
|---|---|---|
| Lab | PR Validation | Syntax/configuration, Busted, structure/version metadata, release label/documentation, major migration requirements, and minimal fixture LÖVE load/smoke path. |
| Lab | Release | Automatic after a merged PR requesting a release; manual recovery enters the same workflow. |
| Template | PR Validation | Relevant Lab-style checks plus the clean bootstrap procedure in section 9, including applicable test, build, and smoke/load checks on the resulting environment. |
| Template | Release | Same PR-label-driven release model as Lab. |
| Template | Sync Lab | Manual compatible Lab update, validation, and trusted automated commit. |
| Experiment | Validation | Manifest, structure/configuration, declared Lab fetch path, optional Busted tests, and LÖVE/package/load behaviour. |
| Experiment | Sync Lab | Manual same-major Lab update, metadata update, validation, and trusted automated commit. |
| Experiment | Start New Case | Manual Case creation at a whole-version boundary without a version bump. |
| Experiment | Release Experiment | Manual progress bump, validation, commit/tag, artifact builds, Pages deployment, and GitHub Release. |
| Experiment | Complete Case | Manual completion update and whole-version bump, followed by the same release artifact process. |

## 12. Packaging, distribution, and provenance

Every published experiment release records its experiment version and exact Template and Lab versions. Release automation reads these from committed metadata and includes them in release metadata or notes.

```text
One exact released source state
├── Source tag
├── Browser build and GitHub Pages deployment
├── Windows x64 distribution
├── macOS distribution
├── Linux distribution
└── Optional .love package
```

Where practical, experiment releases should produce this matrix. Packaged experiments include their required Lab dependency in `vendor/lab`. Exact packaging mechanics, Linux format, and browser compatibility tooling remain implementation details.

GitHub Pages represents the latest released experiment, not arbitrary `main` state. Browser builds should support responsive sizing and fullscreen where practical. They should be suitable for direct use and embedding through an iframe where browser and security constraints permit.

External websites or CMSs may store the hosted URL and embed or link to it. That integration is outside the Lab's scope.

Lab and Template publish only artifacts needed by their own fetch/update mechanisms. They do not need experiment-style desktop or browser distributions.

### Design decisions

The tag, deployed browser build, and packaged artifacts must correspond to the same exact source state. A release should identify what produced it, independently of later development.

# Part C. Runtime architecture

## 13. Responsibilities and data flow

The experiment owns domain meaning and state. The Lab provides generic mechanisms around it.

```text
Experiment coordinator
├── Domain state and definitions
├── Simulation/model functions
├── Control declarations and callbacks
└── Capabilities exposed to Lab
                  │
                  ▼
Lab services
├── Application lifecycle and simulation timing
├── Entity registrations, spatial queries, and contact detection
├── Rendering, camera, input, and audio
├── UI, debugging, and inspection
└── Seeded randomness, run records, and measurements
```

| Concern | Owner |
|---|---|
| LÖVE lifecycle integration and scheduling | Lab |
| Simulation clock, speed, and control orchestration | Lab |
| Camera, viewport, raw input, and generic input bindings | Lab |
| Graphics/audio resources and generic presentation | Lab |
| UI shell, widgets, layout, and debugging facilities | Lab |
| Entity registration records and generic queries | Lab |
| Domain objects, collections, terminology, and selection | Experiment |
| Mathematical/simulation rules and physical response | Experiment |
| Parameter meaning and control callbacks | Experiment |
| Initial conditions and reset behaviour | Experiment |
| Generic concepts proven reusable | May be promoted into Lab |

The Lab does not own the collection of experiments, infer what a selected domain object means, or take ownership of experiment state.

## 14. State, mutation, and model code

Use plain Lua state tables. Coordinators may receive complete state, while lower-level functions receive narrower values or sub-state where that clarifies their dependencies.

```text
Experiment-owned state
├── Domain objects, such as bodies
├── Parameters, such as gravity
└── Domain selection, such as selected body
          │
          ▼
Model calculations → Results → Explicit updates to owned state
```

Pure calculations should return values where natural. A function may also mutate state explicitly passed to it. Neither Lab nor experiment code should secretly mutate unrelated state owned by the other side.

Simulation calculations should ideally depend on neither LÖVE nor the Lab. A function that needs a Lab service can receive that service or the Lab; a function that needs neither should know about neither.

### Design decisions

Purity is a tool, not a rule. A thin mutating layer can apply calculated results without requiring immutable state or framework context types. Physics concepts such as mass, momentum, impulses, and rigid-body response stay in experiment model code until repeated reuse earns a shared abstraction.

## 15. Application and run lifecycle

The experiment owns `main.lua`. A possible entry shape is:

```lua
local Lab = require("vendor.lab.runtime")
local Experiment = require("experiment")

Lab.run(Experiment)
```

### Initialisation

Initialisation happens once when the application loads, before Play. It may create state and domain objects, register capabilities, define instrumentation, and provide update/reset behaviour.

```text
Application load → Experiment initialisation → Ready
                                              ├── Initial scene visible
                                              ├── Simulation time is zero
                                              └── Simulation is not advancing
```

Play does not initialise the experiment. No separate `onPlay()` callback is required unless a real experiment demonstrates a need.

### Established control behaviour

| Control or state | Existing design intent |
|---|---|
| Play | Begin or continue simulation advancement. Requires update capability. |
| Pause | Stop simulation advancement while rendering, camera, UI, and inspection may continue. |
| Step | Perform exactly one fixed simulation update while Ready/Paused. Disabled while running. Requires update capability. |
| Restart | Reproduce the current run's initial conditions. Disabled until simulation has advanced. Restoration requires experiment reset capability. |
| Completed | Optional experiment-defined natural completion stops simulation advancement without closing the application. |

When Play starts a new run, the Lab snapshots the seed and recorded parameters, creates the run record, and locks those parameters. The v0.7 completion direction also allows stable completion reasons/results in run metadata. The completion callback, check order, and post-completion control behaviour are not yet defined.

### Reset mechanism

The Lab owns the control and clock. The experiment supplies the callback that restores its initial conditions.

```text
Reset request
      ↓
Lab stops simulation advancement
      ↓
Experiment reset callback restores initial conditions
      ↓
Lab sets simulation time to zero
      ↓
Ready
```

The Lab does not deep-copy arbitrary experiment state. The experiment chooses its restoration method. Mutating existing registered objects is useful because registrations remain valid.

### Decisions still open

The existing design uses Restart for the current run and Reset for the restoration mechanism, but does not fully define their relationship. It also allows Step while Ready without stating whether that first Step creates and locks a run record. These gaps are preserved for review, not resolved by this diagram or table.

The action for leaving a current/completed run and returning to editable configuration remains open. So do preview rebuilding after configuration changes, record identity on Restart, measurement handling, RNG/accumulator reset details, and registration/contact cleanup during reconstruction.

### Design decisions

The Lab schedules simulation work, but only the experiment knows how to restore its domain. Nested tables, shared references, functions, and caches make automatic copying an unreliable substitute for an experiment-defined reset.

## 16. Simulation timing

The default is a fixed simulation timestep, independent of render frequency.

```text
LÖVE variable real frame time
              ↓
Lab time scaling and accumulator
              ↓
Zero or more fixed simulation steps → Experiment update

LÖVE render frames → Read latest experiment state → Lab rendering
```

For example, `1/60` second could be the fixed step. That number is illustrative, not a chosen default. Experiments do not normally receive raw variable frame time as their simulation timestep.

Simulation speed scales how quickly real time becomes simulation work. At 10 times speed, the Lab performs more fixed steps per real second rather than making each physics step ten times larger.

Lab owns real time, simulation time, the accumulator, and scheduling. Experiment update defines what advancing the domain means. Simulation frequency, rendering frequency, and entity lifetime are separate.

### Design decisions

Fixed steps support predictable numerical experiments. Rendering can remain interactive while simulation is paused. Lab should prevent unbounded catch-up after long frames or stalls; the cap and overflow policy remain implementation choices. Render interpolation is deferred until experiments demonstrate a need.

## 17. Entity registration and lifetime

An experiment registers objects that need generic services such as rendering, picking, or inspection. Registration does not transfer ownership or require a Lab base class.

```text
Experiment-owned object
          │ opaque reference
          ▼
Lab-owned registration
├── Spatial capability
├── Render capability
├── Inspection capability
└── Further capabilities earned through use
```

Callbacks/accessors read the live object. For example, a circle's spatial capability can return its current position and radius, while its render capability describes its appearance. The experiment does not duplicate its state into the Lab or notify it after every change.

```text
Create object → Store in experiment state → Register with Lab
                                                ↓
                              Mutate object; Lab reads live capabilities
                                                ↓
                            Unregister when services are no longer needed
                                                ↓
                                Remove object under experiment rules
```

Unregistering stops Lab services. It does not destroy the object. Application teardown ends registration lifetime. No persistent process is needed to unload one experiment and load another.

The earlier guarantee of clearing registrations on whole-experiment reinitialisation was replaced by the separate-application explanation. Registration cleanup during in-application reconstruction remains an open detail rather than an assumed new guarantee.

### Design decisions

Domain lifetime and registration lifetime are independent. Capabilities let the Lab provide services without learning domain meanings. This model resembles components but does not declare an entity component system.

## 18. Spatial queries, selection, and contact

### Coordinate and selection flow

```text
Screen input → Lab camera transform → World-space query
                                           ↓
                                 Generic spatial results
                                           ↓
                          Experiment interprets objects and selection
```

World interaction should not require experiment code to work in screen pixels. The experiment owns domain selection, such as a selected body. Lab may inspect a returned/selected object but does not decide that it means a selected planet.

The preferred spatial capability exposes generic geometry. Lab can perform point/circle or other shape intersections from current positions and dimensions. Custom hit-test callbacks may remain an escape hatch.

Queries should support multiple hits even when a convenience pick operation returns one. Results may include the object, world position, and distance. Spatial identity must not mean first registered wins.

In 2D, render ordering may help resolve overlapping picks. In future 3D, ray distance will normally be more useful. Render order is separate from world-space depth.

### Contact detection

Lab detects overlap using generic registered spatial geometry. It tracks contact per pair:

```text
Separate → First overlap → Continuing overlap → Separate again
               ENTER             STAY               EXIT
```

The normal contact callback is an enter notification once per contact episode. Stay/exit callbacks may be supported when experiments need them. Callbacks attach to registrations/capabilities so objects may react differently.

Whether both registrations receive an event and the exact contact information remain open. Contact points or normals may be included where the geometry supports them. Contact layers are separate from render order; z-index must not determine which objects can contact.

Lab does not prevent interpenetration, apply impulses, calculate momentum, or bounce objects. The experiment may allow overlap or implement its own response.

### Design decisions

Generic geometry supports picking and contact without embedding domain rules. It also leaves room for future acceleration and ray-based 3D queries. Physical response remains experiment-owned until real reuse justifies promotion.

## 19. Rendering, cameras, and shaders

### Rendering contract

```text
Experiment
├── World-space geometry or procedural definitions
├── Appearance intent
└── Shader code and experiment parameters
                 ↓
Lab
├── Camera, projection, and viewport
├── Graphics resources and shader application
└── Final screen-space rendering
```

Positions and sizes are expressed in world units. The experiment does not convert an object's radius to pixels; the Lab camera/projection determines its screen representation.

| Path | Intended use |
|---|---|
| Retained rendering | Register a persistent object once with a render capability. Lab reads its current description each render frame. |
| Immediate/procedural drawing | Submit transient geometry for curves, graphs, fields, trails, mathematical visualisations, or debug drawing. |

Not every rendered item needs an entity. Not every registration needs a render capability. Spatial and visual representations are separate: complex artwork may use a simple pick shape, a visible object may be unselectable, and an inspectable object may have no normal visible form.

Initial primitives should include straightforward circles, rectangles, lines, and polygons. Experiments may build more complex geometry from generic Lab-understood representations. Exact primitive shapes and API names remain open.

### Camera and 3D direction

The initial 2D camera has fixed sensible defaults with pan and zoom. Experiments may override initial settings and explicitly request framing of registered spatial contents. Automatic framing is not the default.

Future 3D is expected when a concrete experiment needs it. It may require substantial rendering/camera redesign. Preserve experiment-facing APIs and minimise migration where reasonably possible, without pre-designing the 3D system.

### Shaders

Experiments own shader code and specific parameters in their `shaders/` directory. Lab owns generic pipeline application and may compile, manage, and bind shaders, supplying standard uniforms such as simulation time, viewport size, and camera information.

A lensing effect is an example: the experiment supplies the shader/model while Lab provides the rendering pipeline. Reusable effects move into Lab only after demonstrated reuse. Shader API details remain open.

### Design decisions

World-space geometry separates simulation from screen presentation. Retained and immediate paths support both persistent objects and procedural visuals. Separating spatial geometry, visual descriptions, and render order leaves room for 3D without forcing an object hierarchy onto domain code.

## 20. Audio and runtime errors

### Audio

The experiment decides when and why a sound occurs. Lab supplies loading/playback, resource handling, mute, and master volume.

Simulation-world audio follows simulation speed by default, including its pitch/time behaviour. Sounds may opt out of speed-dependent pitch changes for UI sounds or music. Exact APIs, generated-audio support, and pause behaviour remain implementation questions.

### Runtime errors

A runtime failure caught at a Lab/experiment callback boundary is terminal for that application run.

```text
Caught callback error
        ↓
Stop normal interaction
        ↓
Modal error and traceback
        ↓
Single OK action
        ↓
Close experiment/application
```

Normal controls and experiment interaction are blocked. Lab does not repair state, reset, or continue after the failure. Errors too severe to catch fall back to runtime/LÖVE fatal-error behaviour.

The intended equivalent of closing an application in a browser/iframe build is not yet specified.

### Design decisions

Audio uses the same generic-mechanism/domain-meaning boundary as rendering. The terminal error policy replaces the earlier tentative pause-and-debug recovery direction. It avoids continuing from experiment state that may be invalid.

# Part D. Interface and interaction

## 21. Application layout

The experiment remains visually dominant. A small menu button in the top-left toggles a collapsible sidebar. Core simulation and audio controls stay in a persistent bottom bar.

```text
┌────────────────────────────────────────────────────────────┐
│ Menu button              Experiment rendering area          │
│ ┌───────────────────┐                                      │
│ │ Sidebar tabs      │                                      │
│ │ Instrumentation   │                                      │
│ │ Debugging         │                                      │
│ │ Controls          │                                      │
│ │ Data              │                                      │
│ │ Scrollable content│                                      │
│ └───────────────────┘                                      │
├────────────────────────────────────────────────────────────┤
│ Restart · Step · Speed    Play/Pause    Mute · Master Volume │
└────────────────────────────────────────────────────────────┘
```

This is a structural sketch, not a chosen visual style. Play/Pause stays visually stable and isolated in the centre. Speed has a numeric readout.

The retained initial preference is for the sidebar to overlay the experiment rather than resize its viewport. This can be revisited if real experiments show a better approach. The bottom bar's precise viewport treatment remains part of UI implementation.

Each sidebar tab has a scrollable content viewport. Lab lays out controls in registration order with consistent spacing and scrolling when needed. Experiments do not place controls in pixels, paginate them, or manage overflow. Tab scroll positions may persist during the session.

### Design decisions

The overlay preference avoids changing rendering dimensions simply because controls are opened. Persistent simulation controls make them accessible without opening Debugging. Lab-owned layout keeps interface mechanics out of experiment code.

## 22. Instrumentation and parameter controls

Lab renders generic widgets. Experiments declare which controls exist, what they mean, and which callbacks handle changes.

Controls are conventionally declared in `controls.lua` using explicit typed registration for sliders, checkboxes, buttons, dropdowns, numeric inputs, labels, and other supported widgets. Control type is not inferred from metadata. UI declarations use function references for callbacks rather than embedding callback implementations inline.

An illustrative declaration is:

```lua
function Controls.register(lab, experiment)
    lab.ui.registerSlider({
        id = "body_count",
        label = "Celestial Bodies",
        default = 2,
        min = 1,
        max = 12,
        integer = true,
        record = true,
        randomizable = true,
        onChange = experiment.setBodyCount
    })
end
```

Exact public names remain open. Lua functions are the normal bridge between Lab controls and experiment behaviour; no formal binding/context hierarchy is required.

### Run Configuration and Live Controls

| Group | Behaviour |
|---|---|
| Run Configuration | Initial-condition parameters. Editable while preparing a run, automatically recorded, and locked when the run begins. |
| Live Controls | Controls that do not define reproducible initial conditions and may remain usable during simulation, such as showing trails or vectors. |

A recorded control is a run parameter, not just a value written to a file. It has a stable restoration ID, default, appropriate validation, and automatic inclusion in run records. Optional tooltip text explains meaning while labels stay concise. Recorded parameters should have a visible indication; the iconography is not fixed.

Compatible bounded controls may opt into user-triggered randomisation. The default is off. When enabled, a randomise action chooses a valid value within the declared constraints and routes it through the normal change callback. Randomisation is never automatic.

### Availability

Controls normally remain visible but disabled when their capability is unavailable. Both capability and lifecycle state determine enablement. Play and Step need update capability; Pause needs an advancing simulation; restoration needs reset capability. Section 15 records lifecycle behaviour and its unresolved cases.

### Design decisions

Instrumentation is part of the normal interactive experiment, not only debugging. Explicit control types keep declarations clear. Stable IDs and validation support restoration without coupling saved data to display labels.

## 23. Debugging, bindings, and Data tabs

### Debugging

Lab supplies common facilities such as FPS/frame timing, coordinate/grid overlays, vector visualisation, debug drawing and bounds, object inspection where meaningful, and logging. Other diagnostics should be earned through implementation. Simulation controls live in the bottom bar.

### Controls and input priority

The Controls tab is a read-only view of effective keyboard/mouse bindings. Lab defaults appear first, followed by experiment bindings. When an experiment overrides a default using the same effective input, the superseded default is not shown separately.

| Initial 2D input | Default action |
|---|---|
| Spacebar | Play/Pause |
| Arrow keys | Camera pan/movement |
| Mouse wheel | Camera zoom |

Experiments register callbacks for their own meanings and may override defaults. UI focus or consumed input should prevent unintended camera/experiment action from the same input. Camera rotation is not a universal 2D default. Future 3D controls await a real 3D experiment.

### Data

The Data tab is always available. Actions enable or disable according to available data, platform capability, and lifecycle state.

```text
Data
├── Run history and metadata/details
├── Load Run
├── Import Run / Export Run
├── Recorded measurements
├── Export Measurements to CSV
└── Open Data Folder [native platforms where applicable]
```

Import remains available with no local history. Importing or exporting a record need not change the current simulation. Loading/applying one restores initial conditions and is lifecycle-sensitive. Restored controls receive a visible but unobtrusive indication. Part E defines the data behaviour.

### Design decisions

The Controls tab shows what inputs actually do after overrides. Separating import from application allows data management without unexpectedly changing a running experiment.

# Part E. Run data and observations

## 24. Randomness and automatic run records

Lab provides seeded pseudorandom generation through an abstraction over the underlying runtime/library. LÖVE's independent random generators are the expected initial implementation; Lab does not implement a custom generator.

The seed is a Lab-owned Run Configuration parameter. It is visible, protected from accidental typing, and explicitly editable or rerollable before the run. It is recorded automatically and locked once the run begins.

Configuration randomisation must not consume the simulation's future random sequence. Lab establishes/resets the run RNG at the run boundary.

When Play begins a new run, Lab automatically records:

```text
Run record
├── Metadata
│   ├── Experiment version
│   ├── Lab version
│   ├── Template/provenance information where useful
│   ├── Creation/start timestamp
│   └── Completion information, where provided
├── Initial configuration
│   ├── RNG seed
│   ├── Recorded parameter IDs and values
│   └── Relevant simulation settings, such as fixed timestep
└── Measurements
    └── Simulation-time/value samples for each measurement
```

The file format remains open. The record captures initial conditions and experimental data, not arbitrary tables, object graphs, functions, callbacks, shaders, or mid-run save state.

### Design decisions

Recorded parameters and a seed support repeating initial conditions. Lab's RNG abstraction allows the implementation to change without changing experiment code. The exact reproducibility contract across versions, platforms, and state-changing live interaction remains to be clarified.

## 25. Measurements and observations

Experiments register meaningful values with generic Lab measurement infrastructure. Each measurement has a stable ID, optional metadata such as label/unit, and a sampling callback.

```lua
lab.measurements.register({
    id = "total_energy",
    label = "Total System Energy",
    unit = "J",
    interval = 0.1,
    sample = experiment.getTotalEnergy
})
```

API names are illustrative. Lab supplies a sensible default interval; individual measurements may override it.

```text
Simulation time → Sampling schedule → Experiment callback
                                            ↓
                                  Recorded time/value samples
                                            ↓
                                  Data inspection and CSV export
```

Sampling follows simulation time, not render frames or wall-clock time. Simulation speed changes how quickly samples are produced in real time without changing their simulation-time meaning.

Measurements may describe entity properties, derived values, or system-wide quantities such as total energy, average flock speed, or distance between objects. Samples become part of the run's experimental data.

Discrete events/results, such as reaching a finish at a particular time, may later receive their own observation abstraction. They are not forced into periodic sampling in the initial design.

### Design decisions

Callbacks support computed quantities that a property-name-only mechanism would not express. CSV provides an initial path to external analysis without requiring a built-in plotting system. Sampling order, boundary samples, supported value types, and completion interaction remain open.

## 26. Restoration, import/export, and storage

### Restoration

Load Run and Import Run feed the same restoration mechanism when a record is applied:

```text
Validate and read record
          ↓
Restore seed and match stable parameter IDs
          ↓
Update recognised controls through normal callbacks
          ↓
Indicate restored values and report values not restored
```

A historical record should not fail wholesale because a newer experiment no longer recognises one parameter. The compatibility policy and handling of invalid or changed parameter values remain open.

### Storage

| Platform | Automatic history | Portability |
|---|---|---|
| Native | Persist in the platform's writable application-data/save location. | Import/Export; Open Data Folder where applicable. |
| Browser | Session/page-memory only. Reloading or leaving may discard it. | Explicit file Export and later Import. |

The native logical layout is conceptually:

```text
CCL/
└── experiments/
    └── <experiment>/
        └── runs/
```

Persistent browser storage such as IndexedDB is not required. GitHub Pages and iframe experiments need no server-side run storage. File naming, retention, record compatibility, and exact format remain implementation details.

### Design decisions

Stable parameter IDs separate restoration from labels and UI layout. Native persistence supports local history; explicit file transfer gives browser users persistence without adding a storage service.

# Part F. Implementation, open questions, and roadmap

## 27. Initial development sequence

The existing sequence is retained. Focused questions may be settled before code where they materially affect the first implementation. After that, working experiments should guide architectural change.

1. Create the Lab repository and its version/release metadata.
2. Implement the minimal LÖVE runtime and Ready/Running lifecycle.
3. Add fixed-step timing, Play, Pause, Step, Restart/new-run orchestration, and speed control.
4. Add the initial 2D camera and coordinate transforms.
5. Add the sidebar, typed controls, Run Configuration/Live Controls, and bottom bar.
6. Add seeded randomness and minimal automatic run records.
7. Add optional experiment-defined completion and simulation-time measurements.
8. Add run-data inspection and CSV export.
9. Add the minimal entity registry and capabilities.
10. Add spatial queries/picking, basic contact detection, and retained primitive rendering.
11. Add initial audio facilities.
12. Create the Template structure, manifest, dependency tooling, and inherited GitHub Actions.
13. Create the first representative experiment. Orbital mechanics remains a suitable architecture exercise.
14. Observe friction, missing abstractions, and unnecessary abstractions.
15. Promote proven reusable concepts into Lab with documentation and tests.
16. Create another experiment and repeat.

The initial release and GitHub Pages deployment path is part of the delivery work described in Part B. This edition does not silently defer features or reorder implementation priorities to reduce scope.

## 28. Open questions

These questions do not grant permission to replace the existing intent with a new design. They identify details or inconsistencies that still need an explicit decision.

### Behaviour needing focused review

| Topic | Unresolved point |
|---|---|
| Run start | Whether first Step creates/locks a run record, as first Play does. |
| Configuration preview | How editable parameters and seed affect the visible initial scene before execution. |
| Restart, reset, and new run | Their exact relationship; return to editable configuration; RNG/clock/accumulator behaviour; registration/contact cleanup. |
| Completion | Callback/result shape, check order, transition into Completed, and allowed controls afterwards. |
| Run records | Record identity on restart/load, finalisation, preservation or replacement of samples, naming, retention, and compatibility. |
| Measurements | Initial/final samples, interval alignment with fixed steps, timestamp meaning, and pause/step/restart/completion interaction. |
| Reproducibility | Meaning across versions/platforms and when user interaction changes domain state during a run. |
| Error exit | Browser/iframe equivalent of native application close and handling of available run data. |
| Template updates | How compatible updates apply to inherited files and handle experiment edits; explicit compatible-version selection. |
| Case workflow | Repeated starts at a whole-version boundary, initial Case setup, and progress numbering beyond the illustrated examples. |

### Implementation and API details

- Public Lab API, module names, lifecycle callback signatures, and the boundary for direct LÖVE access versus wrappers.
- Entity capability shapes, primitive/render descriptions, and initial camera API.
- Typed widget registration names, metadata, and exact binding API.
- Input press/hold/modifier semantics.
- Contact dispatch, participating recipients, and contact information.
- Fixed-step defaults/configuration, backlog cap, and overflow policy.
- Measurement API, default interval, and initial supported value types.
- Render interpolation if needed.
- Shader API, audio API, generated audio, and simulation-audio pause behaviour.
- Detailed diagnostic/inspection rollout.
- Run-record format, supporting version filenames, and script names.
- Browser build tooling and Linux distribution format.
- Concrete 3D architecture when justified by an experiment.

### Design decisions

Use working code to settle API details where possible. Keep pre-code discussion focused on choices that materially shape the first implementation, rather than reopening broad speculative subsystem design.

## 29. Future work

### Built-in charting and observations

The initial Lab supports measurement inspection and CSV export. Later, real plotting needs may justify generic charts for live or recorded measurements, including time series and comparisons.

Chart types, axes, series grouping, styling, configuration, and live-versus-recorded presentation are deferred. A separate discrete-event/result observation model is also a future possibility.

### Playground or catalogue

A future public catalogue may provide a front door to the project:

```text
Catalogue
├── Links to Lab and Template
├── Links to independent experiments
├── Screenshots, animations, and descriptions
└── Optional scripts to clone a curated or complete collection
```

The catalogue references projects; it does not own them. Lab and experiments must work without it.

### Further rendering and shared facilities

3D work begins when a real experiment needs it. Shared effects, physics response, spatial acceleration, and other facilities are added only when use demonstrates their value.

## 30. Document status and review

Version 0.8 is a structural and editorial consolidation. It includes the current runtime, UI, and run-data direction alongside the restored repository, testing, versioning, Case, release, distribution, and catalogue material. Readers do not need an earlier design document to recover an operative requirement.

Historical status summaries, duplicated principles, stale version labels, inconsistent numbering, and broken illustrative snippets have been replaced by the grouped sections above. Existing uncertainty is recorded in section 28 rather than being presented as a settled choice.

After review, this source can support two separate documents: a developer reference covering architecture, tooling, workflows, and reasons for decisions; and a user-facing introduction covering the project and how to use experiments. Neither derived document is created by this revision.

The design remains open to evidence from implementation and real experiments.
