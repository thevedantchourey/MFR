# Supporting Architecture Guide

MFR surrounds its three core roles with supporting engineering layers to maintain strict separation of concerns:

```text
DataSource → Repository → UseCase → FeatureCoordinator → FluxDeck / Reflector
```

---

## 1. FeatureCoordinator

The **FeatureCoordinator** is the operation coordination layer in MFR. It sits between UseCases and the MFR core (FluxDeck / Reflector) to manage external business operations and side effects.

### Responsibilities
* **Operation Execution**: Invokes and coordinates relevant UseCases.
* **Result Processing**: Handles operation results (`Result.Success`, `Result.Error`) using `StateReflector`.
* **Routing**: Determines whether an operation requires direct FluxDeck participation or bypasses directly to the Reflector.
* **Supporting Infrastructure**: Supporting layer, **not** a fourth core MFR role.

### Architectural Boundary
```text
FeatureCoordinator → UseCase → Repository → DataSource
```
**Rule**: FeatureCoordinator must **never call Repositories or DataSources directly**. The UseCase remains the strict business-operation boundary.

---

## 2. StateReflector

`StateReflector` is a supporting result/event infrastructure utility used by the `FeatureCoordinator`.

### StateReflector vs. MFR Reflector
* **MFR Reflector**: Architectural role (ViewModel). Holds ephemeral screen state and reflects FluxDeck state to the UI.
* **StateReflector**: Supporting utility class used inside `FeatureCoordinator`. Extracts repetitive UseCase result handling and emits transient events. Has **zero FluxDeck responsibility**.

```kotlin
class StateReflector<E : Any>(
    private val scope: CoroutineScope
) {
    private val _events = MutableSharedFlow<E>()
    val events: SharedFlow<E> = _events.asSharedFlow()

    fun emitEvent(event: E) {
        scope.launch { _events.emit(event) }
    }
}
```

---

## 3. Events

Persistent state and transient UI events are fundamentally different concepts and must be kept strictly separate.

* **Persistent / Derived State**: Things that *exist* (e.g., username, item list, searchQuery). Owned by FluxDeck or Reflector.
* **Transient Events**: Things that *happen* once (e.g., Toasts, Snackbars, Navigation triggers, Audio alerts). Emitted via `SharedFlow`.

**Event Invariant**: Never create persistent boolean state (`savedSuccessfully = true`) solely to trigger a transient UI effect.

---

## 4. Result Handling

Operations return a sealed `Result` representing `Success` or `Error`:

```kotlin
sealed interface Result<out D, out E> {
    data class Success<out D>(val data: D, val message: String? = null) : Result<D, Nothing>
    data class Error<out E>(val error: E, val message: String? = null) : Result<Nothing, E>
}
```

`FeatureCoordinator` processes UseCase results and uses `StateReflector` to communicate outcomes to the UI via Events. Raw technical errors are logged for developers (e.g., via Timber) while user-facing feedback uses friendly error messages.
