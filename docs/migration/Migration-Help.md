# Migration Help — Considering MFR from MVVM or MVI

If you are coming from traditional **MVVM** or **MVI** architecture, this document provides a high-level conceptual overview of why developers transition to MFR and what changes at a high level.

---

## "I use MVVM or MVI. Why would I consider MFR?"

Applications growing in complexity often encounter two main architectural pain points:
1. **Screen ViewModel Bloat**: Persistent feature state, derived state calculations, and operation logic get trapped inside individual screen ViewModels.
2. **State Fragmentation & Manual Sync**: Navigating across screens requires passing full objects, re-fetching data, or writing imperative code to keep derived state in sync.

MFR addresses these issues by making state ownership, state derivation, and operation coordination explicit architectural constraints.

---

## MVVM → MFR at a Glance

In standard MVVM, ViewModels accumulate persistent data, derived filtering logic, UseCase calls, and UI toggles.

### Conceptual Shift:
```text
Screen ViewModel State             →   Feature-Lived FluxDeck State (Model)
Manually Maintained Derived State  →   Declarative Formulas (combine, map)
Screen-Only UI Toggles             →   Reflector (ViewModel)
Operation Execution                →   FeatureCoordinator
Transient UI Effects               →   Events / StateReflector
```

---

## MVI → MFR at a Glance

In standard MVI, a single monolithic `UiState` data class is updated by central Reducer functions (`(State, Intent) -> State`) via constant data copying (`state.copy(...)`).

### Conceptual Shift:
```text
Monolithic UiState                →   Decoupled FluxDeck State + Reflector State + Formulas
Reducer-Derived Properties        →   Auto-Evaluating Declarative Formulas
Operations                         →   FeatureCoordinator Operations
Side Effects                       →   Events / StateReflector
```

---

## Complete Step-by-Step Migration Package

The complete step-by-step migration procedures, complete before-and-after code transformations, migration safety processes, and checklists are available in the **Complete MFR Setup**:

**[Save the implementation and migration time — Get the Complete MFR Setup]([GUMROAD_PRODUCT_URL])**
