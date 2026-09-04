# Implementation Rules and Testing Guide

This guide details project package organization, implementation rules, and unit testing strategies for MFR applications.

---

## 1. Project Package Organization

The established project organization for MFR applications is structured as:

```text
<your-package>/
│
├── app/                          # Application & Feature Layer
│   ├── data/
│   │   ├── UserPreferences.kt    # UserPreferences DataStore
│   │   ├── mapper/               # DTO, Entity, and Domain Mappers
│   │   └── repository/           # Repository implementations
│   ├── di/
│   │   └── AppModule.kt          # DI module binding FluxDecks, Coordinators, ViewModels
│   ├── domain/
│   │   ├── coordinator/          # FeatureCoordinators (Operation Layer)
│   │   ├── model/                # Domain Models
│   │   ├── repository/           # Repository Interfaces
│   │   └── usecase/              # Feature UseCases
│   ├── fluxdeck/                 # FluxDecks (The Model - Feature State Owners)
│   ├── presentation/
│   │   ├── screens/              # Feature UI Screens
│   │   ├── components/           # Reusable Presentation Components
│   │   ├── navigation/           # Feature Navigation Definitions
│   │   └── viewmodels/           # Reflectors (ViewModel role)
│   ├── states/                   # UI & Feature State definitions
│   ├── Events.kt                 # App-level Transient Events
│   └── StateReflector.kt         # App-level StateReflector Utility
│
└── core/                         # Generic Shared Infrastructure Layer
    ├── data/
    │   ├── entities/             # Room Database Entities & Enums
    │   ├── local/                # Room Database, DAOs, & LocalDataSources
    │   ├── networking/           # Ktor HttpClientFactory, safeCall, constructUrl
    │   └── remote/               # Ktor RemoteDataSources & DTOs
    ├── domain/
    │   └── util/                 # Reusable Domain Utilities
    └── presentation/
        └── util/                 # Presentation utilities
```

---

## 2. Implementation Rules

1. **Classify Ownership First**:
   * Feature-lived state $\rightarrow$ `FluxDeck`
   * Screen-lived state $\rightarrow$ `Reflector`
2. **Identify Derived State**:
   * If a value can be calculated from existing state, make it a `Formula` (`combine`, `map`). Never maintain manually synchronized copies of derived values.
3. **Keep Operations Out of FluxDeck**:
   * Do not put Repository calls, network requests, UseCase execution, or transient events in `FluxDeck`.
4. **Do Not Duplicate Persistent State in Reflector**:
   * If `FluxDeck` owns `items: StateFlow<List<Item>>`, do not create a parallel `_items` StateFlow inside `Reflector`.
5. **Do Not Manually Synchronize Formulas**:
   * Avoid imperative code that updates multiple state variables on an event. Define declarative Flow formulas instead.
6. **Use FeatureCoordinator for Operations**:
   * Keep UseCase result-handling logic out of ViewModels (`Reflector`). Coordinate operations through `FeatureCoordinator`.
7. **Keep Events Separate**:
   * Never create persistent boolean state (`savedSuccessfully = true`) solely to show a Snackbar or Toast. Use `Events`.

---

## 3. Unit Testing Strategy

MFR's separation of concerns allows components to be tested independently without massive ViewModel tests.

### FluxDeck & Formulas Testing
Test state facts and Formula derivations without Android UI dependencies:
* Set `FluxDeck.items = [A, B, C]` and `FluxDeck.searchQuery = "A"`.
* Assert `FluxDeck.filteredItems` emits `[A]`.
* Update `searchQuery = "B"` and assert `filteredItems` re-evaluates to `[B]`.

### FeatureCoordinator Testing
Test operation coordination and side effects using mock UseCases:
* Mock UseCase returns `Result.Success`.
* Invoke `coordinator.executeOperation()`.
* Assert `StateReflector` emits `Events.Success` or `FluxDeck` updates appropriately.

### Reflector Testing
Test screen-only ephemeral state, user-intent forwarding, and correct exposure of FluxDeck flows.
