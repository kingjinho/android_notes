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

