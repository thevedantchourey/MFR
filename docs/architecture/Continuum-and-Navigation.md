# Continuum and Navigation Guide

The **Continuum** defines feature lifetime and state continuity across screens.

---

## 1. Feature Continuum & Lifetime

Instead of thinking of an application as isolated screens:
```text
Screen A ──> Screen B ──> Screen C
```
MFR thinks of the application as a continuous feature:
```text
                Feature Workflow
                       │
                    FluxDeck
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Screen A       Screen B       Screen C
```

### Feature Instance Rule
> **Each Feature Instance owns exactly one FluxDeck. Its lifetime is independent of an individual screen and lasts as long as the continuous feature workflow requires.**

* Use dependency injection scoping to bind the FluxDeck to the lifetime of the feature workflow.
* Do **NOT** make every FluxDeck a global singleton by default.

---

## 2. Cross-Screen State Ownership

In traditional screen-centric architectures, moving from Screen A to Screen B often requires re-fetching data or passing complete data objects through navigation arguments.

In MFR, if data belongs to the feature's persistent state, it is owned by the **FluxDeck**.

### The `clickedItem` Example
1. User clicks an item on Screen A.
2. Reflector A sets `FluxDeck.clickedItem = item` (or `clickedItemId`).
3. App navigates to Screen B.
4. Reflector B observes `FluxDeck.clickedItem` (or a Formula deriving destination state).
5. Screen B UI displays the state immediately without re-fetching.

---

## 3. Navigation Rules

Navigation is responsible for **moving between screens**. FluxDeck is responsible for **owning feature state**.

### Navigation State Rule
> **Do not pass feature state through navigation arguments when that state is already owned by the FluxDeck.**

Navigation arguments may exist for parameters that genuinely belong to navigation routing itself, but navigation routes must **never** duplicate feature state that already has a FluxDeck owner.
