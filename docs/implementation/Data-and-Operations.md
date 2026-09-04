# Data and Operations Guide

MFR clearly distinguishes between **Operation Flow** (executing user actions) and **Reactive State Flow** (propagating data updates).

---

## 1. Operation Flow

Operation Flow describes the execution path when a user triggers an action:

```text
UI Action → Reflector → FeatureCoordinator → UseCase → Repository → DataSource
```

* The UI forwards intents to the Reflector.
* The Reflector invokes the `FeatureCoordinator`.
* The `FeatureCoordinator` executes the relevant `UseCase`.
* The `UseCase` interacts with the `Repository` and `DataSource`.

---

## 2. Reactive State Flow

Reactive State Flow describes the propagation of data changes up to the user interface:

```text
Data Change (Database / Network) → Repository Flow → FluxDeck → Formula → Reflector → UI
```

* The `FluxDeck` observes Repository Flows.
* When data changes in the Repository/Database, the Flow emits new items automatically.
* `FluxDeck` receives updates and updates its internal state.
* `Formulas` re-evaluate automatically.
* `Reflector` reflects updated state to the UI.

---

## 3. FluxDeck Participation vs. Bypass Rule

### FluxDeck-Participating Operation
If an operation requires direct participation in FluxDeck-owned feature state (e.g. updating a draft form or search query):
```text
UI Action → Reflector → FeatureCoordinator → UseCase → Repository → DataSource → FluxDeck → Formula → Reflector → UI
```

### Bypass Operation
If an operation does **not** directly require FluxDeck state manipulation (e.g. deleting a database record):
```text
Reflector → FeatureCoordinator → UseCase → Repository → DataSource
                                                            │
                                             (Observed Flow updates automatically)
                                                            │
                                                            ▼
                                                         FluxDeck → Formula → Reflector → UI
```

> **Bypassing FluxDeck does not stop FluxDeck from reacting to data changes.** It only means the *operation call itself* is not routed through FluxDeck. FluxDeck independently observes Repository Flows and reacts automatically.
