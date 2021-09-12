# Asynchronous Programming

# Concept of Asynchronous Programming

- These days, we have multiple processor cores
- Why should wait for something when we could do something else?
- This is the basis of asynchronous programming

# How did we implement Asynchronous Programming?

- Creating Threads,
-

# Why using Coroutines?

# What is Coroutine?

- Instead of blocking the thread, it suspends the coroutine
- Often called light-wight thread since we can run code on coroutines similar to how we run code on threads

# How does it work?

- While performing the network requests, it gets `suspended` and releases the underlying thread
    - When the network request returns the result, the computation is resumed
- Such a suspendable computation is called a coroutine
- Coroutines are computations that run on top of threads, can be suspended

> When a coroutine is suspended, the corresponding computation is paused, removed from the thread, stored in memory.
>
> Meanwhile, the the thread is free to be occupied with other activities
>
> When the computation is ready to be continued, it gets returned to a thread(could be different one)

![Coroutines GIF](https://play.kotlinlang.org/resources/hands-on/Introduction%20to%20Coroutines%20and%20Channels/assets/4-suspend/SuspensionProcess.gif)

![Flow](https://play.kotlinlang.org/resources/hands-on/Introduction%20to%20Coroutines%20and%20Channels/assets/4-suspend/SuspendRequests.png)

## Suspend & Resume

- Function with suspend keyword
- When code block encounters function with suspend keyword, the whole block of computation is paused, corresponding
  function is suspended until it gets finished, and removed from thread and stored in memory
- When suspending function finished, the whole block continues its computation

## How to start a coroutines?

- Use one of the main `Coroutine builders : launch, async, or runBlocking`

## Coroutine builders

- Job? Starts a new coroutine
- `async`
    - Returns `Deferred` object
    - `Deferred` is a non-blocking cancellable future. It is a `Job with a result`
        - Stores a computation, Promises the result sometime in the future
    - `await( )`
        - Awaits for completion of this value without blocking thread and resumes when deferred computation is complete
- `launch`
    - Fire and forget
    - Returns a `Job`, which represents the coroutine
    - Possible to wait until it completes by calling `Job.join()`
- Difference between async and launch?
    - `launch is used when starting a computation is not expected to return specific result`
- `runBlocking`
    - Used as a bridge between regular and suspend functions
        - Between blocking and non-blocking worlds
    - Adapter for starting the top-level main coroutine
    - Intended primarily `main( )` and in tests

## Concurrency

- When thread that you specify in coroutineScope is busy, coroutine becomes suspended and scheduled for execution on
  this thread
- The coroutine only resume when the thread is free

- `withContext( )` calls the code with the specified coroutine context, suspends until it completes, and return result

- `If we do not specify which dispatcher, then coroutine follows dispatcher from outer scope's context!`
    - WHY?

## Coroutine Scope & Coroutine Context (Important)

- Coroutine Scope
    - `Responsible for the structure and parent-child relationships between different coroutines`

- Coroutine has to start inside a scope

- Coroutine Context
    - Stores additional technical information used to run a given coroutine
        - Such as `coroutine custom name`, `Dispatcher specifying the threads the coroutine should be scheduled on`

- When using `launch, async and runblocking` to start a coroutine, we automatically create the corresponding scope
    - All these receive lambda and implicit receiver type `coroutieScope`
    - Coroutines can only start in coroutine scope
    - `launch, async` are extensions of CoroutineScope, Implicit and explicit receiver(scope) must be passed
  ```kotlin
  public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
  ): Job {
      val newContext = newCoroutineContext(context)
      val coroutine = if (start.isLazy)
      LazyStandaloneCoroutine(newContext, block) else
      StandaloneCoroutine(newContext, active = true)
      coroutine.start(start, coroutine, block)
      return coroutine
  }
  
  public fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
  ): Deferred<T> {
      val newContext = newCoroutineContext(context)
      val coroutine = if (start.isLazy)
      LazyDeferredCoroutine(newContext, block) else
      DeferredCoroutine<T>(newContext, active = true)
      coroutine.start(start, coroutine, block)
      return coroutine
  }
    ```
    - Coroutine started by `runBlocking` is `the only exception`:
      It is defined as `top-level function, and it blocks current thread`
        - It is intended to use in `main( )` and tests as a bridge function
    - Parent-child relationship works through scopes
        - The child coroutine is started from the scope corresponding the parent coroutine
    - Possible to create a new scope without starting a new coroutine
        - Use of `coroutineScope`
    
Structured Concurrency
    - The mechanism providing the structure of coroutine
    - Benefits
        - Scope is generally responsible for child coroutines, and their lifetime is attached to the
    lifetime of the scope
        - Scope can automatically cancel child coroutines if something goes wrong or user revokes operation
        - Scope automatically waits for completion of all the child coroutines.
            - If the scope corresponds to a coroutine, then the parent coroutine does not complete until all
        the coroutines launched in its scope are complete                
    

# Links

[Introduction to Asynchronous Programming](https://developer.mozilla.org/ko/docs/Learn/JavaScript/Asynchronous/Concepts)

[Asynchronous programming with async and await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)

[Why using Coroutines?](https://kt.academy/article/cc-why)

[How does suspension work in Coroutine](https://kt.academy/article/cc-suspension)

[Coroutines on Android (part I): Getting the background](https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb)

[Mastering Kotlin Coroutines In Android - Step By Step Guide](https://blog.mindorks.com/mastering-kotlin-coroutines-in-android-step-by-step-guide)

[](https://www.baeldung.com/kotlin/coroutines)

[GIF, Images of Coroutines](https://play.kotlinlang.org/resources/hands-on/Introduction%20to%20Coroutines%20and%20Channels)