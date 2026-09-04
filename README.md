# MFR — Model · Formula · Reflector

MFR (Model–Formula–Reflector) is a feature-oriented, reactive application architecture designed for software with shared, interconnected state across multiple screens and complex user workflows.

Its central goal is to separate **state ownership** from **screen lifecycle orchestration** and replace manual state synchronization with declarative, reactive dependencies.

> **Interaction performs. FluxDeck owns. Formula derives. Reflector reflects.**

---

## Introduction

If you want to understand where MFR came from and the thinking behind it, read the original introduction:

**[Read the original MFR introduction on Medium](https://medium.com/@vedantchourey99/solutions-come-from-surviving-52274b967c10?sharedUserId=vedantchourey99)**

Traditional screen-centric architectures (such as standard MVVM or MVI) map state directly to individual ViewModels or screens. As applications grow in complexity, features spanning multiple screens suffer from fragmented ownership, duplicated state, and fragile manual synchronization.

MFR shifts state ownership from individual screens to the **Feature Continuum**, making state dependencies explicit and reactive across the entire feature lifecycle.

---

## What MFR Solves

MFR was created to solve specific architectural bottlenecks in modern application development:

1. **God-Class ViewModels**: ViewModels accumulate persistent state, derived state, filtering, sorting, network operations, event propagation, and UI presentation concerns.

2. **Fragmented Ownership**: Splitting a feature across multiple screen ViewModels leaves no single owner for shared, feature-lived state. Information gets duplicated or re-fetched across navigation boundaries.

3. **Manual Synchronization**: Maintaining multiple independently mutable StateFlows requires imperative code to keep derived UI states in sync, creating state drift and subtle UI bugs.

MFR elevates feature state ownership above individual screens and turns state derivation into a declarative pipeline.

The central question MFR asks is:

> **Who owns this state, what does it depend on, and how long should it live?**

---

## Core Architecture

MFR defines three architectural roles:

```text
Model      → FluxDeck

Formula    → Declarative State Derivation

Reflector  → UI-facing State Reflection
```

```text
                         UI

                          │

                          ▼

                     Reflector

                  ┌──────┴──────┐

                  │             │

          Ephemeral State   Reflected State

                  │             ▲

                  │             │

                  │          Formula

                  │             ▲

                  │             │

                  │          FluxDeck

                  │           Model
```

### 1. Model — FluxDeck

**FluxDeck is the Model in MFR.**

The FluxDeck is the concrete construct that owns the feature's persistent, feature-lived state and acts as the single source of truth for the entire feature workflow.

* **Feature-Lived Lifetime**: Lives with the feature continuum rather than an individual screen. It survives navigation between screens within the same feature workflow.

* **UI Independent**: Free of Android framework dependencies (such as ViewModels, Context, or UI lifecycle classes).

* **Pure State Owner**: Holds feature facts and source state for Formulas.

* **What it does NOT do**: FluxDeck does not execute network calls, invoke Repositories, execute UseCases, handle UI events, or store screen-only ephemeral state.

### 2. Formula — Declarative State Derivation

A Formula describes **how state depends on other state**. It transforms source state from the FluxDeck into derived representations declaratively using reactive Flow operators (`combine`, `map`, `flatMapLatest`, `filter`).

```kotlin
val filteredItems = combine(items, searchQuery) { items, query ->
    items.filter { it.name.contains(query, ignoreCase = true) }
}
```

* **Formula Invariant**: Reads state, derives state, and reacts automatically when dependencies change.

* **No Side Effects**: Formulas do not execute network/database operations, mutate external data, or emit transient UI events.

* **Declarative Dependency**: If a value can be derived from existing state, it must be defined as a Formula rather than manually pushed into another mutable state variable.

### 3. Reflector — UI-Facing Layer

The Reflector is the **ViewModel role in MFR**. It acts as the UI-facing communication bridge between the view layer and the feature core without becoming the owner of feature state.

* **Ephemeral Screen State Owner**: Owns state whose lifetime belongs strictly to one screen (e.g., `isEditing`, `isFilterVisible`, `selectedItemIds`).

* **State Reflection**: Observes FluxDeck persistent state and Formulas, reflecting them to the UI.

* **UI Intent Forwarding**: Receives user actions from the UI and forwards operations to the FeatureCoordinator or FluxDeck.

* **UI Boundary**: The UI **never** observes FluxDeck directly. Feature state always passes through the Reflector before reaching the UI (`FluxDeck → Formula → Reflector → UI`).

---

## Supporting Architecture

MFR surrounds its three core roles with supporting engineering layers to maintain strict separation of concerns:

```text
DataSource → Repository → UseCase → FeatureCoordinator → FluxDeck / Reflector
```

### FeatureCoordinator

Coordinates business operations and UseCases. It handles operation success/error results, uses `StateReflector` for standardized result handling, and determines whether an operation requires direct FluxDeck participation or bypasses directly to the Reflector.

### StateReflector

A supporting utility used by the `FeatureCoordinator` to handle operation results (`Result.Success`, `Result.Error`) and emit transient UI events. **`StateReflector` is supporting infrastructure and must not be confused with the MFR Reflector.**

### UseCase, Repository & DataSource

The business logic and data layer boundary. `FeatureCoordinator` interacts with UseCases, which interact with Repositories and DataSources. `FeatureCoordinator` never calls Repositories directly.

### Bypass Rule

Operations that do not require direct FluxDeck state manipulation may bypass FluxDeck (`Reflector → FeatureCoordinator → UseCase`). Bypassing FluxDeck does not prevent FluxDeck from updating; FluxDeck independently observes Repository Flows and reacts automatically to underlying data changes.

### Events

Transient UI effects (Snackbars, Toasts, Navigation triggers, Dialogs) that happen once. Events are kept strictly separate from persistent feature state.

### The Continuum

Refers to the feature's continuous memory. Instead of recreating state when navigating between Screen A, Screen B, and Screen C, all screens observe the same `FluxDeck`. Feature state lasts as long as the continuous feature workflow requires.

---

## Documentation Index

The complete public documentation in this repository is organized into logical guides:

* **[MFR Architecture Guide](docs/architecture/MFR-Architecture-Guide.md)**: In-depth breakdown of FluxDeck, Formula, and Reflector.

* **[Supporting Architecture Guide](docs/architecture/Supporting-Architecture-Guide.md)**: FeatureCoordinator, StateReflector, Events, and Result handling.

* **[Continuum and Navigation](docs/architecture/Continuum-and-Navigation.md)**: Feature workflow memory, cross-screen state, and navigation rules.

* **[Data and Operations](docs/implementation/Data-and-Operations.md)**: Operation flow, reactive state flow, FluxDeck participation, and the Bypass Rule.

* **[Implementation Rules and Testing](docs/implementation/Implementation-Rules-and-Testing.md)**: Project package organization, coding rules, and unit testing strategy.

* **[Migration Help](docs/migration/Migration-Help.md)**: High-level overview for developers considering MFR from MVVM or MVI.

* **[Quick Reference and Comparisons](docs/reference/Quick-Reference-and-Comparisons.md)**: MFR vs. MVVM, MFR vs. MVI, mental model, and cheat sheet.

---

# Getting Started

The documentation available in this repository contains the complete explanation of MFR, including its architecture, concepts, rules, and implementation guidelines.

You can absolutely implement MFR yourself by following the documentation. In fact, **my recommendation is to do exactly that at least once**. Implementing it yourself gives you a much better understanding of how MFR actually works and, more importantly, how it solves the problems it was designed to solve.

However, if you want to save yourself the reading and implementation time, you can purchase the complete MFR setup.

The complete setup includes:

* The **MFR structure template** for starting a new project

* Complete **AI-agent guidelines and implementation processes** for using MFR with AI coding agents

* A complete **migration guide** for moving existing projects from **MVVM → MFR** or **MVI → MFR**

* Supporting documentation and checklists to make the implementation or migration faster

The purpose of the paid setup is not to hide the architecture from you. The documentation here already explains it.

**You're paying to save time.**

---

### One important recommendation for AI users

If you use AI coding agents to implement or migrate to MFR, **do not trust the AI blindly**.

Keep an eye on what it is doing, make sure it follows the MFR guidelines, and continuously check whether it is actually respecting the architecture.

AI can help you implement MFR much faster, but you should still understand the architecture yourself.

---

### For migration

When migrating an existing project, the safest approach is to first make a copy of your main project and perform the migration on that copy.

Use the migration process on the copied project, verify the results, and only then apply the changes to your main project.

The same approach is recommended when experimenting with a new implementation.

**Learn it yourself when you can. Use the complete setup when you want to save time.**

---

## Get the Complete MFR Setup

**[Save the implementation and migration time — Get the Complete MFR Setup]([GUMROAD_PRODUCT_URL])**

### Free on GitHub

* Complete MFR explanation

* MFR concepts

* Architectural principles

* Responsibilities

* Implementation guidelines

* Architecture examples needed for learning

* Technical documentation

### Paid on Gumroad

* MFR structure template

* Ready-to-use new-project structure

* Complete AI-agent implementation guidelines/process

* Complete AI-assisted migration resources

* MVVM → MFR migration package

* MVI → MFR migration package

* Migration checklists

* Other prepared time-saving setup material

---

# A Note From the Author

MFR is a work of sleepless nights and many try and errors. while I have kept the implementation and core understanding for everyone, the paid version is for someone who wants to save their time or are using AI agents.

But if you've received a free copy from someone else, I totally understand your situation.

I built MFR because I wanted to solve problems, I kept seeing in real application development, and I'm genuinely happy when something I've built ends up helping another developer.

So use it to learn. Experiment with it. Teach it. Build something with it. Use it in an educational project. That's completely fine with me.

I only ask one thing: please don't sell it, redistribute it as your own work, or turn my work into a product you're selling to someone else.

And if MFR ever helps you — especially if you use it to teach, write about architecture, build a project, or learn something new — send me a message at **[vedantchourey99@gmail.com](mailto:vedantchourey99@gmail.com)** and tell me about it.

I don't need a coffee.

Honestly, knowing that something I built helped you is enough.
