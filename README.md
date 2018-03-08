# Kotlin + Λrrow  for FP Scala Devs

---

## Lang Constructs

---

### Declarations

```
Scala       |       Kotlin
------------|--------------------
val         |       val
var         |       var
object      |       object
trait       |       interface
class       |       class
def         |       fun
trait       |       interface
+           |       out
-           |       in
[A]         |       <A>
type        |       typealias
match       |       when
-           |       suspend (non blocking F<A> -> A)
-           |       with (scoped syntax)
```

---

### Functions

Scala

```scala
object {
    def hello[A](a: A): String = s"hello $a"
}
```

Kotlin

```kotlin
fun <A> hello(a: A): String = "hello $a"
```

---

### Nullable types (Lang backed `Option`)

Scala

```scala
val maybeString: Option<String> = None
```

Kotlin

```kotlin
val maybeString: String? = null
```

---

### Data classes

Scala

```scala
case class Person(name: String, age: Int)
```

Kotlin

```kotlin
data class Person(val name: String, val age: Int)
```

---

### Sealed classes

Scala

```scala
sealed abstract class ErrorType
case object MyError1 extends ErrorType
case object MyError2 extends ErrorType
case class MyError3(underlying: Throwable) extends ErrorType
```

Kotlin

```kotlin
sealed class ErrorType
object MyError1 : ErrorType()
object MyError2 : ErrorType()
data class MyError3(val underlying: Throwable): ErrorType
```

---

### Pattern Matching

Scala

```scala
errorType match {
    case MyError1 => ???
    case MyError2 => ???
    case MyError3(ex) => throw ex
}
```

Kotlin

```kotlin
when (errorType) {
    is MyError1 => TODO()
    is MyError2 => TODO()
    is MyError3 => throw errorType.underlying //note smart cast
}
```

---

### object

Scala

```scala
object MySingleton
```

Kotlin

```kotlin
object MySingleton
```

---

### Companion object

Scala

```scala
case class Person(name: String, age: Int)
object Person {...}
```

Kotlin

```kotlin
data class Person(val name: String, val age: Int) {
    companion object { ... }
}
```

---

### Type aliases

Scala

```scala
type X = String
```

Kotlin

```kotlin
typealias X = String
```

---

### Path Dependent Types

Scala 

```scala
abstract class Foo[A] {
  type X = A
}
```

Kotlin

```kotlin
abstract class Foo[A] {
  typealias X = A // will not compile
}
```

---

### Syntax Extensions

Scala

```scala
implicit class StringOps(value: String): AnyVal {
    def quote: String = s"--- $value"
}
```

Kotlin

```kotlin
fun String.quote(): String = "--- $value"
```

---

### Variance

Scala

```scala
class Option[+A] //A is covariant
class Functio1[-I, +O] // I is contravariant
class Leaf[A] // A is invariant
```

Kotlin

```kotlin
class Option<out A> //A is covariant
class Functio1n<in I, out O> // I is contravariant
class Leaf<A> // A is invariant
```

---

### Variance constrains

Scala

```scala
def run[A, B <: A]: Nothing = ???
def run[A, B >: A]: Nothing = ??? //No Kotlin equivalent
```

Kotlin

```kotlin
fun <A, B : A> run(): Nothing = TODO()
```

---

### FP Libraries

Scala : cats, scalaz
Kotlin: Λrrow

---

### Higher Kinded Types

Scala

```scala
class Service[F[_]]
```

Kotlin

```kotlin
class Service<F> // No way to express kind shape F<_>
```

---

### Higher Kinded Types Emulation

Scala

```scala
class Option[+A]
```

Kotlin

```kotlin
class ForOption private constructor()

typealias OptionOf<A> = arrow.Kind<ForOption, A>

inline fun <A> OptionOf<A>.fix(): Option<A> = this as Option<A>
class Option<out A> : Kind<OptionOf<A>>
```

---

### Higher Kinded Types Emulation

Kotlin

```kotlin
import arrow.higherkind

@higherkind class Option<out A> : OptionOf<A>
```

---

### Implicit Resolution

Scala

```scala
import cats.Functor

class Service[F[_]](implicit F: Functor[F]) {
    def doStuff: F[String] = F.pure("Hello Tagless World")
}

new Service[Option]
```

---

### Implicit Resolution (Runtime)

Kotlin

```kotlin
import javax.inject
import arrow.*
import arrow.typeclasses.Functor

@typeclass interface Service<F> : TC {

    fun FF(): Functor<F>

    fun doStuff(): Kind<F, String> = FF().pure("Hello Tagless World")
}

val result: OptionOf<String> = service<ForOption>().doStuff()
van normalizedResult: Option<String> = result.fix()
```

---

### Implicit Resolution (Compile time)

Kotlin

```kotlin
import javax.inject
import arrow.*
import dagger.*
import arrow.dagger.instances.*
import arrow.typeclasses.Functor

class Service<F> @Inject constructor(val FF: Functor<F>) {
    fun doStuff(): Kind<F, String> = FF.pure("Hello Tagless World")
}

@Singleton
@Component(modules = [ArrowInstances::class])
interface Runtime {
    fun optionService(): Service<ForOption>
    companion object {
        val implicits : Implicits = DaggerRuntime.create()
    }
}

val result: OptionOf<String> = Runtime.optionService().doStuff()
van normalizedResult: Option<String> = result.fix()
```

---

### Higher Kinded Types Partial application

Scala

```scala
Monad[Either[String, ?]]
```

Kotlin

```kotlin
Monad<EitherPartialOf<String>> 
//Partial extensions are generated by @higherkind
```

---

### Cartesian Builder

Scala

```scala
import cats.implicits._

case class Result(n: Int, s: String, c: Character)

(Option(1), Option("2"), Option('3')).mapN { case (n, s, c) =>
    Result(n, s, c)
}
```

---

### Cartesian Builder

Kotlin

```kotlin
import arrow.*
import arrow.typeclasses.*

data class Result(val n: Int, val s: String, val c: Character)

Option.applicative().map(Option(1), Option("2"), Option('3'), {
    Result(n, s, c)
})
```

---

### Cartesian Builder

Kotlin

```kotlin
import arrow.*
import arrow.typeclasses.*

@generic
data class Result(val n: Int, val s: String, val c: Character)

Option.applicative().mapToResult(Option(1), Option("2"), Option('3'))
// Option(Result(1, "2", '3'))
```

---

### Monad Comprehensions

Scala

```scala
for {
    a <- Option(1)
    b <- Option(1)
    x <- Option(1)
} yield a + b + c
//Option(3)
```

---

### Monad Comprehensions

Kotlin

```kotlin
import arrow.*
import arrow.typeclasses.*

Option.monad().binding {
    val a = Option(1).bind()
    val b = Option(1).bind()
    val c = Option(1).bind()
    a + b + c
}
//Option(3)
```

---

### Arrow Monad Comprehensions

Built atop Kotlin Coroutines

```kotlin
suspend fun <B> bind(m: () -> Kind<F, B>): B = suspendCoroutineOrReturn { c ->
    val labelHere = c.stackLabels // save the whole coroutine stack labels
    returnedMonad = flatMap(m(), { x: B ->
        c.stackLabels = labelHere
        c.resume(x)
        returnedMonad
    })
    COROUTINE_SUSPENDED
}
```

---

### Arrow Monad Comprehensions

`binding`

```kotlin
import arrow.*
import arrow.typeclasses.*

Try.monad().binding {
    val a = Try { 1 }.bind()
    val b = Try { 1 }.bind()
    a + b 
}
//Success(2)
```

---

### Arrow Monad Comprehensions

`binding`

```kotlin
import arrow.*
import arrow.typeclasses.*

Try.monad().binding {
    val a = Try { 1 }.bind()
    val b = Try { 1 }.bind()
    a + b 
}
//Success(2)
```

---

### Arrow Monad Comprehensions

`bindingCatch`

```kotlin
import arrow.*
import arrow.typeclasses.*

Try.monad().bindingCatch {
    val a = Try { 1 }.bind()
    val b = Try<Int> { throw RuntimeException("BOOM") }.bind()
    a + b 
}
//Failure(RuntimeException(BOOM))
```

---

### Arrow Monad Comprehensions

`bindingStackSafe`

```kotlin
import arrow.*
import arrow.typeclasses.*

val tryMonad = Try.monad()

fun stackSafeTestProgram(n: Int, stopAt: Int): Free<ForTry, Int> = 
    tryMonad.bindingStackSafe {
        val v = Try {n + 1}.bind()
        val r = if (v < stopAt) stackSafeTestProgram(M, v, stopAt).bind() else Try { v }.bind()
        r
    }

stackSafeTestProgram(0, 50000).run(tryMonad)
//Success(50000)
```

---

### Arrow Monad Comprehensions

`binding(context: CoroutineContext)`

```kotlin
import arrow.*
import arrow.typeclasses.*

Try.monad().binding(UIThread) { //flatMap execution happens in the UI thread
    val a = IO { draw(1) }.bind()
    val b = IO { draw(1) }.bind()
    a + b 
}
```

---

### Arrow already includes most things you need for FP

- Basic Data Types (Try, Eval, Option, NonEmptyList, Validated, ...)
- FP Type classes (Functor, Applicative, Monad...)
- Optics (Lenses, Prisms, Iso...)
- Generic (.tupled(), .tupleLabelled(), HListN, TupleN...)
- Effects (Async, Effect, IO, DeferredK, ObservableK...)
- MTL (FunctorFilter, TraverseFilter...)
- Free (Free, FreeApplicative...)
- Integrations (Kotlin Coroutines async/await, Rx2)
- Recursion Schemes

---

### Side projects

- [Ank](https://github.com/arrow-kt/ank): Doc type checker like `tut`
- [Helios](https://github.com/47deg/helios): Json lib: Jawn port and based on arrow optics for its DSL
- [Kollect](https://github.com/47deg/kollect): Fetch port
- [Bow](https://github.com/arrow-kt/ank): Arrow for Swift

---

### Thanks for watching!

- [@raulraja](https://twitter.com/raulraja)
- [@47deg](https://twitter.com/47deg)
- [Λrrow](http://arrow-kt.io/)
- [Λrrow Tutorial Videos](https://www.youtube.com/watch?v=3y9KI7XWXSY&list=PLTx-VKTe8yLzGJLMQ3TKSsGezeVj2ehhb)