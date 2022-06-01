# Dagger

## What is dagger?

- a DI library for Android
- Create a graph of dependencies(application graph), generate DI generated code
- manage dependencies
- use Java's annotation model
    - generates code at compile-time using annotation processor(kapt plugin)

```groovy
    //contains all annotations
implementation "com.google.dagger:dagger:$dagger_version"
//annotation processor that generate the code
kapt "com.google.dagger:dagger-compiler:$dagger_version"
```    

## @Inject

- build application graph automatically
- `tells Dagger what dependency certain class has how to create instances of type 'Foo'`

### Constructor Injection

```kotlin
class Foo @Inject constructor(val dep: dependency) {

    @Inject
    lateinit var bar: Bar
}
```

- Since certain android framework classes(activity, fragments) are instantiated by system, Dagger cannot create these.
    - meaning that we cannot use `@inject constuctor(..)` of a View class
    - Use `field injection` to populate dependencies

### Field Injection

```kotlin
class FooActivity : AppCompatActivity() {
    @Inject
    lateinit var viewModel: BarViewModel

}
```

- commonly used in Activities and Fragments

### Injection in constructor & field

- constructor
    - `tells Dagger how to provide instances of that class`
- field
    - `tells Dagger that it needs to populate the field with an instance of that type`

> So, @Inject in either constructor and field tells Dagger how to provide dependencies
> But How do we say **which objects need to be injected**?

## @Component

> Q: How do we tell Dagger oversees(create, manage ...) graph of dependencies?
>
> A: Create an interface and annotate it with @Component

```kotlin
@Component
interface AppContainer {
    // classes that can be injected by this graph
    // with this, We tell dagger that foo needs di and 
    // Dagger has to provide the dependencies which are annotated with @Inject
    // Dagger creates dependencies that foo needs internally
    fun inject(foo: ClassThatHasDependency)
}
```

- Creates a Container
- An interface with @Component annotation will do the following:
    - Gives information Dagger needs to generate the graph at compile-time
    - parameter of interface methods: classes that request injection
    - `Make dagger generate code with all dependencies required to satisfy the parameters of the methods it exposes`

> classes, fine, use @Inject either at constructor or field, and @Component to tell dagger to
> create a graph of dependencies. What about interfaces?

## @Module, @Binds and @BindsInstance annotations

- As we know, an interface cannot be instantiated directly
- Dagger needs to know `what implementation of an interface` we are going to use

> How?

### How?

- `Need Dagger Module`
- `Dagger Module tells Dagger about how to provide instances of a certain type`
- Define dependencies with @Provides and @Binds annotations
- example

```kotlin
interface Repo {

}

//without dagger module, build will result in failure
class SomeViewModel @Inject constructor(val repo: Repo) {

}

@Module
abstract class DaggerModule {

    /**
     * tells dagger which implementation needs to use when providing an interface
     * must annotate an abstract function
     * return type: interface we want to provide an implementation for(SomeRepo)
     * parameter: implementation of an interface we want to provide(Repo)
     * 
     * "When you need an Repo object, use SomeRepo"
     */
    @Binds
    abstract fun provideRepo(repo: SomeRepo): Repo
}

class SomeRepo : Repo() {

}
```
- Modules are a way to encapsulate how to provide objects
  - `group the logic of providing objects related to storage`

- Until now, we tell Dagger that whenever `Repo` object is needed, it should create and provide an 
  instance of `SomeRepo`
- However, we haven't told Dagger on how to create an instance of `SomeRepo`
  - to do that
    ```kotlin
    class SomeRepo @Inject constructor() : Repo() {
        
    }
    ```
  - Now Dagger knows how to create SomeRepo object

- Still, AppContainer with @Component does not know the existence of Dagger Module
  - To tell which module the app has to use,
```kotlin
@Module(modules = [DaggerModule::class])
interface AppContainer {
    fun inject(foo: ClassThatHasDependency)
}
```