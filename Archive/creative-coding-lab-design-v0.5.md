---
title: Creative Coding Lab
---

*Design Document v0.5 · Living Design Document*

| Field | Value |
|---|---|
| Status | Living design document |
| Version | 0.5 |
| Initial platform | Lua + LÖVE |
| Project type | Personal creative-coding platform and collection of independent experiments |

# 1. Overview

Creative Coding Lab is a lightweight creative-coding environment for building visual, interactive programming experiments. It exists to make mathematical, simulation, procedural-generation, graphics, and other computational ideas easy to explore through code.

The Lab is not intended to become a general-purpose game engine. Its purpose is to remove repetitive infrastructure so that a new experiment can get quickly to the question being explored.

Each experiment is an independent project and portfolio item. The shared Creative Lab runtime supplies reusable equipment such as application lifecycle integration, input, timing, rendering infrastructure, cameras, experiment UI, entity inspection, and debugging facilities.

Core summary:

> A lightweight LÖVE-based creative coding runtime, paired with a standard experiment template, that makes it easy to create independent visual programming experiments while keeping shared infrastructure out of the way of the ideas being explored.

# 2. Goals

- Make it quick to move from an idea or question to a running visual experiment.
- Learn programming, mathematics, graphics, simulation, and related concepts by implementing and manipulating them.
- Keep shared infrastructure reusable without allowing the Lab to become an engine-building project for its own sake.
- Give each experiment enough independence to stand as its own public repository and portfolio item.
- Use clean, modern engineering practices for shared infrastructure while allowing exploratory experiment code to remain flexible.
- Document the premise, setup, intended learning, and eventual findings of each experiment.
- Allow the platform to evolve in response to real experiments rather than speculative requirements.

# 3. Core Philosophy

## 3.1 Start with a question

Experiments should normally begin with something to explore rather than a product requirement: What happens if several bodies interact under Newtonian gravity? What is a Fourier transform doing visually? How do boids produce flocking behaviour?

## 3.2 Learn by building

The objective is not merely to reproduce an established result. The implementation, manipulation of parameters, visualisation, and observation are part of the learning process.

## 3.3 The Lab provides the equipment

The Lab should understand generic concepts such as rendering, cameras, input, timing, UI, spatial queries, entity registration, and debugging. It should not understand planets, boids, Fourier transforms, terrain, or other experiment-specific domains.

## 3.4 Earn abstractions

Reusable concepts should normally begin inside an experiment. If multiple experiments demonstrate that a concept is genuinely general, it may be promoted into the Lab, cleaned up, documented, and tested.

## 3.5 Prefer obvious code

The project may use data-oriented, functional, object-oriented, or other approaches where they make the code clearer. No programming paradigm is imposed as an architectural goal.

## 3.6 Architecture serves experimentation

The architecture serves the experiments—not the other way around. Infrastructure should reduce friction, not require experiments to conform to unnecessary ceremony.

# 4. Technology

| Technology | Role |
|---|---|
| Lua | Primary language for the Lab and experiments. |
| LÖVE | Initial application, graphics, input, and runtime framework. |
| POSIX shell | Bootstrap, dependency-fetching, and development scripts. |
| Markdown | Project documentation and experiment lab-notebook documentation. |
| Git | Source control and version/tag foundation. |
| GitHub | Repository hosting, releases, distribution, and public portfolio presentation. |

VS Code is the intended local editor and may be used with a multi-root workspace for convenience, but it is not a Creative Lab technology dependency. The repositories must remain editor-agnostic.

# 5. High-Level Architecture

```text
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

```text
Experiment coordinator
│
├── Experiment-owned state
├── Simulation / domain logic
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

The central boundary is ownership: the Experiment owns domain meaning and state; the Lab owns generic mechanisms.

# 6. Lab Runtime

Creative Lab is shared infrastructure used by independent experiments. Its API should remain small and should grow in response to demonstrated needs.

## 6.1 Intended responsibilities

- LÖVE lifecycle integration and application setup.
- Real-time and simulation-time scheduling.
- Play, pause, single-step, reset orchestration, and simulation speed.
- Input capture and generic coordinate transformation.
- 2D rendering infrastructure and drawing helpers.
- Camera facilities.
- Generic entity registration and inspection.
- Generic spatial queries and picking.
- A standard collapsible sidebar and UI controls.
- Lab-provided debugging and instrumentation facilities.
- Generic visualisation utilities that have demonstrated reuse across experiments.

## 6.2 Non-responsibilities

- Experiment-specific domain models or terminology.
- Experiment-specific simulation rules.
- Owning Experiment domain state.
- Inferring the meaning of a selected domain object.
- Premature abstractions for concepts not yet shown to be reusable.
- Owning or cataloguing the collection of experiments.

# 7. Experiments

Each experiment is an independent runnable LÖVE application and an independent public Git repository. It owns its entry point, domain vocabulary, simulation logic, domain state, initial conditions, reset semantics, and the controls it exposes through the Lab's instrumentation UI.

Experiment code is exploratory by design. It may be refactored when useful, but production-style ceremony must not become a prerequisite for trying an idea.

# 8. Experiment Entry Point and Lifecycle Contract

The experiment owns `main.lua`. The Lab supplies runtime facilities rather than acting as the application itself.

A possible entry shape remains:

```lua
local Lab = require("vendor.lab.runtime")
local Experiment = require("experiment")

Lab.run(Experiment)
```

The exact public API naming remains an implementation detail, but the lifecycle semantics are now defined.

## 8.1 Initialization

Experiment initialization occurs once when the Experiment/application is loaded, before the user can press Play.

Initialization may:

- create Experiment state;
- create the initial scene/domain objects;
- register objects and their Lab-facing capabilities;
- define instrumentation;
- provide the Experiment's update and reset behaviour.

After initialization the application enters **Ready**. The initial scene is visible, simulation time is zero, and the simulation is not advancing.

Play does not initialize the Experiment.

## 8.2 Simulation lifecycle

```text
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

Lab owns the lifecycle controls and scheduling. Experiment owns the meaning of advancing or restoring its domain state.

No `onPlay()` or equivalent callback is required unless a real experiment demonstrates a need for one.

# 9. Simulation and Domain Logic

Where useful, the mathematical or simulation core should be separated from rendering, input, and other side effects.

A representative shape is:

```lua
function experiment.update(state, dt)
    local forces = calculate_forces(state.bodies)
    local velocities = calculate_velocities(state.bodies, forces, dt)
    update_bodies(state.bodies, velocities, dt)
end
```

Pure calculations should naturally return values:

```lua
local force = calculate_force(body_a, body_b)
local velocity = calculate_velocity(body, force, dt)
local position = calculate_position(body, velocity, dt)
```

A thin mutating layer may then apply those results to explicitly passed Experiment state.

Purity is a tool rather than a rule. Clarity and ease of experimentation take priority.

# 10. Experiment State

Experiment-specific data belongs with the Experiment.

The default model is a plain Lua **state table** explicitly passed to code that needs it. The architecture does not introduce formal read-only/writable context types or a framework-owned state container.

For example:

```lua
local state = {
    bodies = {},
    selected_body = nil,
    parameters = {
        gravity = 1
    }
}
```

Coordinator functions may receive the complete state table. Lower-level functions should receive narrower values or sub-state where that makes their dependencies clearer.

The Lab must not require every function to receive Lab or Experiment state. Simulation-only calculations should ideally depend on neither LÖVE nor Lab.

# 11. Ownership and Mutation Rules

| Concern | Owner |
|---|---|
| LÖVE lifecycle integration | Lab |
| Real-time and simulation scheduling | Lab |
| Camera and viewport state | Lab |
| Raw input state | Lab |
| Sidebar shell and generic widgets | Lab |
| Debugging facilities | Lab |
| Entity registration records | Lab |
| Domain objects and collections | Experiment |
| Simulation/domain rules | Experiment |
| Experiment parameters and instrumentation meaning | Experiment |
| Domain selection state | Experiment |
| Initial conditions and reset semantics | Experiment |
| Proven reusable concepts | May be promoted to Lab |

## 11.1 Mutation rule

A function may mutate state explicitly passed to it. Functions should avoid reaching out to mutate unrelated hidden external state.

This is not a requirement to mutate arguments. Pure calculations should return values normally where that is clearer.

Within an ownership boundary, mutation versus return is an implementation choice. Across the Lab/Experiment boundary, neither side should secretly mutate state owned by the other.

## 11.2 Dependency direction

Code should receive the narrowest useful dependency.

- A simulation calculation may receive bodies and `dt`.
- A coordinator may receive the complete Experiment state.
- Code that needs a Lab service may receive Lab or the relevant Lab service.
- Code that needs neither should know about neither.

# 12. Repository Architecture

The repository model remains unchanged from v0.4:

```text
creative-coding/ [local parent; not a Git repo]
├── creative-lab/ [GIT REPO]
├── creative-lab-experiment/ [GIT REPO / GitHub template]
├── orbital-mechanics/ [GIT REPO]
├── boids/ [GIT REPO]
├── fourier-transform/ [GIT REPO]
└── creative-coding.code-workspace [local convenience]
```

Repositories remain independent. Git submodules are intentionally avoided; dependency retrieval is handled by bootstrap tooling.

# 13. Lab Dependency, Versioning, Cases, Testing, Git and Release Model

The dependency, versioning, Case lifecycle, testing, Git, CI, release, and distribution decisions established through v0.4 remain in force.

In summary:

- Lab and Template are independently Semantic Versioned.
- Experiments use completed-Case/progress numbering rather than SemVer.
- `experiment.toml` records Experiment, Template, and Lab provenance.
- Sync Lab may update minor/patch releases only within the current Lab major.
- Busted is the standard Lua testing framework; Lab/Template are production-quality, Experiment tests are optional.
- Lab/Template use stable `main`, short-lived branches, PR validation, and PR-level `release:*` intent.
- Experiments may use a more relaxed Git workflow.
- Experiment releases target browser/GitHub Pages, Windows, macOS, Linux, and optionally `.love`.
- `EXPERIMENT.md` remains the living experiment notebook with Case history.

# 14. Experiment UI

Creative Lab owns a standard application UI shell while the Experiment remains visually dominant. The normal application view is the Experiment rendering area with a small menu button in the top-left corner.

The sidebar contains **Instrumentation** and **Debugging**.

## 14.1 Instrumentation

Instrumentation is rendered by Lab but defined by Experiment. Lab supplies generic widgets; Experiment defines what controls exist, what values they expose, and what changing them means.

Callbacks/functions are first-class Lua values and are the normal bridge where Lab needs Experiment-defined behaviour.

The architecture does not require a formal binding/context hierarchy.

## 14.2 Debugging and simulation controls

Lab owns generic controls including:

- Play;
- Pause;
- Step;
- Reset;
- simulation speed;
- FPS/frame timing;
- coordinate/grid overlays;
- generic inspection and debug drawing.

Controls should normally remain visible but disabled when their required capability is unavailable.

Enablement is determined by both:

1. whether Experiment provides the required capability; and
2. the current Lab lifecycle state.

For example:

- Play requires `update`;
- Pause requires only that Lab is currently advancing the simulation;
- Step requires `update` and an appropriate paused/ready state;
- Reset requires an Experiment-provided reset capability.

# 15. Entity Registration

Lab provides a generic entity registry for Experiment objects that need Lab services such as rendering, picking, or inspection.

An Experiment domain object does not inherit from a Lab base entity and does not become Lab-owned.

For example:

```lua
local earth = {
    name = "Earth",
    mass = 5.972e24,
    position = { x = 100, y = 200 },
    velocity = { x = 0, y = 10 },
    radius = 20
}
```

Experiment may register the object:

```lua
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

Lab stores a Lab-owned registration record containing an opaque reference to the Experiment-owned object plus generic Lab-facing capabilities.

Registration does not transfer ownership.

## 15.1 Capability model

Capabilities describe what generic Lab services can do with an object without teaching Lab its domain meaning.

Likely capability families include:

```text
Entity registration
├── spatial
├── render
├── inspect
└── future capabilities earned by real use
```

This is component-shaped but is not declared to be an Entity Component System.

Callbacks/accessors read the live Experiment object, so Experiment does not need to duplicate positional or visual state into Lab or notify Lab every time a value changes.

# 16. Entity Lifetime

Experiment owns object/domain lifetime. Lab owns only registration lifetime.

Typical flow:

```text
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

`unregister` does not destroy the object. It means only that Lab stops providing registered services for it.

When an Experiment unloads or is reinitialized, Lab automatically clears all registrations belonging to that Experiment. Experiment should not need to manually unregister every object during whole-Experiment teardown.

No entity manager, factory, pooling system, destruction event system, or formal ECS is introduced without demonstrated need.

# 17. Input, Coordinate Spaces, Selection and Spatial Queries

Lab owns raw input and camera transforms.

Experiment should not need to know screen pixels for world interaction.

For a pointer interaction:

```text
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

Selection is Experiment-owned domain state. For example, an orbital experiment may set:

```lua
state.selected_body = hit.entity
```

Lab may provide generic inspection UI for the selected/returned entity, but Lab does not decide that the object means "selected planet."

## 17.1 Spatial capability

The preferred spatial model describes generic geometry rather than delegating all picking to an arbitrary Experiment `hit_test` function.

For example, an Experiment can expose a circle's current position and radius. Lab then performs the generic point/circle or ray/shape intersection.

This allows Lab to reason about geometry and leaves room for future spatial acceleration without changing the Experiment's domain model.

Custom hit-test callbacks may remain an escape hatch for unusual cases.

## 17.2 Query results and overlap

The spatial API should support multiple hits even if a convenience `pick()` operation returns one.

A generic result may contain:

```lua
{
    entity = object,
    position = world_position,
    distance = distance
}
```

The architecture must not define spatial identity as "first registered wins."

For 2D, render ordering may sometimes provide a useful interaction tie-breaker. For future 3D, ray distance will normally be the more meaningful ordering.

2D render order and 3D world Z are separate concepts and must not be conflated.

# 18. Rendering Contract

Experiment owns world-space geometry/procedural definitions and appearance intent.

Lab owns camera, projection, viewport, graphics-resource management, and final screen-space rendering.

World geometry is expressed in world units:

```lua
position = { x = 100, y = 200 }
radius = 10
```

The Experiment does not convert that radius to pixels. Lab's camera/projection determines its screen representation.

## 18.1 Retained registered rendering

Registered entities are persistent renderables.

Experiment registers the object once. On each render frame, Lab reads the object's current render capability and renders the resulting representation. The Experiment does not resubmit the object every simulation tick.

This is the normal model for persistent domain objects such as planets or agents.

## 18.2 Immediate/procedural rendering

Lab should also provide a transient drawing path for geometry that naturally exists only for the current frame, such as:

- generated curves;
- graphs;
- fields;
- trails;
- mathematical visualisations;
- debug geometry.

Immediate drawing does not require creating thousands of persistent entities merely to issue temporary primitives.

Therefore not everything rendered by Lab is an entity.

## 18.3 Spatial shape versus render shape

Spatial and render capabilities are deliberately separate.

An object may:

- render with complex geometry but use a simple picking shape;
- be visible but not selectable;
- be selectable/inspectable without having a normal visible representation.

## 18.4 Procedural geometry

The initial renderer should support straightforward primitives such as circles, rectangles, lines, and polygons. Experiment may compute more complex geometry and describe it using generic Lab-understood representations.

Exact primitive/API names are deferred to implementation.

## 18.5 Shaders

LÖVE shader support is compatible with this architecture.

Experiment should describe shader intent or provide shader definitions through the render capability rather than directly binding global LÖVE graphics state as the normal path.

Lab may compile/manage/bind shaders and provide standard uniforms such as simulation time, viewport size, and camera information. Experiment supplies experiment-specific shader parameters.

The shader API is intentionally not designed in detail yet.

# 19. Simulation Timing

Lab owns real time, simulation time, scheduling, time scaling, and simulation control.

Experiment owns what advancing its simulation means.

## 19.1 Fixed timestep

The default simulation model is a fixed timestep.

LÖVE supplies variable real frame time to Lab. Lab accumulates elapsed real time and invokes Experiment in fixed simulation steps.

Conceptually:

```lua
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

The exact default step value is configuration/implementation detail; `1/60` is illustrative.

Experiment therefore does not normally receive LÖVE's raw variable frame `dt`.

## 19.2 Rendering is independent

Rendering occurs according to LÖVE's render loop and reads the latest available Experiment state.

```text
LÖVE real time
      │
      ▼
Lab timing
      │
      ├── fixed simulation steps ──► Experiment.update(state, dt)
      │
      └── render frames ───────────► Lab rendering
```

Simulation update frequency, render frequency, and entity lifetime are separate concepts.

## 19.3 Play, pause, step and speed

**Play** allows real elapsed time to accumulate into simulation work.

**Pause** stops simulation-time advancement. Rendering, camera interaction, UI, inspection, and other Lab interaction may continue.

**Step** performs exactly one fixed simulation update while paused/ready.

**Simulation speed** scales how quickly real time becomes simulation time. It does not enlarge Experiment's physics timestep.

For example, at 10× speed Lab performs more fixed simulation steps per real second rather than passing a timestep ten times larger.

This preserves predictable simulation behaviour and is particularly appropriate for numerical experiments such as orbital mechanics.

## 19.4 Backlog protection

Lab should prevent an unbounded simulation-update backlog after a long frame or stall. A maximum catch-up policy or equivalent safeguard should be chosen during implementation.

The precise cap and overflow policy are implementation details rather than architectural commitments.

# 20. Reset Semantics

Lab owns the Reset control and simulation clock. Experiment owns what restoring its initial conditions means.

Experiment provides a reset callback/capability during initialization:

```lua
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

```text
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

Lab does not deep-copy arbitrary Experiment state. It cannot safely infer Experiment copy semantics for nested tables, shared references, functions, caches, or other domain structures.

Experiment may implement reset however it chooses. Mutating existing registered objects back to their initial values is a useful approach because Lab registrations remain valid automatically.

# 21. Error Handling Direction

Detailed error/debug UI remains an implementation topic, but the ownership direction is clear.

Lab is responsible for invoking Experiment lifecycle and capability callbacks. A runtime Experiment error should not require Lab to reinterpret domain state.

A likely implementation is for Lab to catch callback failures at its boundary, stop/pause simulation advancement, and expose the failure through Lab debugging facilities rather than allowing repeated failing updates to continue.

The exact recovery and presentation model should be determined during implementation.

# 22. 2D First, 3D Later

Creative Lab initially implements 2D rendering, cameras, and spatial queries.

Future 3D remains expected but is not pre-designed.

The v0.5 architecture deliberately uses concepts that can evolve naturally:

- world space rather than screen-pixel domain state;
- camera/projection owned by Lab;
- generic spatial queries rather than click-specific domain APIs;
- spatial capability separate from render capability;
- 2D render ordering separate from world-space depth.

Future 3D picking will naturally tend toward ray-based spatial queries. That evolution should not require Experiment domain objects to inherit a new engine object hierarchy.

# 23. Architectural Principles

- Optimize for experimentation.
- Prefer obvious code.
- Keep the Lab generic.
- Keep experiments expressive in their own domain vocabulary.
- Make ownership boundaries explicit.
- Pass dependencies explicitly where practical.
- Allow mutation of explicitly supplied owned state; avoid hidden cross-boundary mutation.
- Separate simulation advancement from rendering.
- Separate domain object lifetime from Lab registration lifetime.
- Separate spatial representation from visual representation.
- Keep world-space concepts out of screen-pixel domain logic.
- Use capability callbacks to bridge Experiment-owned live state into generic Lab mechanisms.
- Do not make every rendered thing an entity.
- Do not declare an ECS merely because capabilities resemble components.
- Push side effects toward the edges where doing so improves clarity.
- Do not impose a programming paradigm unnecessarily.
- Earn abstractions through repeated use.
- Treat promoted shared infrastructure as production-quality code.
- Avoid premature engine development.
- Design for 2D now without closing the door on 3D.
- Keep development tools optional and repositories editor-agnostic.

The Lab provides the equipment. The Experiment provides the question.

# 24. Development and Workflow Model

The development, Git, issue-tracking, PR, CI, release, workflow-inventory, and dependency-sync decisions established in v0.4 remain unchanged.

Key retained rules include:

- commit = checkpoint;
- branch = ongoing work;
- pull request = complete proposed change;
- `main` = trusted stable state;
- release = published version;
- Lab/Template PRs select exactly one `release:none`, `release:patch`, `release:minor`, or `release:major` label;
- PR descriptions contain Summary, Release Notes, and Migration;
- release automation materializes reviewed release documentation rather than inventing it;
- Template validation performs a real clean Experiment bootstrap;
- Sync Lab is manually triggered, same-major only, and uses shared locally runnable tooling where practical.

# 25. Initial Development Strategy

With the state/data-flow architecture now settled sufficiently for implementation, the next development sequence is:

1. Create the Creative Lab repository and establish its Semantic Versioning and release metadata.
2. Implement the minimum viable LÖVE runtime and Ready/Running simulation lifecycle.
3. Implement fixed-step timing with Play, Pause, Step, Reset orchestration, and speed control.
4. Implement the initial 2D camera and screen/world coordinate transforms.
5. Implement the minimal entity registry and capability mechanism.
6. Implement simple spatial queries/picking and retained primitive rendering.
7. Implement the sidebar shell with Instrumentation and Debugging.
8. Create the Experiment Template, TOML manifest, dependency tooling, and inherited GitHub Actions.
9. Create the first representative experiment, with orbital mechanics remaining a suitable architecture exercise.
10. Observe friction, missing abstractions, and unnecessary abstractions.
11. Promote genuinely reusable concepts into Lab with documentation and tests.
12. Create another experiment and repeat.

# 26. Remaining Open Questions

State Management & Data Flow is no longer a principal unresolved architectural topic after v0.5.

The following remain deliberately open implementation/API questions:

- exact public Lab API and module names;
- exact entity capability table shapes and naming;
- exact primitive/render-description representation;
- exact initial 2D camera API;
- exact UI widget/binding API;
- detailed input API beyond the established ownership/coordinate boundary;
- detailed callback error presentation and recovery;
- fixed-step default/configuration and backlog-overflow policy;
- render interpolation, if real experiments demonstrate a need;
- shader API details;
- browser-build tooling;
- Linux distribution format;
- exact version filenames/script names;
- eventual concrete 3D architecture.

These are not reasons to delay the initial implementation. They should be resolved against real code and real experiments.

The following are specifically **not** planned without demonstrated need:

- a full ECS;
- spatial trees/BVH/quadtree/octree infrastructure;
- entity factories/managers/pooling;
- elaborate lifecycle hooks;
- automatic deep-copy/reset infrastructure;
- formal read-only/writable state-context hierarchies;
- serialization or undo architecture;
- speculative 3D engine abstractions.

# 27. Non-Goals — For Now

- Building a general-purpose game engine.
- Designing a complete 3D renderer before a real experiment needs one.
- Creating a universal plugin/package ecosystem.
- Enforcing production-quality structure or tests inside exploratory experiments.
- Abstracting every LÖVE API pre-emptively.
- Solving hypothetical future requirements without evidence from experiments.
- Requiring Experiment domain objects to inherit Lab entity types.
- Turning capability registration into an ECS by default.

# 28. Definition of Success

Creative Coding Lab succeeds when an idea such as “what happens if I simulate several bodies under Newtonian gravity?” quickly becomes work on gravity, integration, visualisation, and experimentation rather than an evening spent rebuilding window setup, input handling, camera controls, sliders, timing, picking, and debugging infrastructure.

A successful experiment should also leave behind a comprehensible record of what was being explored and what was learned.

# 29. Current Architectural Summary

| Area | v0.5 direction |
|---|---|
| Runtime | Independent Creative Lab repository; reusable shared infrastructure. |
| Experiments | Independent public LÖVE repositories with a structured Case lifecycle. |
| Template | Independent SemVer'd GitHub template; production-quality infrastructure and experiment tooling. |
| Dependencies | Lab fetched into `vendor/lab`; committed metadata authoritative; same-major Sync Lab supported. |
| Versioning | Lab/Template use independent SemVer; Experiments use completed-Case/progress numbering. |
| Documentation | `README.md` plus living `EXPERIMENT.md` with Case history. |
| Testing | Busted standard; expected for Lab/Template; optional but supported for Experiments. |
| Git | Stable `main` for Lab/Template with short-lived branches and PR validation; Experiments more relaxed. |
| Releases | PR-label-driven Lab/Template releases; manual Experiment progress and Case-completion releases. |
| Distribution | Experiment releases target web/GitHub Pages, Windows, macOS, Linux, and optionally `.love`. |
| State | Plain Experiment-owned state passed explicitly; no framework context hierarchy. |
| Mutation | Mutation of explicitly passed owned state is allowed; hidden cross-boundary mutation is avoided. |
| Lifecycle | Initialize once before Ready; Play advances time; Pause stops simulation; Step advances one fixed step; Reset delegates restoration to Experiment. |
| Timing | Lab-owned fixed simulation timestep, accumulator, simulation clock, and speed scaling; rendering independent. |
| Entities | Experiment owns domain objects; Lab owns registrations and generic capabilities. |
| Lifetime | Register/unregister is independent of object creation/destruction; Lab clears registrations on Experiment unload. |
| Input | Lab owns raw input and coordinate transforms; Experiment owns domain interpretation. |
| Selection | Domain selection is Experiment state; Lab provides generic query/inspection mechanisms. |
| Spatial | Generic world-space geometry/query capability; multiple-hit capable; future ray-based 3D evolution. |
| Rendering | Lab renders Experiment-described world-space primitives; retained entities plus immediate procedural drawing. |
| Camera | Lab-owned; Experiment world logic does not depend on screen pixels. |
| UI | Lab-owned sidebar with Experiment-defined Instrumentation and Lab-defined Debugging. |
| 3D | Architecture leaves a path open; concrete 3D system deferred until a real experiment needs it. |
| Development tools | Editor-agnostic; VS Code workspace is optional convenience. |

# 30. Design Agenda for v0.6

v0.6 should be driven primarily by **implementation evidence rather than another broad architecture pass**.

The first runtime and representative Experiment should test the decisions recorded in v0.5. Questions that emerge from actual code—particularly exact capability shapes, rendering primitives, camera API, input routing, error presentation, and timing configuration—can then be promoted into the next design iteration where they prove architecturally significant.

A useful v0.6 discussion should therefore begin from observed friction or a concrete implementation decision rather than selecting another subsystem to design speculatively.

# 31. Document Status

Version 0.5 resolves the State Management & Data Flow agenda left open by v0.4.

It establishes:

- explicit Experiment-owned state and ownership boundaries;
- a pragmatic mutation rule rather than mandatory immutability;
- initialization/Ready/Running lifecycle semantics;
- Experiment-defined reset behaviour;
- Lab-owned simulation scheduling with a fixed timestep;
- independent rendering and simulation loops;
- generic entity registration without ownership transfer or inheritance;
- capability callbacks/accessors over live Experiment objects;
- Experiment-owned object lifetime and Lab-owned registration lifetime;
- Lab-owned input transformation and generic spatial querying;
- Experiment-owned selection/domain meaning;
- world-space rendering descriptions interpreted by Lab;
- retained rendering for registered entities plus immediate/procedural drawing;
- separation of spatial and visual representations;
- a clean evolution path toward shaders and eventual 3D without pre-building either system.

The architecture is now considered sufficiently defined to begin implementing the initial runtime and representative Experiment. Remaining API details should be allowed to emerge from that implementation.

This remains a living document. Its purpose is to record current intent and decisions while allowing implementation experience and real experiments to challenge and improve the architecture.
