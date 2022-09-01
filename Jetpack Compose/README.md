# Jetpack Compose

- Regenerate the entire screen from the scratch, then applying the only necessary changes
- Maybe potentially be expensive, in terms of time, computing power and battery usage
    - **To resolve this, Compose intellectually choose which to update**

```kotlin
@Composable
fun greeting(name: String) {
    Text("Hello $name")
}
```

## Recomposition

- Call composable functions again when inputs change

## Side-effect

- Any change that is visible to the rest of an app
    - Writing to a property of a shared object
    - Updating an observable in ViewModel
    - Updating shared preferences

## Things to be aware with Compose

1. Composable functions can execute in any order
2. Composable functions can execute in parallel
3. Recomposition can skip as many composable functions and lambdas as possible
4. Recomposition is optimistic and may be canceled
5. A composable function might be run quite frequently, as often as every frame of an animation

## In detail

#### 1. Composable functions can execute in any order

```kotlin
@Composable
fun buttonRow() {
    MyFancyNavigation {
        StartScreen()
        MiddleScreen()
        EndScreen()
    }
}
```

- May not run in the order you expected
- Compose has the option of recognizing that some UI elements are higher priority than others,
  and drawing them first
- **Composable functions needs to be self-contained, should not depend on other composable**

#### 2. Composable functions can execute in parallel

- Compose can run composable functions in parallel to optimize recomposition
- Take advantage of multiple cores
- Optimization = composable function might execute within a pool of background threads
- To ensure your application behaves correctly, all composable functions should have no side effects
    - Instead, trigger side effects from callbacks(onClick) that always execute on the UI thread
- **Should be thread-safe, since composable function can be invoked from different threads**
- Example

```kotlin
//the function writes to a local variable, 
//this code will not be thread-safe or correct:
@Composable
@Deprecated("Example with bug")
fun listWithBug(myList: List<String>) {
    var items = 0

    Row(horizontalArrangement = Arrangement.SpaceBetween) {
        Column {
            for (item in myList) {
                Text("Item: $item")
                items++ // Avoid! Side-effect of the column recomposing.
            }
        }
        Text("Count: $items")
    }
}
```

#### 3. Recomposition can skip as many composable functions and lambdas as possible

- Compose will try to update the portion that needs change

#### 4. Recomposition is optimistic and may be canceled

- Compose expects to finish before the parameters change again
- if parameters change before finish, it might cancel the current one and start new recomposition
- Ensure all composable functions and lambdas are idempotent and side-effect free

#### 5. A composable function might be run quite frequently, as often as every frame of an animation

- ddd