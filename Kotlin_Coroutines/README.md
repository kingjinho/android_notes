# Asynchronous Programming

# Concept of Asynchronous Programming

- These days, we have multiple processor cores
- Basis of asynchronous programming
    - `Why should wait for something when we could do something else?`

# How did we implement Asynchronous Programming?

- Creating Threads
- Callback

# Why using Coroutines?
- Concurrency
  - Long time ago, Computer can do only one thing
    - `When downloading, just download`
  - Without concurrency -> resource waste
- Parallelism
  - SMP
    - Symmetric Multiprocessor
        - Multiple processes use one memory
        - Multiple processes work at once
    - Problem with SMP
        - Visibility: Multiple CPU using separate cache -> Lock, memory barrier
        - Cache = Collection of cache line(64 or 128 byte)
            - Multiple cores cannot access to same cache line to update data at once
                - Has to wait

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
- `When code block encounters function with suspend keyword, the whole block of computation is paused, corresponding
  function is suspended until it gets finished, and removed from thread and stored in memory`
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

- CoroutineScope
    - `Responsible for the structure and parent-child relationships between different coroutines`
    - `Start of every coroutine`

- Coroutine has to start inside a scope

- Coroutine Context
    - Stores additional technical information used to run a given coroutine
        - Such as `coroutine custom name`, `Dispatcher specifying the threads the coroutine should be scheduled on`

- When using `launch, async and runblocking` to start a coroutine, we automatically create the corresponding scope
    - All these receive lambda and implicit receiver type `coroutineScope`
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
    - Scope is generally responsible for child coroutines, and their lifetime is attached to the lifetime of the scope 
    - Scope can automatically cancel child coroutines if something goes wrong or user revokes operation 
    - Scope automatically waits for completion of all the child coroutines. 
    - If the scope corresponds to a coroutine, then the parent coroutine does not complete until all the
coroutines launched in its scope are complete 
    - The new scope created by the `coroutineScope` or by the coroutine
builders, always inherits the context from the outer scope 
    - All the nested coroutines are automatically started with
the inherited context - And the dispatcher is a part of this context 
    - With structured concurrency, we can specify major
context elements(like dispatcher) once, when we create a top-level coroutine. 
      All the nested coroutines then inherit the
context and modify it only if needed 
    - In Android, it is common practice to use `Main` by default for the top coroutine.

- And then to explicitly put a different dispatcher when we need to run code on different thread

- Channels
    - `Coroutines can communicate with each other via channels`
    - It is communication primitives that allows us to pass data between different coroutines
    - One can send information while the other can receive it
    - Producer: coroutine that sends information
    - Consumer: coroutine that receives information
    - N : N relationship

    - When many coroutines receive information from the same channel, each element is handled only by one of the
      consumers
        - Handling it automatically means removing this element from the channel
    - Channel is similar to a collection of elements(queue)
        - However, the main different is that unlike collections, even in synchronized versions
          a `channel can suspend send and receive` operations, depending on whether channel is empty or full
    - Channel is represented by 3 interfaces
        - `sendChannel`
            - Create a channel and give it to coroutine as `SendChannel` instance
            - Send only
            - Can close a channel to indicate no more elements are coming
        - `receiveChannel`
            - Create a channel and give it to coroutine as `ReceiveChannel` instance
            - Receive only
        - `Channel` which extends the first two
    - Coroutines library has several types of channels
        - Differ in how many elements they can internally store, and whether the send call can suspend or not
    - For all channel types, the `receive` call behaves in the same manner
        - It receives when the channel is not empty, otherwise suspends
    - Types of Channel
        - Unlimited Channel
            - The closest analog to queue
            - Producer can send elements to channel, and channel will grow infinitely
            - `send` call will never be suspended
            - If no memory, `OutOfMemory Exception`
            - Difference with a queue?
                - When consumer tries to receive from an empty channel and suspended until new elements are sent
        - Buffered Channel
            - Has size constraint by number
            - Producer can send element until channel reaches its maximum capacity
            - All elements are internally stored
            - When full, next send call will be suspended until channel has free space
        - Rendezvous Channel
            - A channel without buffer
            - Channel with size of zero
            - Either `send or receive will always be suspended` until the other is called
                - If `send` is called and no suspended `receive` call ready to process, then `send` suspend
                - If `receive` is called and the channel is empty(meaning no suspended send call), then `receive` suspend
            - `Both send and receive should meet on time`
        - Conflated Channel
            - A new element sent to this channel will overwrite the previously sent elements
                - Thus, receiver only gets the latest element
            - `send` call will never suspended
          
        - Example of Instantiating Channel
         ```kotlin
        //When we take a lookg at Channel, by default it is rendezvous
            val rendezvousChannel = Channel<String>()
            val bufferedChannel = Channel<String>(10)
            val conflatedChannel = Channel<String>(CONFLATED)
            val unlimitedChannel = Channel<String>(UNLIMITED)
        ```
        

# Links

[Introduction to Asynchronous Programming](https://developer.mozilla.org/ko/docs/Learn/JavaScript/Asynchronous/Concepts)

[Asynchronous programming with async and await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)

[Why using Coroutines?](https://kt.academy/article/cc-why)

[How does suspension work in Coroutine](https://kt.academy/article/cc-suspension)

[Coroutines on Android (part I): Getting the background](https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb)

[Mastering Kotlin Coroutines In Android - Step By Step Guide](https://blog.mindorks.com/mastering-kotlin-coroutines-in-android-step-by-step-guide)

[](https://www.baeldung.com/kotlin/coroutines)

[GIF, Images of Coroutines](https://play.kotlinlang.org/resources/hands-on/Introduction%20to%20Coroutines%20and%20Channels)
