# Creative Coding Lab: Introduction and Getting Started

Creative Coding Lab is a platform for building interactive programming experiments, and the experiments created with it. Together, they support exploration of mathematics, simulation, graphics, and procedural generation through code and visual results.

Each experiment starts with a question. How do bodies move under gravity? How does a flock form? What happens when a mathematical rule changes?

The platform provides shared tools for cameras, controls, timing, and inspecting results. These give each experiment a starting point for exploring its question. The aim is to learn by building, changing values, and observing what happens.

**Project status:** Creative Coding Lab is in the design stage. This guide describes the intended experience. Experiment links, downloads, and verified setup instructions will be added as releases become available.

## Contents

- [Creative Coding Lab: Introduction and Getting Started](#creative-coding-lab-introduction-and-getting-started)
  - [Contents](#contents)
  - [Choose a way to explore](#choose-a-way-to-explore)
  - [Start your first experiment](#start-your-first-experiment)
  - [Find your way around](#find-your-way-around)
  - [Keep your results](#keep-your-results)
  - [Create an experiment](#create-an-experiment)
  - [Explore further](#explore-further)

## Choose a way to explore

| Your goal | Starting point |
|---|---|
| Try an experiment | Open its browser version when available. |
| Run it on your computer | Choose a published download for your operating system. |
| Explore its code | Open its repository and follow its README. |
| Build your own idea | Start from `CCL-Template` and use the developer reference. |

Experiments have their own repositories, named `CCL-Exp-{Experiment_Name}`. Each README explains the experiment and provides its available ways to run it. The project plans browser builds and downloads for Windows, macOS, and Linux where practical.

## Start your first experiment

The intended first-run flow is:

```text
Choose an experiment
        ↓
Read its question and setup
        ↓
Open the browser build or follow the download instructions
        ↓
Inspect the initial scene and choose starting values
        ↓
Press Play and observe
        ↓
Pause to inspect details or save useful results
```

Before pressing Play, open **Instrumentation** to see the experiment's controls. **Run Configuration** contains starting values. These values are recorded and locked when the run begins. **Live Controls** includes options such as showing trails or vectors during the run.

An experiment may also provide a seed. This value sets the starting point for its random choices. Recording it helps you revisit the same initial conditions.

## Find your way around

The experiment occupies the main view. A menu button in the top-left opens the sidebar. Simulation controls sit along the bottom.

```text
Sidebar                         Bottom bar
├── Instrumentation             ├── Restart
├── Debugging                   ├── Step
├── Controls                    ├── Simulation speed
└── Data                        ├── Play/Pause
                                └── Mute and volume
```

| Control | Purpose |
|---|---|
| Play/Pause | Start or pause simulation time. |
| Step | Advance one simulation step while paused. |
| Restart | Restore the current run's initial conditions. |
| Simulation speed | Change how quickly the simulation progresses. |
| Mute and volume | Adjust audio. |

Control availability depends on the experiment and its current state. The exact process for preparing another run is being finalised.

| Sidebar tab | What you can find |
|---|---|
| Instrumentation | Experiment settings and interactive controls. |
| Debugging | Performance information, overlays, and inspection tools. |
| Controls | The experiment's active keyboard and mouse bindings. |
| Data | Run history, recorded measurements, and import/export actions. |

The planned default bindings are Space for Play/Pause, arrow keys for camera movement, and the mouse wheel for zoom. Check the Controls tab for the bindings used by a particular experiment.

## Keep your results

A **run** is one execution from a set of starting conditions. A **session** is one launch of the application and may contain several runs.

The planned Data tab provides these actions:

| Action | Purpose |
|---|---|
| Export Run | Save a run record to a file. |
| Import Run | Bring a saved record into the application. |
| Load Run | Apply recognised starting values from a record. |
| Export Measurements to CSV | Take recorded values into an external analysis tool. |

Measurements depend on the experiment. They might include total energy, average speed, or distance between objects.

Native applications are intended to keep run history in application storage. Browser history lasts for the current page session. Export browser records you want to keep before reloading or leaving the page.

## Create an experiment

Creative Coding Lab uses Lua and LÖVE for its initial platform. The Template provides a starting structure and tools for fetching the shared Lab.

1. Choose a question to explore.
2. Create a repository from `CCL-Template`, named `CCL-Exp-{Experiment_Name}`.
3. Follow the Template's setup instructions for the required tools and Lab dependency.
4. Build the experiment, add useful controls, and record what you learn.

The [Developer Architecture and Reference](../Docs/developer-architecture.md) explains the intended structure and development workflow. The [Coding Standards](../Docs/coding-standards.md) provides practical guidance for writing code.

## Explore further

Each experiment includes an `EXPERIMENT.md` notebook with its question, setup, observations, and findings. Related scenarios are recorded as **Cases**, so an experiment can grow as new questions emerge.

| Repository | What it contains |
|---|---|
| `CCL-Docs` | Project documentation and guides. |
| `CCL-Lab` | Shared runtime and tools. |
| `CCL-Template` | Starting files and tooling for new experiments. |
| `CCL-Exp-{Experiment_Name}` | An individual experiment and its notebook. |

This guide follows the [v0.8 design](../Docs/Archive/creative-coding-lab-design-v0.8.md). It will grow with the first working experiments and their verified setup instructions.
