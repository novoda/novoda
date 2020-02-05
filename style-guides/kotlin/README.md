# The Official Novoda Kotlin Style Guide.

## Baseline

Unless we defined otherwise on this page we are to follow the default [Android Style Guide for Kotlin](https://developer.android.com/kotlin/style-guide).

Additionally, we use [Ktlint](https://ktlint.github.io/) for linting (`ktlint`) and auto-formatting (`ktlintFormat`). 

### Important: 
~~Make sure you run `ktlintFormat` before committing or at the very least before opening a PR.~~

Not ready yet

## Prime directives

 - Rules are made to make our life easier, not harder.
 - If we don't like something, we can change it.
 - If we want to change something, we need objective reasons.

## Conversion Guidelines

 - Aim at planning beforehand to isolate the candidate classes for conversion, this should help asses the associated impact and costs involved with the conversion; this should include unit tests. At this point ensure no other feature teams will be touching the candidates, if so consider whether this is the correct time to convert these classes
 - Use this initial planning to group the conversions into a pull request which will contain only this refactoring; this may include renames, moving objects to more appropriate modules etc. But refrain from adding behavioural changes.
 - Part of the initial planning should also consider the impact on features and therefore the QA effort required to approve the request and it should be included with the feature development so there is only one QA pass to ensure no regression or bugs have been introduced by the conversion; Unit tests should be extended if applicable and the affected feature should be tested which should include a sanity check of any potential dex guard issues by running a QA build. Pairing with a QA team member if necessary during testing will help ensure all uses cases are covered
 - And finaly ensure converted code makes sense using correct Kotlin syntax outlined in this document or other team wide agreed Kotlin guidelines

## Conventions

### Expression vs block body:

Block body (using `{}`) should be used in cases when the return is `Unit` and there is more than one operation.

**YEP**
```koltin
fun onCreate() {
  super.onCreate() // first operation
  setContentView(R.layout.my_layout) // second operation
}
```
**NOPE**
```kotlin
fun onCreate() = super.onCreate().also { 
  setContentView(R.layout.my_layout) 
}
```

### When to specify return type?

Specifying the return type of a function or a property should always be present when they are public or internal. It's ok to not provide it on private ones as the scope should be clear enough to determine. Similarly to how we don't define the type for local properties (inside a function). 

One special case could we if we have a complex operation in a function. However, we should strive for small atomic operations and their combination rather than a long and arduous chain of operations. Especially with extension functions.

### When to use lateinit vs lazy vs property without a field?

#### lateinit
- Using `lateinit` is equivalent to a `@SuppressWarming` as it acknowledges that something can go wrong but we are confident it won't happen. The next person that comes to change your code may not be so confident. 
- Use `lateinit` when a value **will** be used in the future but it has to be initialized at a later time.

#### lazy
- Lazy creates an object that we use to delegate the value so it's evaluated once needed. There shouldn't be more than a handful of `lazy` properties. It could be the case that they are used to create another lazy property, in which case we can extract that to a factory function. 
- If you must use `lazy` and you are not under a multithread environment (as in an Activity creation) avoid locking the class using `lazy(LazyThreadSafetyMode.NONE)`, we might provide an alias for this in the future. 
- Use `lazy(LazyThreadSafetyMode.NONE)` when a field might not be used at all and it's optional.

```kotlin
fun <T> lazySingleThread(initializer: () -> T): Lazy<T> = lazy(LazyThreadSafetyMode.NONE, initializer)
```

#### Property without a field
- Sometimes, if we only need an object once or in infrequent occasions (use clicks) and it's not expensive to obtain then we may be able to use a property without a backing field.

```kotlin
val notificationService get() = app.getNotificationServiceFor(this)
```

### let/also/apply/run/with When to use which?

|  | Receiver(`this`) | Parameter(`it`) || Not extension |
|-|-|-|-|-|
| Returns self | `apply { invoke() }` | `also { it.invoke() }` | | `with(obj) { invoke() }` |
| Returns result | `run { invoke() }` | `let { it.invoke() }` | | |

 - The first row returns the object to which this operation was applied to (subject) irrespectively to what we do inside.
 - The second row returns the result from what we do inside of the provided closure.
 - The fist column takes the subject as the receiver of the closure, we don't have to refer to the subject
 - The second column takes the subject as a parameter instead, so we have to reference `it` or by name is we give it a name inside the lambda.
 - The last column is a special case for `with` which can be considered an alias of `apply`.

Use cases:

 - `apply`: when we are mutating one or multiple properties but we want to continue more operations with the subject (like returning it)
```kotlin
fun State.toLoading() = apply {
  type = Loading
  count = 0
})
```
 - `with`: THis one is, to all effects, an alias of `apply` which takes the subject as a parameter instead of as the receiver of the function. Can be used when creating a type that we want to mutate straight away.
```kotlin
val state = with(State()) {
  type = Loading
  count = 0
}
```
 - `also`: When we want to make clear the name of the object that we are operating on (renaming `it`) or we are inside of a context where `apply` may be ambiguous.
```kotlin
fun State.resetWithOne() = this.toLoading().also { it.count = 1 }
```
In this case, if we use `apply` we have to use `this@also` to refer to the new state.
 - `run`: If we want to convert from a type to another and we have a single action or we want to inherit properties from the same object.
```kotlin
val fullname = user.run { "$name $surname" }
```
With `name` and `surname` being properties of `user`. Here we save having to name the subject of the extension or use `it`.
 - `let`: Sort of an alias for `map` and `flatmap` but for a single and unwrapped object. Useful to change the order or requirements.
```kotlin
val userRank = user.let { rankFor(it.id) }
```
Saves having to make a redundant property, similar to what we do with `nullable`. Instead of doing:

```
if(input != null) {
  val trullyNotNull = input!!
  return createFromNotNull(trullyNotNull)
} 
```
We can do:
```kotlin
input?.let { createFromNotNull(it) }
```

This is especially important when consuming properties that aren't local as we have no warranties that the value in `input` will remain not null after we checked it. With `let` here we can grab a copy of the current value and deal with it on-site.

**Note:**

These functions should be used when we are using the receiver of the function inside of the closure, otherwise a simple if-else or where statement is more readable.
Do not use `this` keyword inside `apply`, `with`, and `run`. - because `this` can be confusing inside there 
Only use `?.let` if you use consume the `it` inside, otherwise just use `if`.
 
**Note 2:**

All these functions (but specially `run`) can be run without an explicit receiver, in which case the receiver is the object we are in a particular scope. 
 
```kotlin
class Somer {
  val value = run { // this here is Somer
    val dependencyOne = resolveDependency(One)
    val dependencyTwo = resolveDependency(Two)
    createValue(dependencyOne, dependencyTwo)
  }
}
```


### Extension functions location

Where do we put extension functions depends on two factors: Does it change state and how reusable is it.

 - Inside a type as private:
If we have an extension function that changes the internal state of an object (not the one being extended) it should always live inside of the object.

 - In the file as private:
The logic of the function is only relevant to one or many of the types defined in that file

 - In a file as public:
Only if the file contains just extension functions. When do we extract them to an extension file? This should come to on-time judgment. Each situation may be different. However, the golden rule is that there is some functionality that can be used by more than one component.

### Companion object vs file definitions (properties and functions)

Properties and functions that are required across multiple **instances** can live in the file (as private). Rationale: We avoid creating a new object as when defining a companion object we effectively doing that.

Properties and functions that are required across multiple **types** can live in the companion type *if* we require namespacing for that particular type. This is especially important when declaring a factory function that creates instances of a given type: `MyComponent.from()`.

If we declare something on a file (type, function or property) it belongs to the package in Kotlin and to the class generated for Java (MyFileKt).

### When to use aliases and where should they be declared?

Cases were we can use type aliases:
  - Name clash from an external library (as opposed to an import alias)
  - Generics make a type too long.

Case to **NOT** use type aliases:
  - To shadow the type of a function. For example:
```kotlin
typealias UserProcessor = (User) -> Unit
```
This adds cognitive overhead and adds no more readability.

### Functions and Constructors:

For both constructors and functions, if there is more than 2 parameter then we put each on their own line. Both for their definition and their usage.

**YEP**
```kotlin
data class User(
  val name: String,
  val surname: String,
  val dab: Date
)
```
**NOPE**
```kotlin
data class User(val name: String,
                val surname: String,
                val dab: Date)
```
(Look at all that space, one could build a cathedral in there.)

*Rational:* This makes it clear what parameters do we have available (with long names we may miss the first one) and makes fewer changes (on PRs) when we want to do refactors like renaming or adding/removing parameters. A rename of the type would shift every line of the constructor.

### Factory functions:

Use named factory functions (`newInstance`, `create`, etc. ) either on the `companion` object or in a separate `*Factory` file.

Kotlin allows defining `operator fun invoke()` function that can be called without `invoke` just by using the brackets `()`. This approach for creating factory methods can lead to subtle accidental behaviour that can be really hard to understand. 

Consider the following (contrived) example. It's not obvious which method is called when a new user instance is created.
```
class User internal constructor() {

    companion object {
        operator fun invoke() = User()
    }
}

val user = User()
```

As opposed to this approach:
```
class User internal constructor() {

    companion object {
        fun newInstance() = User()
    }
}

val user = User.newInstance()
```

### Data classes:

  - The constructor should not be private, even if unused. It's not possible to hide it completely as data classes generate `copy` functions. There is a conversation about this here:  https://youtrack.jetbrains.com/issue/KT-11914

#### Where to declare them, nested or at the same level?

This depends on its visibility. If we have a `private` sealed class then it is safe to have them in the same level, for external usage then we should nest them to avoid name clashing.

## Functions as parameters

**NOPE**
Do not pass a function as a parameter in a class constructor if there is only one implementation of the function that you are passing. 
- This makes is complex to follow up a call hierarchy when there is a bug in the code
- Is it not clear how many different implementations or providers of that function are in the code

**YES**
Pass functions as params in other functions or local private internal classes in the same file

## Interfaces

**NOPE**
Do not use platform-system interfaces as parameters in class constructor except for data structures. It makes it impossible to find out who is implementing what you are passing.

Do not create an interface that has just one implementation if it's not strictly necessary. 
- Adds one more level of indirection that's unnecessary

Avoid system-wide use of interfaces with hundreds of different implementations, like `Marshaller`. This is similar in approach to `do not use platform-system` interfaces.

**YES**
There are different valid implementation and you want to make sure that those implementations are easily swappable.
