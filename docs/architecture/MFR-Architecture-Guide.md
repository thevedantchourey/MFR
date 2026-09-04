# MFR Architecture Guide

This guide provides a comprehensive technical breakdown of the three core architectural roles in MFR: **Model (FluxDeck)**, **Formula**, and **Reflector**.

---

## 1. Model — FluxDeck

**FluxDeck is the Model in MFR.**

The FluxDeck is the concrete construct that owns the feature's persistent, feature-lived state and acts as the single source of truth for the entire feature workflow. Every Feature Instance owns exactly one FluxDeck.

### Key Responsibilities
* **State Ownership**: Holds feature-lived facts (e.g., `items`, `searchQuery`, `selectedFeatureId`, `draft`).
* **Source of Truth**: Acts as the feature's memory across screens.
* **Workflow Lifetime**: Bound to the feature workflow lifetime rather than an individual screen lifecycle.
* **UI Framework Independent**: Free of Android ViewModels, Context, or UI view classes.
* **Source for Formulas**: Provides source StateFlows to Formulas for declarative derivation.

### Meaning of "Persistent"
In MFR, "persistent feature state" means **feature-lived state**, not necessarily data persisted to disk. State variables such as `searchQuery`, `clickedItem`, `activeFilter`, and `draft` are feature-lived state variables even if they exist only in memory.

### FluxDeck Invariants & Boundaries
Do **NOT** put into FluxDeck:
* Network calls or API requests
* Repository invocations or DataSource access
* UseCase execution
* Operation result/error handling
* Transient UI events (Toasts, Snackbars)
* Ephemeral screen-only state (e.g., `dialogVisible`, `isCardExpanded`)

---

## 2. Formula — Declarative State Derivation

A **Formula** describes **how state depends on other state**.

Instead of imperative code that manually synchronizes multiple StateFlows when an event occurs, a Formula establishes a declarative, reactive dependency pipeline.

### Flow Mechanisms
Formulas use reactive Kotlin Flow operators:
* `combine(...)`
* `map(...)`
* `flatMapLatest(...)`
* `filter(...)`

```kotlin
private val items = MutableStateFlow<List<Item>>(emptyList())
private val searchQuery = MutableStateFlow("")

// Formula: Filtered items automatically update whenever items or searchQuery change
val filteredItems = combine(items, searchQuery) { items, query ->
    items.filter { it.name.contains(query, ignoreCase = true) }
}
```

### Formula Invariant
* Reads source state as input.
* Computes derived state representations (e.g., filtered lists, totals, progress percentages).
* Reacts automatically whenever any input dependency emits a new value.
* Never manually push updates into a derived state variable when a Formula dependency can express it.
* A Formula performs **no side effects** (no network/DB calls, no data mutation, no UI events).

---

## 3. Reflector — UI-Facing Layer

The **Reflector** is the **ViewModel role in MFR**.

It acts as the communication bridge between the view layer and the feature core without becoming the owner of persistent feature state.

### Key Responsibilities
1. **Ephemeral Screen State Ownership**: Owns state whose lifetime belongs strictly to a single screen (e.g., `isEditing`, `isFilterVisible`, `selectedItemIds`).
2. **State Reflection**: Observes FluxDeck persistent state and Formulas, reflecting them to the UI as read-only StateFlows.
3. **UI Intent Forwarding**: Receives user actions from the UI and forwards operations to the FeatureCoordinator or FluxDeck.

### Reflector Invariant
> **The Reflector owns ephemeral screen state and reflects persistent/derived feature state to the UI without becoming the owner of that feature state.**

### Reflector Boundaries
* Do not duplicate persistent feature state inside the Reflector (e.g., maintaining `_reflectorItems` alongside `FluxDeck.items`).
* Do not write UseCase result-handling logic inside the Reflector (use `FeatureCoordinator` and `StateReflector`).
* The UI **never** observes FluxDeck directly. Feature state always passes through the Reflector (`FluxDeck → Formula → Reflector → UI`).
