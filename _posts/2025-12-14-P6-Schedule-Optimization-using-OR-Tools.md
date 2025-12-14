---
title:  P6 Schedule Optimization using OR-Tools
date: 2025-12-14
---

![Portrait of I.V.Klyun (Klyunkov)](https://upload.wikimedia.org/wikipedia/commons/a/a9/Portrait_of_I.V.Klyun_%28Klyunkov%29.jpg)

<p align="center">
  <em>Kazimir Malevich. "Portrait of Ivan Vasilyevich Klyun (Improved portrait of Klyun (Builder)"</em>
</p>

# Resource Optimization: From CPLEX to OR-Tools

In a series of previous posts, I discussed two related but initially independent topics:

* parsing **Primavera P6 (XER)** files using the Python library `xerparser`
* building schedule optimization models using **CPLEX**

The natural next step was to combine these approaches into a single applied project: resource optimization directly on top of P6 schedules. At this stage, however, it became clear that the **limits of the CPLEX Community Edition are too restrictive for this type of problem**, and since the commercial version was not available to me at the time, I decided to switch to **Google OR-Tools**. As a result, the project evolved into a prototype called **Equilibrium**.

In this article, I explain why this transition was necessary, how the optimization model is structured, and which scenarios have been implemented.

---

## The Problem: Limitations of CPLEX Community Edition

CPLEX works very well for educational and small-scale scheduling problems. However, when moving to bigger XER files exported from Primavera, the constraints of the Community Edition (1000 variables and 1000 constraints) become noticeable.
As long as access to the commercial version is not available for me, this naturally led me to look for an alternative solver better suited for working with real project data.

---

## Why OR-Tools

As an alternative, **Google OR-Tools (CP-SAT)** was selected:

* fully open-source
* well suited for **discrete scheduling problems**
* native support for:

  * interval variables
  * precedence constraints
  * resource capacity constraints
* good scalability on scheduling benchmarks

---

## Overall Architecture

The **Equilibrium** project implements a full processing pipeline for Primavera P6 schedules:

```text
XER (Primavera P6)
        ↓
   xerparser
        ↓
   pandas / preprocessing
        ↓
   OR-Tools (CP-SAT)
        ↓
 Streamlit + visualization
```

Technology stack:

* **xerparser** — XER file parsing
* **pandas** — data transformation and preparation
* **OR-Tools (CP-SAT)** — optimization engine
* **Streamlit** — UI and scenario execution
* **matplotlib** — schedule and resource visualization

This allows P6 schedules to be passed directly into the solver without manual intermediate transformations.

---

## Implemented Optimization Scenarios

The current version includes two scenarios that reflect different management perspectives.

---

### Scenario 1. Automatic Resource Assignment

**Problem statement**

* a set of activities with durations and precedence constraints
* `N` interchangeable resources
* objective: minimize total project duration (makespan)

**Model characteristics**

* all activities are automatically assigned to resources
* each resource can process only one activity at a time
* all logical dependencies are respected
* the solver determines both the assignment and execution order

This scenario is useful for estimating the **theoretical minimum project duration** given a fixed number of crews.

---

### Scenario 2. Optimization Within Existing Resource Groups (Sub-Crews)

This scenario is closer to real Primavera workflows.

**Initial setup**

* resources are already defined in Primavera
* activities are grouped by a UDF (crew / zone / contractor)
* each group is effectively executed sequentially

**Core idea**

Each resource group is split into `N` parallel **Sub-Crews**, while:

* preserving the original schedule structure
* performing optimization **locally within each group**
* introducing controlled parallel execution

**Practical meaning**

The model can answer questions such as:

* what happens to the schedule if a second sub-crew is added?
* where do conflicts between precedence logic and resource capacity arise?
* does increasing resources help, or is the schedule precedence-bound?

---

## Current Limitations

The project is currently a demo/test build, with several intentional simplifications:

* a continuous **7-day working calendar** is used
* weekends, holidays, shifts in Primavera calendars are ignored
* **activity execution status is not considered** — started and completed activities are treated the same as not-yet-started ones

Even with these limitations, the tool is already effective for identifying bottlenecks in resource logic.

---

## Conclusions

The combination of **xerparser + OR-Tools** has proven to be a viable alternative to commercial solvers for analyzing and optimizing Primavera P6 schedules:

* Community Edition limitations are avoided
* flexible optimization scenarios become possible
* real XER files can be used directly
* the approach is well suited for R&D, pre-project analysis, and hypothesis testing

Future development directions: support for P6 calendars, richer resource models, activity status handling.
