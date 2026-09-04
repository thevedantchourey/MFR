# Quick Reference and Comparisons Guide

## 1. MFR vs. MVVM

MFR does not claim that MVVM is bad. MVVM can be implemented cleanly and can use reactive state.

| Dimension | Standard MVVM | MFR (Model · Formula · Reflector) |
| :--- | :--- | :--- |
| **State Ownership** | Fragmented across screen ViewModels. | Owned centrally by feature `FluxDeck`. |
| **Derived State** | Manually calculated in ViewModels or Repositories. | Declaratively derived via `Formulas` (`combine`, `map`). |
| **ViewModel Role** | Holds persistent state, derived state, network operations, and UI concerns. | `Reflector` holds only ephemeral screen state and reflects FluxDeck. |
| **Feature Lifetime** | Tied to screen/Activity/Fragment lifecycle. | Bound to the `Continuum` (feature workflow lifecycle). |
| **Operation Coordination**| Direct UseCase/Repository calls inside ViewModels. | Coordinated separately via `FeatureCoordinator`. |

---

## 2. MFR vs. MVI

| Dimension | Standard MVI | MFR (Model · Formula · Reflector) |
| :--- | :--- | :--- |
| **State Structure** | Single monolithic immutable `UiState` data class. | Decoupled feature state (`FluxDeck`), derived state (`Formulas`), and screen state (`Reflector`). |
| **State Calculation** | Central Reducer functions (`(State, Intent) -> State`). | Declarative, independent reactive Flow `Formulas`. |
| **User Intent** | Dispatched as Intent/Action objects to Reducers. | Forwarded via Reflector to `FeatureCoordinator` operations. |
| **Side Effects** | Managed via SideEffect/SingleEvent streams. | Managed via `Events` and `StateReflector`. |

---

## 3. Quick Reference Cheat Sheet

### Core Roles
```text
Model      → FluxDeck (Feature-lived state owner)
Formula    → Declarative State Derivation (combine, map)
Reflector  → UI-facing Layer / ViewModel (Ephemeral screen state & state reflection)
```

### Supporting Infrastructure
```text
FeatureCoordinator → Operation coordination & UseCase execution
StateReflector     → Result handling & event emission
Events             → Transient one-time UI effects
Continuum          → Feature memory & lifetime across screens
```

### The Mental Model
> **Interaction performs.**\
> **FluxDeck owns.**\
> **Formula derives.**\
> **Reflector reflects.**
