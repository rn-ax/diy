# diy

A collection of hands-on experiments in electronics, firmware, 3D printing and home automation, and the tools that support them.

## Layout

- `experiments/` holds one directory per experiment.
- `tools/` holds one directory per tool.

## Experiments

Each experiment lives in its own directory under `experiments/`, named `YYYY-MM-DD - <name>`, where the date is the day the experiment started and the name is a short lowercase, hyphenated description, for example `experiments/2026-11-21 - led-candles`.

The directory names contain spaces, so every shell path to an experiment is quoted.

Each experiment has its own `AGENTS.md` that holds what an agent needs for that experiment: the goal, the constraints, and the non-obvious facts established along the way. An agent working inside an experiment reads that file in addition to this one, and the experiment's file takes precedence inside its directory.

An experiment keeps everything it needs inside its directory: design notes, plans, firmware, configuration, CAD sources, measurements and photos. Design documents and plans go under the experiment's `docs/` directory.

Experiments are independent of each other and import nothing across directories. Code or a model that two experiments both need is copied into each.

An experiment that outgrows the repository graduates to its own repository under `rn-ax`. Its directory is then replaced by a README that links to the new repository.

## Tools

Each tool lives in its own directory under `tools/`, named with a short lowercase, hyphenated description and no date, for example `tools/breadboard-diagrams`. A tool has its own `AGENTS.md` and a `mise.toml` with the tasks that build and test it, and keeps everything it needs inside its directory.

A tool is a command that takes its input and output paths as arguments. An experiment uses a tool by running it, and keeps the tool's inputs and the files the tool generates inside the experiment's own directory.

Tools are independent of experiments and of each other. A tool that outgrows the repository graduates like an experiment, to its own repository under `rn-ax`.

## CAD

Printable parts are modeled in CadQuery, as a Python script whose dimensions are named variables at the top. The exported STEP and STL files are committed next to the script. Tests check dimensions on the exported geometry, so they hold if the modeling tool changes.

## Starting an experiment

Create the directory under `experiments/` with today's date, write its `AGENTS.md` with the goal and constraints as they are understood so far, and keep that file current as facts are established.

## Personal information

Files in the repository carry no personal information: no names, no locations, and no metadata that identifies a person or a device. The person the experiments are for is called "the user". Photos have their metadata stripped before they are committed.

## Changes

All changes reach `main` through a pull request. A pull request covers one experiment or one tool.
