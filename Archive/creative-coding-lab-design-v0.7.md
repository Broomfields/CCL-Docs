---
title: Creative Coding Lab
---

*Design Document v0.7 · Living Design Document*

  ---------------------------------------------------------------------
  Field                              Value
  ---------------------------------- ----------------------------------
  Status                             Living design document

  Version                            0.6

  Initial platform                   Lua + LÖVE

  Project type                       Personal creative-coding platform
                                     and collection of independent
                                     experiments
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
lifecycle integration, input, timing, rendering infrastructure, cameras,
experiment UI, entity inspection, and debugging facilities.

Core summary:

> A lightweight LÖVE-based creative coding runtime, paired with a
> standard experiment template, that makes it easy to create independent
> visual programming experiments while keeping shared infrastructure out
> of the way of the ideas being explored.

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
input, timing, UI, spatial queries, entity registration, and debugging.
It should not understand planets, boids, Fourier transforms, terrain, or
other experiment-specific domains.

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
  Technology                         Role
  ---------------------------------- ----------------------------------
  Lua                                Primary language for the Lab and
                                     experiments.

  LÖVE                               Initial application, graphics,
                                     input, and runtime framework.

  POSIX shell                        Bootstrap, dependency-fetching,
                                     and development scripts.

  Markdown                           Project documentation and
                                     experiment lab-notebook
                                     documentation.

  TOML                               Experiment manifest and committed
                                     version/provenance metadata.

  Git                                Source control and version/tag
                                     foundation.

  GitHub                             Repository hosting, releases,
                                     distribution, and public portfolio
                                     presentation.
  ---------------------------------------------------------------------

VS Code is the intended local editor and may be used with a multi-root
workspace for convenience, but it is not a Creative Lab technology
dependency. The repositories must remain editor-agnostic.

# 5. High-Level Architecture

``` text
Creative Lab
│
├── Shared runtime / reusable infrastructure
│
└── Experiment template
    │
    ├── Experiment A
    ├── Experiment B
    └── Experiment C
```

Within an experiment:

``` text
Experiment coordinator
│
├── Experiment-owned state
├── Domain definitions
├── Simulation / mathematical model
├── Experiment-defined instrumentation
├── Experiment-defined capabilities
│
└── Lab runtime
    ├── lifecycle
    ├── simulation timing
    ├── entity registry
    ├── spatial queries / picking
    ├── rendering
    ├── input
    ├── camera
    ├── UI shell
    └── debugging / inspection
```

The central boundary is ownership: the Experiment owns domain meaning
and state; the Lab owns generic mechanisms.

# 6. Lab Runtime

Creative Lab is shared infrastructure used by independent experiments.
Its API should remain small and should grow in response to demonstrated
needs.

## 6.1 Intended responsibilities

-   LÖVE lifecycle integration and application setup.
-   Real-time and simulation-time scheduling.
-   Play, pause, single-step, reset orchestration, and simulation speed.
-   Input capture and generic coordinate transformation.
-   2D rendering infrastructure and drawing helpers.
-   Camera facilities.
-   Generic entity registration and inspection.
-   Generic spatial queries, picking, and basic contact detection.
-   A standard collapsible sidebar, persistent simulation controls, and
    generic UI controls.
-   Lab-provided debugging and instrumentation facilities.
-   Input binding/default-control infrastructure.
-   Generic audio loading/playback, master volume, and mute.
-   Seeded pseudorandom generation and run/reproducibility
    infrastructure.
-   Generic visualisation utilities that have demonstrated reuse across
    experiments.

## 6.2 Non-responsibilities

-   Experiment-specific domain models or terminology.
-   Experiment-specific simulation rules.
-   Physics simulation or collision response by default.
-   Owning Experiment domain state.
-   Inferring the meaning of a selected domain object.
-   Premature abstractions for concepts not yet shown to be reusable.
-   Owning or cataloguing the collection of experiments.

# 7. Experiments

Each experiment is an independent runnable LÖVE application and an
independent public Git repository. It owns its entry point, domain
vocabulary, domain definitions, mathematical/simulation model, live
domain state, initial conditions, reset semantics, and the controls it
exposes through the Lab's instrumentation UI.

The Experiment Template provides a conventional lightweight structure:

``` text
experiment/
├── experiment.lua      # domain state, lifecycle/orchestration, callbacks
├── definitions.lua     # optional domain-data constructors/definitions
├── simulation.lua      # mathematical/model functions
├── controls.lua        # Experiment UI declarations
└── shaders/            # Experiment-owned shaders
```

`definitions.lua` is a conventional home for domain-data construction,
not a framework type system. Plain Lua tables remain the normal
representation. Experiments are not expected to recreate C#/Java-style
class hierarchies, interfaces, or inheritance unless a real problem
makes such a technique useful.

`simulation.lua` should normally contain the mathematical/model layer.
Pure calculations are preferred where natural, but functions may mutate
state explicitly passed to them in accordance with the general mutation
rule. It should not secretly reach into Experiment-owned global state.

`controls.lua` explicitly declares Experiment UI controls through typed
Lab registration functions. A conventional entry point is:

``` lua
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

The exact public names remain implementation details. The important
contract is that control type is explicit rather than inferred from
metadata, and callbacks are function references rather than callback
implementations embedded inline.

Experiment code is exploratory by design. It may be refactored when
useful, but production-style ceremony must not become a prerequisite for
trying an idea.

# 8. Experiment Entry Point and Lifecycle Contract

The experiment owns `main.lua`. The Lab supplies runtime facilities
rather than acting as the application itself.

A possible entry shape remains:

``` lua
local Lab = require("vendor.lab.runtime")
local Experiment = require("experiment")

Lab.run(Experiment)
```

The exact public API naming remains an implementation detail, but the
lifecycle semantics are now defined.

## 8.1 Initialization

Experiment initialization occurs once when the Experiment/application is
loaded, before the user can press Play.

Initialization may:

-   create Experiment state;
-   create the initial scene/domain objects;
-   register objects and their Lab-facing capabilities;
-   define instrumentation;
-   provide the Experiment's update and reset behaviour.

After initialization the application enters **Ready**. The initial scene
is visible, simulation time is zero, and the simulation is not
advancing.

Play does not initialize the Experiment.

## 8.2 Simulation lifecycle

``` text
APPLICATION LOAD
      ↓
Experiment initialization
      ↓
READY ───── Play ─────► RUNNING
  ▲                       │
  │                       │ Pause
  └──────── Reset ◄───────┘

While paused/ready:
Step → exactly one simulation step
```

Lab owns the lifecycle controls and scheduling. Experiment owns the
meaning of advancing or restoring its domain state.

No `onPlay()` or equivalent callback is required unless a real
experiment demonstrates a need for one.

A **run** is one execution from a defined set of initial conditions. A
**session** is one launch of the Experiment application and may contain
multiple runs.

Run Configuration parameters are editable while preparing a run. When
Play begins a new run, Lab snapshots the seed and recorded run
parameters, creates the run record, and locks those parameters for that
run. Restart reproduces the same run rather than silently creating a
different configuration.

The exact UI/lifecycle action for leaving a completed/current run and
returning to configuration for a genuinely new run remains to be settled
before implementation.

# 9. Simulation and Domain Logic

Where useful, the mathematical/model core should live in the
Experiment's `simulation.lua`, separated from rendering, input, Lab
interaction, and ownership of the overall Experiment state.

For example:

``` lua
function Simulation.gravitationalForce(mass_a, mass_b, distance)
    return force
end

function Simulation.acceleration(force, mass)
    return force / mass
end
```

`experiment.lua` remains the coordinator. It owns live domain state and
calls the simulation/model functions while advancing the Experiment:

``` lua
function Experiment.update(dt)
    for _, body in ipairs(Experiment.bodies) do
        local force = Simulation.calculateForce(...)
        local acceleration = Simulation.acceleration(force, body.mass)
        body.velocity = Simulation.velocityAfterStep(body.velocity, acceleration, dt)
    end
end
```

Pure calculations should naturally return values. A thin mutating layer
may apply results to explicitly passed Experiment state.

Purity is a tool rather than a rule. A simulation function may mutate an
object explicitly passed to it where that is clearer. What it should not
do is secretly reach out to unrelated Experiment state.

Physics remains Experiment-owned. If an Experiment needs momentum, mass,
impulse response, rigid-body behaviour, or other physics, those concepts
belong in its own model/simulation code until repeated real reuse
demonstrates that a generic Lab facility has been earned.

# 10. Experiment State

Experiment-specific data belongs with the Experiment.

The default model is a plain Lua **state table** explicitly passed to
code that needs it. The architecture does not introduce formal
read-only/writable context types or a framework-owned state container.

For example:

``` lua
local state = {
    bodies = {},
    selected_body = nil,
    parameters = {
        gravity = 1
    }
}
```

Coordinator functions may receive the complete state table. Lower-level
functions should receive narrower values or sub-state where that makes
their dependencies clearer.

The Lab must not require every function to receive Lab or Experiment
state. Simulation-only calculations should ideally depend on neither
LÖVE nor Lab.

# 11. Ownership and Mutation Rules

  Concern                                             Owner
  --------------------------------------------------- ------------------------
  LÖVE lifecycle integration                          Lab
  Real-time and simulation scheduling                 Lab
  Camera and viewport state                           Lab
  Raw input state                                     Lab
  Sidebar shell and generic widgets                   Lab
  Debugging facilities                                Lab
  Entity registration records                         Lab
  Domain objects and collections                      Experiment
  Simulation/domain rules                             Experiment
  Experiment parameters and instrumentation meaning   Experiment
  Domain selection state                              Experiment
  Initial conditions and reset semantics              Experiment
  Proven reusable concepts                            May be promoted to Lab

## 11.1 Mutation rule

A function may mutate state explicitly passed to it. Functions should
avoid reaching out to mutate unrelated hidden external state.

This is not a requirement to mutate arguments. Pure calculations should
return values normally where that is clearer.

Within an ownership boundary, mutation versus return is an
implementation choice. Across the Lab/Experiment boundary, neither side
should secretly mutate state owned by the other.

## 11.2 Dependency direction

Code should receive the narrowest useful dependency.

-   A simulation calculation may receive bodies and `dt`.
-   A coordinator may receive the complete Experiment state.
-   Code that needs a Lab service may receive Lab or the relevant Lab
    service.
-   Code that needs neither should know about neither.

# 12. Repository Architecture

The local Creative Coding workspace is **not** itself a Git repository.
It groups independent repositories for development convenience:

``` text
creative-coding/ [local workspace; not a Git repo]
├── creative-lab/ [GIT REPO]
├── experiments/
│   ├── orbital-mechanics/ [GIT REPO]
│   ├── boids/ [GIT REPO]
│   └── ...
├── creative-lab-experiment/ [GIT REPO / GitHub template]
└── creative-coding.code-workspace [optional local convenience]
```

Each Experiment is a separate application and repository. There is no
persistent Lab process that unloads one Experiment and loads another.
Each Experiment runs with its own fetched/vendored Lab dependency.

Repositories remain independent. Git submodules are intentionally
avoided; dependency retrieval is handled by bootstrap tooling.

# 13. Lab Dependency, Versioning, Cases, Testing, Git and Release Model

The dependency, versioning, Case lifecycle, testing, Git, CI, release,
and distribution decisions established through v0.4 remain in force.

In summary:

-   Lab and Template are independently Semantic Versioned.
-   Experiments use completed-Case/progress numbering rather than
    SemVer.
-   `experiment.toml` records Experiment, Template, and Lab provenance.
-   Sync Lab may update minor/patch releases only within the current Lab
    major.
-   Busted is the standard Lua testing framework; Lab/Template are
    production-quality, Experiment tests are optional.
-   Lab/Template use stable `main`, short-lived branches, PR validation,
    and PR-level `release:*` intent.
-   Experiments may use a more relaxed Git workflow.
-   Experiment releases target browser/GitHub Pages, Windows, macOS,
    Linux, and optionally `.love`.
-   `EXPERIMENT.md` remains the living experiment notebook with Case
    history.

# 14. Experiment UI

Creative Lab owns a standard application UI shell while the Experiment
remains visually dominant. The normal application view is the Experiment
rendering area with a small menu button in the top-left corner.

The sidebar contains four first-class tabs:

1.  **Instrumentation**
2.  **Debugging**
3.  **Controls**
4.  **Data**

Each tab owns a scrollable content viewport. Experiment code does not
position controls in pixels, paginate them, or manage overflow. Lab lays
controls out in registration order with consistent spacing and
introduces scrolling when required. Tab scroll positions may be retained
for the current session.

## 14.1 Instrumentation

Instrumentation is rendered by Lab but defined by Experiment. Lab
supplies generic widgets; Experiment defines what controls exist, their
metadata, and what changing them means.

Experiment controls are conventionally declared in `controls.lua`
through explicit control-specific registration calls such as slider,
checkbox, button, dropdown, or numeric input registration.

Controls may provide optional help/tooltip text. Lab renders the
corresponding help affordance; the label remains concise while the
tooltip can explain meaning or purpose.

Instrumentation distinguishes two conceptual groups:

**Run Configuration** contains parameters that define a run's initial
conditions. These parameters are automatically recorded, are editable
before the run, and become immutable when the run begins.

**Live Controls** contains controls that do not define reproducible
initial conditions and may remain usable while the simulation runs
according to their normal behaviour. Examples include purely visual
toggles such as showing trails or vectors.

A recorded/run parameter should be visually identifiable so the user can
see which values will be included in run data. The exact iconography is
a UI implementation detail.

## 14.2 Recorded parameters and randomization

A control marked for recording is treated as a run parameter rather than
merely as a value to write to a file.

A run parameter has:

-   a stable ID used for restoration;
-   a default value;
-   control-specific validation such as bounds/integer requirements;
-   optional tooltip/help text;
-   automatic inclusion in run records;
-   immutability once the run begins.

Compatible bounded controls may **opt in** to user-triggered
randomization. Randomization is never automatic. The opt-in defaults to
false.

If enabled, Lab presents a randomize action (conceptually a dice button)
that selects a valid value within the control's declared constraints and
routes the change through the same callback as a manual edit.

## 14.3 Debugging

Lab owns generic debugging facilities such as:

-   FPS/frame timing;
-   coordinate/grid overlays;
-   generic inspection;
-   debug drawing;
-   other diagnostics earned through implementation.

Simulation controls are not buried in Debugging.

## 14.4 Controls tab

The Controls tab is a read-only view of the effective keyboard/mouse
bindings for the current Experiment.

Lab defaults appear first. Experiment-specific bindings follow. If
Experiment overrides a Lab default by registering the same input, the
superseded default entry is not shown separately; the effective
Experiment binding is shown in the appropriate custom/effective
position.

## 14.5 Data tab

The Data tab is always available. Individual actions enable or disable
according to available data, platform capability, and lifecycle state.

It provides the UI for:

-   run history;
-   Load Run;
-   Import Run;
-   Export Run;
-   run metadata/details;
-   recorded measurements/tracked values;
-   **Export Measurements to CSV**;
-   Open Data Folder on native platforms where applicable.

Import remains available even when there is no local run history.
Importing a record does not have to apply it immediately to a running
simulation.

Loading/applying a run is lifecycle-sensitive because it restores
initial conditions. Exporting or importing a run record is a
data-management action and need not mutate the current simulation.

When a run is loaded, Lab restores recognized run-parameter IDs and the
RNG seed through the normal control/callback path. Restored controls
should receive a visible but unobtrusive indication so the user can see
what was populated from the record.

## 14.6 Persistent bottom control bar

Core simulation and audio controls live in a persistent bottom bar
rather than in the sidebar.

The intended layout is:

``` text
Left:    Restart · Step · Simulation Speed
Centre:  Play/Pause
Right:   Mute · Master Volume
```

Play/Pause is visually stable and isolated in the centre.

Restart is disabled until the simulation has advanced and reproduces the
current run's initial conditions. Step is available while Ready/Paused
and disabled while running. Simulation speed remains directly accessible
with a numeric readout.

Exact visual styling remains an implementation detail.

# 15. Entity Registration

Lab provides a generic entity registry for Experiment objects that need
Lab services such as rendering, picking, or inspection.

An Experiment domain object does not inherit from a Lab base entity and
does not become Lab-owned.

For example:

``` lua
local earth = {
    name = "Earth",
    mass = 5.972e24,
    position = { x = 100, y = 200 },
    velocity = { x = 0, y = 10 },
    radius = 20
}
```

Experiment may register the object:

``` lua
lab.entities.register(earth, {
    spatial = {
        shape = "circle",
        position = function(entity)
            return entity.position
        end,
        radius = function(entity)
            return entity.radius
        end
    },
    render = {
        describe = function(entity)
            return {
                type = "circle",
                position = entity.position,
                radius = entity.radius
            }
        end
    }
})
```

Exact names are not locked.

Lab stores a Lab-owned registration record containing an opaque
reference to the Experiment-owned object plus generic Lab-facing
capabilities.

Registration does not transfer ownership.

## 15.1 Capability model

Capabilities describe what generic Lab services can do with an object
without teaching Lab its domain meaning.

Likely capability families include:

``` text
Entity registration
├── spatial
├── render
├── inspect
└── future capabilities earned by real use
```

This is component-shaped but is not declared to be an Entity Component
System.

Callbacks/accessors read the live Experiment object, so Experiment does
not need to duplicate positional or visual state into Lab or notify Lab
every time a value changes.

# 16. Entity Lifetime

Experiment owns object/domain lifetime. Lab owns only registration
lifetime.

Typical flow:

``` text
Experiment creates object
        ↓
Experiment stores object in its state
        ↓
Experiment registers object with Lab
        ↓
Experiment mutates object normally
        ↓
Lab capabilities read current object state
        ↓
Experiment unregisters object when Lab services are no longer needed
        ↓
Experiment may remove/destroy object according to its own rules
```

`unregister` does not destroy the object. It means only that Lab stops
providing registered services for it.

Because each Experiment is its own application/runtime instance, no
architecture is required for switching from Experiment A to Experiment B
inside one persistent Lab process. Application teardown naturally ends
the Lab registration lifetime.

No entity manager, factory, pooling system, destruction event system, or
formal ECS is introduced without demonstrated need.

# 17. Input, Coordinate Spaces, Selection and Spatial Queries

Lab owns raw input, effective input bindings, UI input priority, and
camera transforms.

Experiment decides what Experiment-specific controls mean and registers
callbacks/delegates for them. Lab provides sensible default bindings
that Experiment may override by registering the same effective input.

Initial 2D defaults include:

-   Spacebar for Play/Pause;
-   arrow keys for camera pan/movement;
-   mouse wheel for camera zoom.

Camera rotation is not treated as a universal 2D default. Future 3D
camera controls should be designed against a real 3D Experiment rather
than speculated now.

When UI has focus or consumes an input, Lab should prevent that same
input from unintentionally reaching camera/Experiment behaviour.

Experiment should not need to know screen pixels for world interaction.

For a pointer interaction:

``` text
screen input
    ↓
Lab camera / coordinate transform
    ↓
world-space query
    ↓
Lab generic spatial query
    ↓
Experiment interprets returned object(s)
```

Selection is Experiment-owned domain state. For example, an orbital
experiment may set:

``` lua
state.selected_body = hit.entity
```

Lab may provide generic inspection UI for the selected/returned entity,
but Lab does not decide that the object means "selected planet."

## 17.1 Spatial capability

The preferred spatial model describes generic geometry rather than
delegating all picking to an arbitrary Experiment `hit_test` function.

For example, an Experiment can expose a circle's current position and
radius. Lab then performs the generic point/circle or ray/shape
intersection.

This allows Lab to reason about geometry and leaves room for future
spatial acceleration without changing the Experiment's domain model.

Custom hit-test callbacks may remain an escape hatch for unusual cases.

## 17.2 Query results and overlap

The spatial API should support multiple hits even if a convenience
`pick()` operation returns one.

A generic result may contain:

``` lua
{
    entity = object,
    position = world_position,
    distance = distance
}
```

The architecture must not define spatial identity as "first registered
wins."

For 2D, render ordering may sometimes provide a useful interaction
tie-breaker. For future 3D, ray distance will normally be the more
meaningful ordering.

2D render order and 3D world Z are separate concepts and must not be
conflated.

# 18. Contact Detection

Lab provides basic generic **contact/collision detection**, not a
physics engine.

The same generic spatial geometry used for picking may be used to
determine whether registered shapes overlap. Lab detects and tracks
contact; Experiment decides what contact means.

Contact state is tracked per registered pair:

``` text
separate
   ↓ first overlap
ENTER
   ↓ remains overlapping
STAY
   ↓ separates
EXIT
```

The normal collision/contact callback is an enter notification fired
once per contact episode. Optional stay/exit callbacks may be supported
for Experiments that need continuous or separation behaviour.

Callbacks are associated with registered entities/capabilities so
different Experiment objects may react differently. Exact dispatch
details, including whether both participating registrations receive each
event and the final contact-info shape, remain implementation/API
questions.

Contact information may eventually include generic details such as
contact point or normal where the geometry implementation can provide
them.

Collision/contact layers are separate from render ordering.
Z-index/render order must not be used to decide which objects can
contact one another.

Lab does not prevent interpenetration, apply impulses, calculate
momentum, bounce objects, or otherwise impose physical response.
Experiment may allow two shapes to remain overlapped or may implement
any response appropriate to its domain.

If multiple Experiments later demonstrate genuinely reusable
physics-response code, that code may earn promotion into Lab. No physics
engine is introduced in advance.

# 19. Rendering Contract

Experiment owns world-space geometry/procedural definitions and
appearance intent.

Lab owns camera, projection, viewport, graphics-resource management, and
final screen-space rendering.

World geometry is expressed in world units:

``` lua
position = { x = 100, y = 200 }
radius = 10
```

The Experiment does not convert that radius to pixels. Lab's
camera/projection determines its screen representation.

## 18.1 Retained registered rendering

Registered entities are persistent renderables.

Experiment registers the object once. On each render frame, Lab reads
the object's current render capability and renders the resulting
representation. The Experiment does not resubmit the object every
simulation tick.

This is the normal model for persistent domain objects such as planets
or agents.

## 18.2 Immediate/procedural rendering

Lab should also provide a transient drawing path for geometry that
naturally exists only for the current frame, such as:

-   generated curves;
-   graphs;
-   fields;
-   trails;
-   mathematical visualisations;
-   debug geometry.

Immediate drawing does not require creating thousands of persistent
entities merely to issue temporary primitives.

Therefore not everything rendered by Lab is an entity.

## 18.3 Spatial shape versus render shape

Spatial and render capabilities are deliberately separate.

An object may:

-   render with complex geometry but use a simple picking shape;
-   be visible but not selectable;
-   be selectable/inspectable without having a normal visible
    representation.

## 18.4 Procedural geometry

The initial renderer should support straightforward primitives such as
circles, rectangles, lines, and polygons. Experiment may compute more
complex geometry and describe it using generic Lab-understood
representations.

Exact primitive/API names are deferred to implementation.

## 18.5 Shaders

LÖVE shader support is compatible with this architecture.

Experiment owns shader code and experiment-specific parameters. The
Template provides a conventional `shaders/` directory, which may remain
empty when unused.

Lab owns generic application of shaders in the render pipeline and may
compile/manage/bind them and provide standard uniforms such as
simulation time, viewport size, and camera information.

A gravitational-lensing/post-processing effect is a representative case:
Experiment owns the lensing shader/model while Lab provides the generic
rendering pipeline in which it is applied.

Reusable effects may be promoted into Lab only after real reuse
demonstrates value.

The shader API is intentionally not designed in detail yet.

# 20. Audio

Audio is a generic Lab facility.

Experiment decides **when and why** a sound occurs. Lab owns generic
loading/playback, master volume, mute, and the underlying audio-resource
mechanism.

Simulation-world audio follows simulation speed by default: pitch/time
behaviour should reflect the speed of the simulated world. A sound may
explicitly opt out of simulation-speed pitch mutation for cases such as
UI sounds or music.

The exact audio API, generated-audio support, and pause behaviour for
simulation-bound sounds remain implementation questions.

# 21. Randomness and Reproducible Run Data

Lab provides seeded pseudorandom generation using the underlying
runtime/library implementation rather than implementing a custom
random-number generator.

LÖVE's independent random-generator facilities are the initial expected
implementation. Experiment receives randomness through Lab's abstraction
so the implementation can change without changing Experiment code.

The seed is a Lab-owned **Run Configuration** parameter. It is visible
and editable before a run, recorded automatically, and locked once the
run begins.

The seed control is read-only in the sense that normal accidental typing
is avoided, but while Run Configuration is editable the user may
explicitly edit the seed or reroll it through a Lab-provided randomize
action. Once committed, that value seeds the run.

Configuration randomization must not accidentally consume the
simulation's future random sequence. Lab should establish/reset the run
RNG at the run boundary.

## 21.1 Automatic run records

A run record captures reproducible initial conditions rather than
arbitrary live save-state serialization.

When Play begins a new run, Lab automatically records:

-   RNG seed;
-   recorded Run Configuration parameter IDs and values;
-   relevant Lab simulation settings such as fixed timestep where needed
    for reproduction;
-   Experiment version;
-   Lab version;
-   Template/provenance information where useful;
-   creation/start timestamp.

The exact file format remains an implementation decision.

Lab does not serialize arbitrary Experiment tables, functions, object
graphs, callbacks, shaders, or mid-run state.

## 21.2 Measurements and Observations

Lab provides generic measurement/tracking infrastructure so Experiments
can expose meaningful values for observation and recording.

A measurement is registered by the Experiment with a stable ID,
human-readable metadata such as label/unit where useful, and a sampling
callback/delegate. The callback model is preferred over a
property-name-only mechanism because measurements may be derived
quantities rather than direct entity fields.

Conceptually:

``` lua
lab.measurements.register({
    id = "total_energy",
    label = "Total System Energy",
    unit = "J",
    interval = 0.1,
    sample = experiment.getTotalEnergy
})
```

The exact API names remain an implementation detail.

Lab samples registered measurements against **simulation time**, not
render frames or wall-clock time. Changing simulation speed therefore
changes how quickly samples are produced in real time without changing
their simulation-time meaning.

Lab provides a sensible default sampling interval. Individual
measurements may override it when higher or lower temporal resolution is
appropriate.

Measurements may refer to a particular entity, but the measurement
abstraction is not limited to entity properties. An Experiment may
expose system-wide or derived values such as total energy, average flock
speed, distance between two objects, or any other meaningful numeric
observation.

Recorded samples become part of the run's experimental data.
Conceptually a run record therefore contains:

``` text
Run
├── Metadata
│   ├── versions
│   ├── timestamp
│   └── completion information
├── Configuration
│   ├── seed
│   └── recorded Run Configuration values
└── Measurements
    ├── measurement A: (simulation time, value) samples
    ├── measurement B: (simulation time, value) samples
    └── ...
```

Discrete events/results such as "body reached finish at 3.217 s" may
eventually deserve a separate first-class observation abstraction. That
distinction is deliberately left as a roadmap/design possibility rather
than forcing event-like data through periodic sampling in the initial
implementation.

## 21.3 Restoration

Load Run and Import Run feed the same restoration mechanism.

Lab:

1.  validates/reads the run record;
2.  restores the recorded seed;
3.  matches stable parameter IDs to currently registered run parameters;
4.  updates recognized controls;
5.  routes those changes through their normal callbacks;
6.  reports or visibly indicates values that could not be restored
    because the Experiment/version has changed.

A historical record should not fail wholesale merely because a newer
Experiment version no longer recognizes one old parameter.

## 21.4 Storage

Native builds automatically persist run history in the
platform-appropriate writable application-data/save location. Lab
maintains a logical structure beneath that storage, conceptually:

``` text
CCL/
└── experiments/
    └── <experiment>/
        └── runs/
```

The Data tab may expose an **Open Data Folder** action on native
platforms.

Web builds deliberately keep automatic run history **session/page-memory
only**. Creative Lab does not require persistent browser storage such as
IndexedDB for run history. Reloading/leaving the page may discard that
history.

Users who want persistence on the web explicitly Export a run to a file
and may Import it in a later session.

No server-side run storage is required for GitHub Pages or iframe-hosted
Experiments.

# 22. Simulation Timing

Lab owns real time, simulation time, scheduling, time scaling, and
simulation control.

Experiment owns what advancing its simulation means.

## 19.1 Fixed timestep

The default simulation model is a fixed timestep.

LÖVE supplies variable real frame time to Lab. Lab accumulates elapsed
real time and invokes Experiment in fixed simulation steps.

Conceptually:

``` lua
fixed_dt = 1 / 60
accumulator = 0

function love.update(real_dt)
    if lab.running then
        accumulator = accumulator + real_dt * lab.time_scale
    end

    while accumulator >= fixed_dt do
        experiment.update(state, fixed_dt)
        lab.simulation_time = lab.simulation_time + fixed_dt
        accumulator = accumulator - fixed_dt
    end
end
```

The exact default step value is configuration/implementation detail;
`1/60` is illustrative.

Experiment therefore does not normally receive LÖVE's raw variable frame
`dt`.

## 19.2 Rendering is independent

Rendering occurs according to LÖVE's render loop and reads the latest
available Experiment state.

``` text
LÖVE real time
      │
      ▼
Lab timing
      │
      ├── fixed simulation steps ──► Experiment.update(state, dt)
      │
      └── render frames ───────────► Lab rendering
```

Simulation update frequency, render frequency, and entity lifetime are
separate concepts.

## 19.3 Play, pause, step and speed

**Play** allows real elapsed time to accumulate into simulation work.

**Pause** stops simulation-time advancement. Rendering, camera
interaction, UI, inspection, and other Lab interaction may continue.

**Step** performs exactly one fixed simulation update while
paused/ready.

**Simulation speed** scales how quickly real time becomes simulation
time. It does not enlarge Experiment's physics timestep.

For example, at 10× speed Lab performs more fixed simulation steps per
real second rather than passing a timestep ten times larger.

This preserves predictable simulation behaviour and is particularly
appropriate for numerical experiments such as orbital mechanics.

## 19.4 Backlog protection

Lab should prevent an unbounded simulation-update backlog after a long
frame or stall. A maximum catch-up policy or equivalent safeguard should
be chosen during implementation.

The precise cap and overflow policy are implementation details rather
than architectural commitments.

# 23. Reset Semantics

Lab owns the Reset control and simulation clock. Experiment owns what
restoring its initial conditions means.

Experiment provides a reset callback/capability during initialization:

``` lua
lab.set_experiment({
    update = function(dt)
        update(state, dt)
    end,

    reset = function()
        reset_state(state)
    end
})
```

Exact API naming is illustrative.

Reset flow:

``` text
Reset pressed
      ↓
Lab stops simulation advancement
      ↓
Lab invokes Experiment reset
      ↓
Experiment restores its initial conditions
      ↓
Lab resets simulation time to zero
      ↓
READY
```

Lab does not deep-copy arbitrary Experiment state. It cannot safely
infer Experiment copy semantics for nested tables, shared references,
functions, caches, or other domain structures.

Experiment may implement reset however it chooses. Mutating existing
registered objects back to their initial values is a useful approach
because Lab registrations remain valid automatically.

# 24. Error Handling

A runtime error that Lab can catch at an Experiment/Lab callback
boundary is terminal for that application run.

Lab should:

1.  stop normal application interaction;
2.  display a dedicated effectively modal error UI containing the error
    and traceback;
3.  prevent interaction with the Experiment or normal controls;
4.  provide a single **OK** action;
5.  close the Experiment/application when OK is pressed.

Lab does not attempt to repair Experiment state, reset the simulation,
or continue after a caught runtime failure.

Errors too catastrophic to be caught by Lab naturally fall back to the
runtime/LÖVE fatal-error behaviour.

# 25. 2D First, 3D Later

Creative Lab initially implements 2D rendering, cameras, and spatial
queries.

Lab owns the camera and coordinate transforms. Experiment speaks in
world space. The initial 2D camera has sensible fixed defaults with pan
and zoom. Experiment may override initial camera settings and may
explicitly ask Lab to frame its registered spatial contents; automatic
framing is not the default.

Future 3D remains expected but is not pre-designed.

The v0.5 architecture deliberately uses concepts that can evolve
naturally:

-   world space rather than screen-pixel domain state;
-   camera/projection owned by Lab;
-   generic spatial queries rather than click-specific domain APIs;
-   spatial capability separate from render capability;
-   2D render ordering separate from world-space depth.

Future 3D picking will naturally tend toward ray-based spatial queries.
That evolution should not require Experiment domain objects to inherit a
new engine object hierarchy.

# 26. Architectural Principles

-   Optimize for experimentation.
-   Prefer obvious code.
-   Keep the Lab generic.
-   Keep experiments expressive in their own domain vocabulary.
-   Make ownership boundaries explicit.
-   Pass dependencies explicitly where practical.
-   Allow mutation of explicitly supplied owned state; avoid hidden
    cross-boundary mutation.
-   Separate simulation advancement from rendering.
-   Keep physics/domain response Experiment-owned until reuse earns a
    Lab abstraction.
-   Treat recorded parameters as immutable run initial conditions once a
    run begins.
-   Prefer reproducible seeded randomness for simulation/procedural
    work.
-   Separate domain object lifetime from Lab registration lifetime.
-   Separate spatial representation from visual representation.
-   Keep world-space concepts out of screen-pixel domain logic.
-   Use capability callbacks to bridge Experiment-owned live state into
    generic Lab mechanisms.
-   Do not make every rendered thing an entity.
-   Do not declare an ECS merely because capabilities resemble
    components.
-   Push side effects toward the edges where doing so improves clarity.
-   Do not impose a programming paradigm unnecessarily.
-   Earn abstractions through repeated use.
-   Treat promoted shared infrastructure as production-quality code.
-   Avoid premature engine development.
-   Design for 2D now without closing the door on 3D.
-   Keep development tools optional and repositories editor-agnostic.

The Lab provides the equipment. The Experiment provides the question.

# 27. Development and Workflow Model

The development, Git, issue-tracking, PR, CI, release,
workflow-inventory, and dependency-sync decisions established in v0.4
remain unchanged.

Key retained rules include:

-   commit = checkpoint;
-   branch = ongoing work;
-   pull request = complete proposed change;
-   `main` = trusted stable state;
-   release = published version;
-   Lab/Template PRs select exactly one `release:none`, `release:patch`,
    `release:minor`, or `release:major` label;
-   PR descriptions contain Summary, Release Notes, and Migration;
-   release automation materializes reviewed release documentation
    rather than inventing it;
-   Template validation performs a real clean Experiment bootstrap;
-   Sync Lab is manually triggered, same-major only, and uses shared
    locally runnable tooling where practical.

# 28. Initial Development Strategy

The architecture is close to implementation-ready, but several concrete
pre-code topics may still be discussed where they could materially
affect the first implementation. These should remain focused rather than
reopening broad speculative engine design.

Once those remaining topics are settled, the intended development
sequence is:

1.  Create the Creative Lab repository and establish its Semantic
    Versioning and release metadata.
2.  Implement the minimum viable LÖVE runtime and Ready/Running
    simulation lifecycle.
3.  Implement fixed-step timing with Play, Pause, Step, Restart/new-run
    orchestration, and speed control.
4.  Implement the initial 2D camera and screen/world coordinate
    transforms.
5.  Implement the sidebar shell, typed control registration, Run
    Configuration/Live Controls, and bottom control bar.
6.  Implement seeded Lab randomness and minimal automatic run-record
    infrastructure.
7.  Implement optional Experiment-defined run completion and generic
    simulation-time measurement sampling.
8.  Implement run-data inspection and CSV export for recorded
    measurements.
9.  Implement the minimal entity registry and capability mechanism.
10. Implement simple spatial queries/picking, basic contact detection,
    and retained primitive rendering.
11. Implement the initial audio facility.
12. Create the Experiment Template with `experiment.lua`,
    `simulation.lua`, `controls.lua`, optional `definitions.lua`,
    `shaders/`, TOML manifest, dependency tooling, and inherited GitHub
    Actions.
13. Create the first representative experiment, with orbital mechanics
    remaining a suitable architecture exercise.
14. Observe friction, missing abstractions, and unnecessary
    abstractions.
15. Promote genuinely reusable concepts into Lab with documentation and
    tests.
16. Create another experiment and repeat.

# 29. Remaining Open Questions

The following remain deliberately open implementation/API questions:

-   exact public Lab API and module names;
-   exact entity capability table shapes and naming;
-   exact primitive/render-description representation;
-   exact initial 2D camera API;
-   exact typed UI widget registration names and metadata naming;
-   detailed input press/hold/modifier semantics;
-   contact-event dispatch details and contact-info shape;
-   fixed-step default/configuration and backlog-overflow policy;
-   exact completion callback/result shape and completion-check
    ordering;
-   exact measurement registration API, default sampling interval, and
    supported initial value types;
-   exact transition/action from an existing run back to editable
    configuration for a genuinely new run;
-   render interpolation, if real experiments demonstrate a need;
-   shader API details;
-   audio API details and pause semantics for simulation-bound audio;
-   run-record file format, naming, retention, and compatibility policy;
-   browser-build tooling;
-   Linux distribution format;
-   exact version filenames/script names;
-   eventual concrete 3D architecture.

Additional focused design topics may be resolved before implementation
where they materially affect the first code. Otherwise, API details
should be tested against real code and real Experiments.

The following are specifically **not** planned without demonstrated
need:

-   a full ECS;
-   a physics engine or generic collision-response system;
-   spatial trees/BVH/quadtree/octree infrastructure;
-   entity factories/managers/pooling;
-   elaborate lifecycle hooks;
-   automatic deep-copy/reset infrastructure;
-   formal read-only/writable state-context hierarchies;
-   arbitrary mid-run save-state serialization or undo architecture;
-   persistent/server-side browser run storage;
-   built-in charting/graphing in the first working Lab versions;
-   speculative 3D engine abstractions.

# 30. Non-Goals --- For Now

-   Building a general-purpose game engine.
-   Designing a complete 3D renderer before a real experiment needs one.
-   Creating a universal plugin/package ecosystem.
-   Enforcing production-quality structure or tests inside exploratory
    experiments.
-   Abstracting every LÖVE API pre-emptively.
-   Solving hypothetical future requirements without evidence from
    experiments.
-   Requiring Experiment domain objects to inherit Lab entity types.
-   Turning capability registration into an ECS by default.

# 31. Roadmap Beyond the Initial Working Versions

The initial Lab should make experimental data useful without requiring
an internal plotting system. Recorded measurements can be inspected and
exported to CSV for analysis in external tools.

A later roadmap item is **built-in charting and graphing of tracked
measurements**. Once real Experiments demonstrate the required plotting
workflows, Lab may add generic visualisation of recorded/live
measurement series, such as value-over-time graphs or comparative
charts.

Chart configuration, plotting types, axes, series grouping, styling, and
live-versus-recorded presentation are intentionally deferred until the
measurement system has been exercised by real Experiments.

# 32. Definition of Success

Creative Coding Lab succeeds when an idea such as "what happens if I
simulate several bodies under Newtonian gravity?" quickly becomes work
on gravity, integration, visualisation, and experimentation rather than
an evening spent rebuilding window setup, input handling, camera
controls, sliders, timing, picking, and debugging infrastructure.

A successful experiment should also leave behind a comprehensible record
of what was being explored and what was learned.

# 33. Current Architectural Summary

  ---------------------------------------------------------------------
  Area                               v0.6 direction
  ---------------------------------- ----------------------------------
  Runtime                            Independent Creative Lab
                                     repository; reusable
                                     runtime/presentation/interaction
                                     infrastructure rather than a
                                     game/physics engine.

  Experiments                        Independent public LÖVE
                                     repositories grouped locally
                                     beneath an `experiments/`
                                     workspace folder.

  Template                           Independent SemVer'd GitHub
                                     template with `experiment.lua`,
                                     `simulation.lua`, `controls.lua`,
                                     optional `definitions.lua`, and
                                     `shaders/`.

  Dependencies                       Lab fetched into `vendor/lab`;
                                     committed metadata authoritative;
                                     same-major Sync Lab supported.

  Versioning                         Lab/Template use independent
                                     SemVer; Experiments use
                                     completed-Case/progress numbering.

  Documentation                      `README.md` plus living
                                     `EXPERIMENT.md` with Case history.

  Testing                            Busted standard; expected for
                                     Lab/Template; optional but
                                     supported for Experiments.

  Git                                Stable `main` for Lab/Template
                                     with short-lived branches and PR
                                     validation; Experiments more
                                     relaxed.

  State                              Experiment owns live domain state;
                                     `definitions.lua` may construct
                                     plain domain data.

  Simulation                         `simulation.lua` holds
                                     mathematical/model logic; pure
                                     functions preferred where natural;
                                     physics remains Experiment-owned.

  Mutation                           Mutation of explicitly passed
                                     owned state is allowed; hidden
                                     cross-boundary mutation is
                                     avoided.

  Lifecycle                          Initialize once before Ready; Play
                                     begins/continues simulation; Pause
                                     stops simulation; Step advances
                                     one fixed step; Restart reproduces
                                     the current run.

  Runs                               Run Configuration is snapshotted
                                     and locked at run start; Session
                                     may contain multiple runs; new-run
                                     transition still to be finalized.

  Timing                             Lab-owned fixed simulation
                                     timestep, accumulator, simulation
                                     clock, and speed scaling;
                                     rendering independent.

  Randomness                         Lab-owned seeded pseudorandom
                                     generator; seed is a recorded Run
                                     Configuration parameter.

  Run data                           Automatic reproducibility records;
                                     native history persists in app
                                     data, web history is
                                     memory/session-only; Import/Export
                                     provide portability.

  Entities                           Experiment owns domain objects;
                                     Lab owns registrations and generic
                                     capabilities.

  Contact                            Lab detects/tracks generic
                                     enter/stay/exit contact;
                                     Experiment owns response/physics.

  Input                              Lab owns raw input/default
                                     bindings/UI priority; Experiment
                                     may register and override
                                     effective bindings.

  Spatial                            Generic world-space geometry/query
                                     capability; multiple-hit capable;
                                     future ray-based 3D evolution.

  Rendering                          Lab renders Experiment-described
                                     world-space primitives; retained
                                     entities plus immediate procedural
                                     drawing.

  Shaders                            Experiment owns shader
                                     code/parameters; Lab owns generic
                                     pipeline application/management.

  Camera                             Lab-owned 2D camera with pan/zoom
                                     defaults; Experiment may override
                                     initial settings or explicitly
                                     request framing.

  Audio                              Lab owns generic audio
                                     mechanism/master controls;
                                     simulation-world audio follows
                                     simulation speed by default with
                                     opt-out.

  UI                                 Sidebar tabs: Instrumentation,
                                     Debugging, Controls, Data;
                                     persistent bottom simulation/audio
                                     bar.

  Controls                           Explicit typed registration in
                                     `controls.lua`; Run Configuration
                                     versus Live Controls; optional
                                     tooltips and opt-in user-triggered
                                     randomization.

  Errors                             Caught runtime errors are
                                     terminal: modal error/traceback,
                                     OK closes application.

  3D                                 Architecture leaves a path open;
                                     concrete 3D system deferred until
                                     a real experiment needs it.

  Development tools                  Editor-agnostic; VS Code workspace
                                     is optional convenience.
  ---------------------------------------------------------------------

# 34. Design Agenda After v0.7

v0.6 records the substantial pre-implementation decisions made after
v0.5.

A small number of focused topics may still be worth settling before code
where they could materially shape the first implementation. The
architecture should not, however, return to broad speculative subsystem
design.

After those focused decisions, implementation should become the primary
source of architectural evidence. Exact API names, control shapes,
rendering primitives, run-record format, and similar details should be
allowed to emerge from working Lab/Template/Experiment code.

# 35. Document Status

Version 0.7 extends the v0.6 architecture with the remaining
pre-implementation decisions around run completion and experimental
observations.

It establishes:

-   optional Experiment-defined natural completion conditions;
-   a Completed run state that stops simulation advancement without
    closing the application;
-   room for stable completion reasons/results in run metadata;
-   generic Lab-owned measurement/tracking infrastructure;
-   callback/delegate-based measurements so derived values are
    first-class;
-   simulation-time-based sampling with a Lab default interval and
    optional per-measurement overrides;
-   recorded measurements as part of run experimental data;
-   measurement visibility in the Data tab;
-   CSV export as the initial interoperable analysis path;
-   built-in charting/graphing as a later roadmap item rather than a
    requirement for the first working versions.

All architectural decisions recorded in v0.6 remain in force unless
explicitly refined here. The next step is to review this document as the
consolidated source design, resolve any contradictions or stale wording
found during that review, and then derive audience-specific
technical/architecture and user-facing documentation before
implementation begins.

This remains a living source document. It is intentionally broad enough
to support those later derived documents while preserving the
distinction between settled architecture, implementation details, open
questions, and roadmap items.
