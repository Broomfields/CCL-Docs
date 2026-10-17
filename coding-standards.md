# Creative Coding Lab: Coding Standards

| Field | Value |
|---|---|
| Audience | Developers writing or reviewing Lab, Template, and experiment code |
| Source | Design document v0.8 |
| Status | Coding guidance derived from the intended design |
| Companion | [Developer Architecture and Reference](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/developer-architecture.md) |

Use this document while implementing and reviewing code. Architecture and delivery contracts live in the companion reference. These standards translate existing design principles into practical rules and illustrative examples.

## Contents

- [1. Rule strength and scope](#1-rule-strength-and-scope)
- [2. Programming style and data](#2-programming-style-and-data)
- [3. State and dependencies](#3-state-and-dependencies)
- [4. Naming and comments](#4-naming-and-comments)
- [5. Functions and modules](#5-functions-and-modules)
- [6. Callbacks and Lab integration](#6-callbacks-and-lab-integration)
- [7. Tests and shared abstractions](#7-tests-and-shared-abstractions)
- [8. Tooling, commits, and review](#8-tooling-commits-and-review)

## 1. Rule strength and scope

| Level | Meaning |
|---|---|
| Requirement | Preserve an established ownership, interface, validation, or release contract. |
| Default | Start here and adapt when a clearer solution fits the responsibility. |
| Judgement | Choose using the local problem and observed needs. |

| Code area | Expectation |
|---|---|
| Lab | Production-quality shared behaviour, clear interfaces, documentation, and appropriate tests. |
| Template | Reliable inherited tooling and a validated starting environment. |
| Experiment | Clear domain intent and compatible Lab integration, with flexible exploratory structure and optional tests. |

### Reasoning

Shared code affects several experiments. Exploratory code serves a developing question. Distinguishing requirements from defaults keeps important boundaries reliable while allowing practical local choices.

Source: [v0.8 principles](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#2-principles-and-boundaries) and [testing policy](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#9-testing-and-validation).

## 2. Programming style and data

| Level | Rule |
|---|---|
| Default | Represent domain data with plain Lua tables and small constructors. |
| Default | Use pure functions for calculations where they express the operation clearly. |
| Judgement | Choose functional, data-oriented, or object-oriented techniques according to the responsibility. |
| Default | Follow the [Lua language reference](https://www.lua.org/manual/), [LÖVE API conventions](https://love2d.org/wiki/love), and the surrounding module's style. |
| Judgement | Add abstractions when use demonstrates a coherent shared need. |

### Language and framework references

| Reference | Use it for |
|---|---|
| [Official Lua reference manuals](https://www.lua.org/manual/) | Language syntax, semantics, tables, functions, scope, and standard libraries. Choose the manual matching the Lua runtime used by the selected LÖVE build. |
| [Official LÖVE API reference](https://love2d.org/wiki/love) | Framework modules, API names, callback signatures, and documented behaviour. Check availability against the selected LÖVE version. |
| [LÖVE callback tutorial](https://love2d.org/wiki/Tutorial:Callback_Functions) | How application callbacks such as `love.load`, `love.update`, and `love.draw` fit together. |

These references establish language and framework usage. This document supplies project-specific readability and ownership guidance. Formatting and casing choices follow the surrounding code until a project-wide convention is selected.

### Example

```lua
local function create_body(mass, position, velocity)
    return {
        mass = mass,
        position = position,
        velocity = velocity
    }
end

local function velocity_after_step(velocity, acceleration, timestep)
    return velocity + acceleration * timestep
end
```

The constructor stores the supplied references. The calculation illustrates one scalar component. Copying policy and a full vector model belong to the experiment's chosen representation.

### Reasoning

Plain data exposes the domain directly. Pure calculations make inputs and results easy to inspect and test. A constructor is useful when it clarifies creation; more elaborate object behaviour can follow an actual need.

The example names illustrate readable code. A project-wide casing convention and formatter remain implementation choices.

Source: [v0.8 source layout](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#5-template-and-source-layout) and [state/model code](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#14-state-mutation-and-model-code).

## 3. State and dependencies

| Level | Rule |
|---|---|
| Requirement | Keep domain state and its meaning experiment-owned. |
| Requirement | Make cross-boundary state changes explicit through the owning side's interface or callbacks. |
| Default | Pass the narrowest useful values, sub-state, or service to each function. |
| Default | Keep model calculations dependent on their mathematical/domain inputs. |
| Judgement | Mutate explicitly supplied owned state or return results according to clarity. |

### Example

```lua
local function advance_body(body, acceleration, timestep)
    body.velocity = body.velocity + acceleration * timestep
    body.position = body.position + body.velocity * timestep
end
```

This scalar illustration makes the changed object visible at the call boundary. Integration method and physical model remain experiment decisions.

```text
Calculation       ← domain values
State update      ← owned object or sub-state
Coordinator       ← complete experiment state where useful
Lab interaction   ← relevant Lab service or Lab reference
```

### Reasoning

Explicit dependencies reveal what a function can read or change. Small calculations are easy to reuse in different experiments. Coordinators retain enough context to organise updates and route effects through the appropriate owner.

Source: [v0.8 ownership and mutation](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#13-responsibilities-and-data-flow).

## 4. Naming and comments

| Level | Rule |
|---|---|
| Default | Use names that describe domain meaning and intention. |
| Default | Expand acronyms where the intended reader would need to decode them. |
| Default | Use a meaningful domain name when a generic name hides useful information. |
| Default | Let structure and naming explain routine behaviour. |
| Judgement | Add comments for reasoning, constraints, compromises, or temporary context. |

### Naming examples

| Meaning | Example name |
|---|---|
| Position in the body collection | `body_index` |
| Current selected domain object | `selected_body` |
| Simulation time between observations | `sampling_interval` |
| Run initial conditions | `run_configuration` |
| Values returned by spatial queries | `spatial_hits` |

### Comment example

```lua
-- Reuse the registered body so Lab accessors keep their reference.
body.position.x = initial_position.x
body.position.y = initial_position.y
```

### Reasoning

Names carry context through calls and reviews. The comment explains a lifetime constraint that the assignments alone cannot express. Mathematical terms and established API names can remain concise where their meaning is clear.

Source: [v0.8 coding practices](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#8-git-tracking-and-coding-practices).

## 5. Functions and modules

| Level | Rule |
|---|---|
| Default | Give each function or module a coherent responsibility. |
| Default | Use guard clauses and early returns when they simplify control flow. |
| Default | Keep nesting shallow where practical. |
| Judgement | Split code according to responsibilities and readability. |
| Default | Refactor when emerging structure is clearly poor. |

### Guard clause example

```lua
local function selected_body_name(selected_body)
    if selected_body == nil then
        return "Select a body"
    end

    return selected_body.name
end
```

### Module responsibilities

```text
experiment.lua  → state ownership and coordination
simulation.lua  → calculations and explicit model updates
controls.lua    → typed control declarations
definitions.lua → optional domain-data construction
shaders/        → experiment-specific graphics effects
```

### Reasoning

Guard clauses keep the main path easy to follow. Responsibility-based modules let related behaviour change together. Function length and complexity are review judgements informed by clarity; the design leaves numeric limits open.

Source: [v0.8 module layout](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#5-template-and-source-layout) and [coding practices](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#8-git-tracking-and-coding-practices).

## 6. Callbacks and Lab integration

| Level | Rule |
|---|---|
| Requirement | Declare UI control types explicitly and use function references for their behaviour. |
| Requirement | Preserve stable IDs and declared validation for recorded parameters. |
| Requirement | Respect the Lab-owned run-configuration lock after run start. |
| Requirement | Keep domain object lifetime experiment-owned and registration lifetime Lab-owned. |
| Default | Expose live object values through capability accessors. |
| Default | Use Lab services for the generic rendering, input, timing, and audio responsibilities they own. |
| Requirement | Keep experiment contact response in experiment model/coordination code. |

### UI declaration example

This declaration illustrates the source design's intended API shape. Names remain subject to implementation.

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

### Accessor example

```lua
local function body_position(body)
    return body.position
end
```

The accessor exposes a live position reference for the capability reader. State mutation follows the ownership rules in section 3. The named-function rule above applies to UI declarations; capability adapters and coordinator closures follow their local needs.

### Reasoning

Typed controls make UI intent explicit. Named behaviour keeps declarations easy to scan and handlers easy to find. Live accessors preserve one authoritative domain state. Each side can evolve its implementation while keeping the ownership boundary clear.

For error policy, lifecycle, and open API choices, use the [runtime reference](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/developer-architecture.md#5-lifecycle-and-scheduling).

Source: [v0.8 entity contracts](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#17-entity-registration-and-lifetime) and [instrumentation](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#22-instrumentation-and-parameter-controls).

## 7. Tests and shared abstractions

Busted is the project's standard Lua testing framework. Use the [official Busted documentation](https://lunarmodules.github.io/busted/) for installation, running tests, assertions, test hooks, spies, and mocks. The [Busted repository](https://github.com/lunarmodules/busted) provides source code and release history.

| Level | Rule |
|---|---|
| Requirement | Generalise, document, and appropriately test code promoted into the Lab. |
| Default | Cover deterministic shared behaviour with Busted. |
| Requirement | Exercise the real Template bootstrap through distributed fetch tooling. |
| Requirement | Run experiment tests when present; allow the test stage to skip successfully when absent. |
| Judgement | Add experiment tests when they help investigate or protect useful behaviour. |
| Judgement | Promote abstractions after repeated use demonstrates shared value. |

### Behaviour example

This self-contained Busted illustration checks a coordinate relationship. The function name is illustrative.

```lua
local function world_to_screen(world_position, camera_position, zoom)
    return (world_position - camera_position) * zoom
end

describe("camera coordinate transform", function()
    it("places the camera position at the screen origin", function()
        assert.are.equal(0, world_to_screen(12, 12, 3))
    end)

    it("scales displacement from the camera", function()
        assert.are.equal(15, world_to_screen(17, 12, 3))
    end)
end)
```

### Promotion path

```text
Experiment-specific implementation
                ↓ repeated useful applications
Identify generic responsibility
                ↓
Generalise → Document → Test → Share through Lab
```

### Reasoning

Busted was chosen because the project author has used it on previous Lua and LÖVE projects. That familiarity provides a practical starting point for the project's testing workflow.

Behaviour-focused checks protect relationships the system relies on. Real bootstrap validation tests the path users take to obtain the dependency. Promotion adds shared maintenance obligations when reuse has established their value.

The architecture reference specifies the [repository validation matrix](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/developer-architecture.md#12-engineering-and-validation).

Source: [v0.8 testing](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#9-testing-and-validation) and [principles](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#2-principles-and-boundaries).

## 8. Tooling, commits, and review

| Level | Rule |
|---|---|
| Default | Make small, focused development commits as work progresses. |
| Requirement | Integrate ordinary human Lab/Template changes through reviewed, validated PRs. |
| Requirement | Supply exactly one release-intent label and the required PR documentation. |
| Default | Share locally runnable POSIX script logic with GitHub workflows where practical. |
| Requirement | Read authoritative version state from committed metadata. |
| Judgement | Use Issues, branches, and experiment tests where they support the work. |

### Review prompts

- Is each state change visible and owned by the appropriate side?
- Do names communicate domain meaning?
- Do functions receive useful, focused dependencies?
- Does each module have a coherent responsibility?
- Do comments explain reasoning or constraints?
- Do UI declarations keep metadata and behaviour easy to locate?
- Are generic mechanisms and domain responses assigned clearly?
- Does shared behaviour have appropriate tests and documentation?
- Do applicable validation and bootstrap checks pass?
- Does the PR express release intent and required migration guidance?

### Reasoning

Focused changes make review and diagnosis easier. Shared scripts keep local and automated behaviour aligned. The PR records a complete proposed change, so it carries compatibility intent and release documentation.

Release labels, workflow steps, protected-main automation, and distribution contracts are maintained in the [delivery reference](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/developer-architecture.md#13-releases-and-deployment).

Formatting tools, linter selections, exact casing rules, and API naming can be recorded when chosen during implementation. Current examples illustrate the principles above.

Source: [v0.8 development practices](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#8-git-tracking-and-coding-practices) and [release contracts](/Users/shaun/Documents/GitHub/Creative-Coding-Lab/Docs/Archive/creative-coding-lab-design-v0.8.md#10-pull-requests-and-release-automation).
