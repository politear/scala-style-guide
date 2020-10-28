# Zignal Labs Scala Guide

## DRAFT! THIS IS IN PROGRESS!!

This guide was adapted from the [Databricks Style Guide](https://github.com/databricks/scala-style-guide) and modified to suite Zignal Labs' specific needs. 

Code is __written once__ by its author, but __read and modified multiple times__ by lots of other engineers. As most bugs actually come from future modification of the code, we need to optimize our codebase for long-term, global readability and maintainability. The best way to achieve this is to write simple code.

Scala is an incredibly powerful language that is capable of many paradigms. We have found that the following guidelines work well for us on projects with high velocity. Depending on the needs of your team, your mileage might vary.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.


## <a name='TOC'>Table of Contents</a>

1. [Document History](#history)

2. [Syntactic Style](#syntactic)
    * [Naming Convention](#naming)
    * [Variable Naming Convention](#variable-naming)
    * [Line Length](#linelength)
    * [Rule of 30](#rule_of_30)
    * [Spacing and Indentation](#indent)
    * [Blank Lines (Vertical Whitespace)](#blanklines)
    * [Parentheses](#parentheses)
    * [Curly Braces](#curly)
    * [Long Literals](#long_literal)
    * [Documentation Style](#doc)
    * [Imports](#imports)
    * [Pattern Matching](#pattern-matching)
    * [Infix Methods](#infix)
    * [Anonymous Methods](#anonymous)

1. [Scala Language Features](#lang)
    * [Case Classes and Immutability](#case_class_immutability)
    * [apply Method](#apply_method)
    * [override Modifier](#override_modifier)
    * [Destructuring Binds](#destruct_bind)
    * [Call by Name](#call_by_name)
    * [Multiple Parameter Lists](#multi-param-list)
    * [Symbolic Methods (Operator Overloading)](#symbolic_methods)
    * [Type Inference](#type_inference)
    * [Return Statements](#return)
    * [Recursion and Tail Recursion](#recursion)
    * [Implicits](#implicits)
    * [Exception Handling (Try vs try)](#exception)
    * [Options](#option)
    * [Symbol Literals](#symbol)

1. [Testing](#testing)
    * [Intercepting Exceptions](#testing-intercepting)

1. [Miscellaneous](#misc)
    * [Prefer nanoTime over currentTimeMillis](#misc_currentTimeMillis_vs_nanoTime)
    * [Prefer existing well-tested methods over reinventing the wheel](#misc_well_tested_method)



## <a name='history'>Document History</a>
- 2015-03-16: Initial version.
- 2015-05-25: Added [override Modifier](#override_modifier) section.
- 2015-08-23: Downgraded the severity of some rules from "do NOT" to "avoid".
- 2015-11-17: Updated [apply Method](#apply_method) section: apply method in companion object should return the companion class.
- 2015-11-17: This guide has been [translated into Chinese](README-ZH.md). The Chinese translation is contributed by community member [Hawstein](https://github.com/Hawstein). We do not guarantee that it will always be kept up-to-date.
- 2015-12-14:  This guide has been [translated into Korean](README-KO.md). The Korean translation is contributed by [Hyukjin Kwon](https://github.com/HyukjinKwon) and reviewed by [Yun Park](https://github.com/yunpark93), [Kevin (Sangwoo) Kim](https://github.com/swkimme), [Hyunje Jo](https://github.com/RetrieverJo) and [Woochel Choi](https://github.com/socialpercon). We do not guarantee that it will always be kept up-to-date.
- 2016-06-15: Added [Anonymous Methods](#anonymous) section.
- 2016-06-21: Added [Variable Naming Convention](#variable-naming) section.
- 2016-12-24: Added [Case Classes and Immutability](#case_class_immutability) section.
- 2017-02-23: Added [Testing](#testing) section.
- 2017-04-18: Added [Prefer existing well-tested methods over reinventing the wheel](#misc_well_tested_method) section.
- 2019-12-18: Added [Symbol Literals](#symbol) section.
- 2020-10-27: Made modifications for Zignal Labs

## <a name='syntactic'>Syntactic Style</a>

### <a name='naming'>Naming Convention</a>

We mostly follow Java's and Scala's standard naming conventions.

- Classes, traits, objects should follow Java class convention, i.e. PascalCase style.
  ```scala
  class ClusterManager

  trait Expression
  ```

- Packages should follow Java package naming conventions, i.e. all-lowercase ASCII letters.
  ```scala
  package com.zignallabs.resourcemanager
  ```

- Methods/functions should be named in camelCase style.

- Constants should be all uppercase letters and be put in a companion object.
  ```scala
  object Configuration {
    val DEFAULT_PORT = 10000
  }
  ```

- Methods/functions should always have return types.
  ```scala
  // Correct
  def getItems(): List[Int] = {}
  
  // Wrong
  def getItems() = {}
  ```

- An enumeration class or object which extends the `Enumeration` class shall follow the convention for classes and objects, i.e. its name should be in PascalCase style. Enumeration values shall be in the upper case with words separated by the underscore character `_`. For example:
  ```scala
    private object ParseState extends Enumeration {
    type ParseState = Value

    val PREFIX,
        TRIM_BEFORE_SIGN,
        SIGN,
        TRIM_BEFORE_VALUE,
        VALUE,
        VALUE_FRACTIONAL_PART,
        TRIM_BEFORE_UNIT,
        UNIT_BEGIN,
        UNIT_SUFFIX,
        UNIT_END = Value
  }
  ```

- Annotations should also follow Java convention, i.e. PascalCase. Note that this differs from Scala's official guide.
  ```scala
  final class MyAnnotation extends StaticAnnotation
  ```


### <a name='variable-naming'>Variable Naming Convention</a>

- Variables should be named in camelCase style, and should have self-evident names.
  ```scala
  val serverPort = 1000
  val clientPort = 2000
  ```

- The excpetion to this rule is for members of classes that get serialized, e.g. during into json or get sent to Elasticsearch. These names will use snake_case.

- It is OK to use one-character variable names in small, localized scope. For example, "i" is commonly used as the loop index for a small loop body (e.g. 10 lines of code). However, do NOT use "l" (as in Larry) as the identifier, because it is difficult to differentiate "l" from "1", "|", and "I".

### <a name='linelength'>Line Length</a>

- Limit lines to 120 characters.
- The only exceptions are import statements and URLs (although even for those, try to keep them under 120 chars).


### <a name='rule_of_30'>Rule of 30</a>

"If an element consists of more than 30 subelements, it is highly probable that there is a serious problem" - [Refactoring in Large Software Projects](http://www.amazon.com/Refactoring-Large-Software-Projects-Restructurings/dp/0470858923).

In general:

- A method should contain less than 30 lines of code.
- A class should contain less than 30 methods.


### <a name='indent'>Spacing and Indentation</a>

- Put one space before and after operators, including the assignment operator.
  ```scala
  def add(int1: Int, int2: Int): Int = int1 + int2
  ```

- Put one space after commas.
  ```scala
  Seq("a", "b", "c") // do this

  Seq("a","b","c") // don't omit spaces after commas
  ```

- Put one space after colons.
  ```scala
  // do this
  def getConf(key: String, defaultValue: String): String = {
    // some code
  }

  // don't put spaces before colons
  def calculateHeaderPortionInBytes(count: Int) : Int = {
    // some code
  }

  // don't omit spaces after colons
  def multiply(int1:Int, int2:Int): Int = int1 * int2
  ```

- Use 2-space indentation in general.
  ```scala
  if (true) {
    println("Wow!")
  }
  ```

- For method declarations, use 4 space indentation for their parameters and put each in each line when the parameters don't fit in two lines. Return types should be on the same line as the last parameter.

  ```scala
  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration): RDD[(K, V)] = {
    // method body
  }
  ```

- For classes whose header doesn't fit in two lines, use 4 space indentation for its parameters, put each in each line, put the extends on the next line with 2 space indent, and add a blank line after class header.

  ```scala
  class Foo(
      val param1: String,  // 4 space indent for parameters
      val param2: String,
      val param3: Array[Byte])
    extends FooInterface  // 2 space indent here
    with Logging {

    def firstMethod(): Unit = { ... }  // blank line above
  }
  ```

- For method and class constructor invocations, use 2 space indentation for its parameters and put each in each line when the parameters don't fit in two lines.

  ```scala
  foo(
    someVeryLongFieldName,  // 2 space indent here
    andAnotherVeryLongFieldName,
    "this is a string",
    3.1415)

  new Bar(
    someVeryLongFieldName,  // 2 space indent here
    andAnotherVeryLongFieldName,
    "this is a string",
    3.1415)
  ```

- Do NOT use vertical alignment. They draw attention to the wrong parts of the code and make the aligned code harder to change in the future.
  ```scala
  // Don't align vertically
  val plus     = "+"
  val minus    = "-"
  val multiply = "*"

  // Do the following
  val plus = "+"
  val minus = "-"
  val multiply = "*"
  ```


### <a name='blanklines'>Blank Lines (Vertical Whitespace)</a>

- A single blank line appears:
  - Between consecutive members (or initializers) of a class: fields, constructors, methods, nested classes, static initializers, instance initializers.
    - Exception: A blank line between two consecutive fields (having no other code between them) is optional. Such blank lines are used as needed to create logical groupings of fields.
  - Within method bodies, as needed to create logical groupings of statements.
  - Optionally before the first member or after the last member of the class (neither encouraged nor discouraged).
- Use one blank line to separate class or object definitions.
- Excessive number of blank lines is discouraged.


### <a name='parentheses'>Parentheses</a>

- Methods should be declared with parentheses, unless they are accessors that have no side-effect (state mutation, I/O operations are considered side-effects).
  ```scala
  class Job {
    // Wrong: killJob changes state. Should have ().
    def killJob: Unit

    // Correct:
    def killJob(): Unit
  }
  ```
- Callsite should follow method declaration, i.e. if a method is declared with parentheses, call with parentheses.
  Note that this is not just syntactic. It can affect correctness when `apply` is defined in the return object.
  ```scala
  class Foo {
    def apply(args: String*): Int
  }

  class Bar {
    def foo: Foo
  }

  new Bar().foo  // This returns a Foo
  new Bar().foo()  // This returns an Int!
  ```


### <a name='curly'>Curly Braces</a>

Put curly braces even around one-line conditional or loop statements. The only exception is if you are using if/else as an one-line ternary operator that is also side-effect free.
```scala
// Correct:
if (true) {
  println("Wow!")
}

// Correct:
if (true) statement1 else statement2

// Correct:
try {
  foo()
} catch {
  ...
}

// Wrong:
if (true)
  println("Wow!")

// Wrong:
try foo() catch {
  ...
}
```


### <a name='long_literal'>Long Literals</a>

Suffix long literal values with uppercase `L`. It is often hard to differentiate lowercase `l` from `1`.
```scala
val longValue = 5432L  // Do this

val longValue = 5432l  // Do NOT do this
```


### <a name='doc'>Documentation Style</a>

Use Scaladoc style.
```scala
/** This is a correct one-liner, short description. */

/**
  * This is correct multi-line ScalaDoc comment. And
  * this is my second line, and if I keep typing, this would be
  * my third line.
  */
```

### <a name='imports'>Imports</a>

- __Avoid using wildcard imports__, unless you are importing more than 6 entities, or implicit methods. Wildcard imports make the code less robust to external changes.
- Always import packages using absolute paths (e.g. `scala.util.Random`) instead of relative ones (e.g. `util.Random`).
- In addition, sort imports in the following order:
  * `java.*` and `javax.*`
  * `scala.*`
  * Third-party libraries (`org.*`, `com.*`, etc)
  * Project classes (`com.databricks.*` or `org.apache.spark` if you are working on Spark)
- Within each group, imports should be sorted in alphabetic ordering.
- You can use IntelliJ's import organizer to handle this automatically, using the following config:

  ```
  java
  javax
  _______ blank line _______
  scala
  _______ blank line _______
  all other imports
  _______ blank line _______
  com.databricks  // or org.apache.spark if you are working on Spark
  ```


### <a name='pattern-matching'>Pattern Matching</a>

- For method whose entire body is a pattern match expression, put the match on the same line as the method declaration if possible to reduce one level of indentation.
  ```scala
  def test(msg: Message): Unit = msg match {
    case ...
  }
  ```

- When calling a function with a closure (or partial function), if there is only one case, put the case on the same line as the function invocation.
  ```scala
  list.zipWithIndex.map { case (elem, i) =>
    // ...
  }
  ```
  If there are multiple cases, indent and wrap them.
  ```scala
  list.map {
    case a: Foo =>  ...
    case b: Bar =>  ...
  }
  ```

- If the only goal is to match on the type of the object, do NOT expand fully all the arguments, as it makes refactoring more difficult and the code more error prone.
  ```scala
  case class Pokemon(name: String, weight: Int, hp: Int, attack: Int, defense: Int)
  case class Human(name: String, hp: Int)
  
  // Do NOT do the following, because
  // 1. When a new field is added to Pokemon, we need to change this pattern matching as well
  // 2. It is easy to mismatch the arguments, especially for the ones that have the same data types
  targets.foreach {
    case target @ Pokemon(_, _, hp, _, defense) =>
      val loss = sys.min(0, myAttack - defense)
      target.copy(hp = hp - loss)
    case target @ Human(_, hp) =>
      target.copy(hp = hp - myAttack)
  }
  
  // Do this:
  targets.foreach {
    case target: Pokemon =>
      val loss = sys.min(0, myAttack - target.defense)
      target.copy(hp = target.hp - loss)
    case target: Human =>
      target.copy(hp = target.hp - myAttack)
  }
  ```


### <a name='infix'>Infix Methods</a>

__Avoid infix notation__ for methods that aren't symbolic methods (i.e. operator overloading).
```scala
// Correct
list.map(func)
string.contains("foo")

// Wrong
list map (func)
string contains "foo"

// But overloaded operators should be invoked in infix style
arrayBuffer += elem
```


### <a name='anonymous'>Anonymous Methods</a>

__Avoid excessive parentheses and curly braces__ for anonymous methods.
```scala
// Correct
list.map { item =>
  ...
}

// Correct
list.map(item => ...)

// Wrong
list.map(item => {
  ...
})

// Wrong
list.map { item => {
  ...
}}

// Wrong
list.map({ item => ... })
```


## <a name='lang'>Scala Language Features</a>

### <a name='case_class_immutability'>Case Classes and Immutability</a>

Case classes are regular classes but extended by the compiler to automatically support:
- Public getters for constructor parameters
- Copy constructor
- Pattern matching on constructor parameters
- Automatic toString/hash/equals implementation

Constructor parameters should NOT be mutable for case classes. Instead, use copy constructor.
Having mutable case classes can be error prone, e.g. hash maps might place the object in the wrong bucket using the old hash code.
```scala
// This is OK
case class Person(name: String, age: Int)

// This is NOT OK
case class Person(name: String, var age: Int)

// To change values, use the copy constructor to create a new instance
val p1 = Person("Peter", 15)
val p2 = p1.copy(age = 16)
```


### <a name='apply_method'>apply Method</a>

Avoid defining apply methods on classes. These methods tend to make the code less readable, especially for people less familiar with Scala. It is also harder for IDEs (or grep) to trace. In the worst case, it can also affect correctness of the code in surprising ways, as demonstrated in [Parentheses](#parentheses).

It is acceptable to define apply methods on companion objects as factory methods. In these cases, the apply method should return the companion class type.
```scala
object TreeNode {
  // This is OK
  def apply(name: String): TreeNode = ...

  // This is bad because it does not return a TreeNode
  def apply(name: String): String = ...
}
```


### <a name='override_modifier'>override Modifier</a>
Always add override modifier for methods, both for overriding concrete methods and implementing abstract methods. The Scala compiler does not require `override` for implementing abstract methods. However, we should always add `override` to make the override obvious, and to avoid accidental non-overrides due to non-matching signatures.
```scala
trait Parent {
  def hello(data: Map[String, String]): Unit = {
    print(data)
  }
}

class Child extends Parent {
  import scala.collection.Map

  // The following method does NOT override Parent.hello,
  // because the two Maps have different types.
  // If we added "override" modifier, the compiler would've caught it.
  def hello(data: Map[String, String]): Unit = {
    print("This is supposed to override the parent method, but it is actually not!")
  }
}
```



### <a name='destruct_bind'>Destructuring Binds</a>

Destructuring bind (sometimes called tuple extraction) is a convenient way to assign two variables in one expression.
```scala
val (a, b) = (1, 2)
```

However, do NOT use them in constructors, especially when `a` and `b` need to be marked transient. The Scala compiler generates an extra Tuple2 field that will not be transient for the above example.
```scala
class MyClass {
  // This will NOT work because the compiler generates a non-transient Tuple2
  // that points to both a and b.
  @transient private val (a, b) = someFuncThatReturnsTuple2()
}
```


### <a name='call_by_name'>Call by Name</a>

__Avoid using call by name__. Use `() => T` explicitly.

Background: Scala allows method parameters to be defined by-name, e.g. the following would work:
```scala
def print(value: => Int): Unit = {
  println(value)
  println(value + 1)
}

var a = 0
def inc(): Int = {
  a += 1
  a
}

print(inc())
```
in the above code, `inc()` is passed into `print` as a closure and is executed (twice) in the print method, rather than being passed in as a value `1`. The main problem with call-by-name is that the caller cannot differentiate between call-by-name and call-by-value, and thus cannot know for sure whether the expression will be executed or not (or maybe worse, multiple times). This is especially dangerous for expressions that have side-effect.


### <a name='multi-param-list'>Multiple Parameter Lists</a>

__Avoid using multiple parameter lists__. They complicate operator overloading, and can confuse programmers less familiar with Scala. For example:

```scala
// Avoid this!
case class Person(name: String, age: Int)(secret: String)
```

One notable exception is the use of a 2nd parameter list for implicits when defining low-level libraries. That said, [implicits should be avoided](#implicits)!


### <a name='symbolic_methods'>Symbolic Methods (Operator Overloading)</a>

__Do NOT use symbolic method names__, unless you are defining them for natural arithmetic operations (e.g. `+`, `-`, `*`, `/`). Under no other circumstances should they be used. Symbolic method names make it very hard to understand the intent of the methods. Consider the following two examples:
```scala
// symbolic method names are hard to understand
channel ! msg
stream1 >>= stream2

// self-evident what is going on
channel.send(msg)
stream1.join(stream2)
```


### <a name='type_inference'>Type Inference</a>

Scala type inference, especially left-side type inference and closure inference, can make code more concise. That said, there are a few cases where explicit typing should be used:

- __Public methods should be explicitly typed__, otherwise the compiler's inferred type can often surprise you.
- __Implicit methods should be explicitly typed__, otherwise it can crash the Scala compiler with incremental compilation.
- __Variables or closures with non-obvious types should be explicitly typed__. A good litmus test is that explicit types should be used if a code reviewer cannot determine the type in 3 seconds.


### <a name='return'>Return Statements</a>

__Avoid using return in closures__. `return` is turned into ``try/catch`` of ``scala.runtime.NonLocalReturnControl`` by the compiler. This can lead to unexpected behaviors. Consider the following example:
  ```scala
  def receive(rpc: WebSocketRPC): Option[Response] = {
    tableFut.onComplete { table =>
      if (table.isFailure) {
        return None // Do not do that!
      } else { ... }
    }
  }
  ```
  the `.onComplete` method takes the anonymous closure `{ table => ... }` and passes it to a different thread. This closure eventually throws the `NonLocalReturnControl` exception that is captured __in a different thread__ . It has no effect on the poor method being executed here.

However, there are a few cases where `return` is preferred.

- Use `return` as a guard to simplify control flow without adding a level of indentation
  ```scala
  def doSomething(obj: Any): Any = {
    if (obj eq null) {
      return null
    }
    // do something ...
  }
  ```

- Use `return` to terminate a loop early, rather than constructing status flags
  ```scala
  while (true) {
    if (cond) {
      return
    }
  }
  ```

### <a name='recursion'>Recursion and Tail Recursion</a>

__Avoid using recursion__, unless the problem can be naturally framed recursively (e.g. graph traversal, tree traversal).

For methods that are meant to be tail recursive, apply `@tailrec` annotation to make sure the compiler can check it is tail recursive. (You will be surprised how often seemingly tail recursive code is actually not tail recursive due to the use of closures and functional transformations.)

Most code is easier to reason about with a simple loop and explicit state machines. Expressing it with tail recursions (and accumulators) can make it more verbose and harder to understand.  For example, the following imperative code is more readable than the tail recursive version:

```scala
// Tail recursive version.
def max(data: Array[Int]): Int = {
  @tailrec
  def max0(data: Array[Int], pos: Int, max: Int): Int = {
    if (pos == data.length) {
      max
    } else {
      max0(data, pos + 1, if (data(pos) > max) data(pos) else max)
    }
  }
  max0(data, 0, Int.MinValue)
}

// Explicit loop version
def max(data: Array[Int]): Int = {
  var max = Int.MinValue
  for (v <- data) {
    if (v > max) {
      max = v
    }
  }
  max
}
```


### <a name='implicits'>Implicits</a>

__Avoid using implicits__, unless:
- you are building a domain-specific language
- you are using it for implicit type parameters (e.g. `ClassTag`, `TypeTag`)
- you are using it private to your own class to reduce verbosity of converting from one type to another (e.g. Scala closure to Java closure)

When implicits are used, we must ensure that another engineer who did not author the code can understand the semantics of the usage without reading the implicit definition itself. Implicits have very complicated resolution rules and make the code base extremely difficult to understand. From Twitter's Effective Scala guide: "If you do find yourself using implicits, always ask yourself if there is a way to achieve the same thing without their help."

If you must use them (e.g. enriching some DSL), do not overload implicit methods, i.e. make sure each implicit method has distinct names, so users can selectively import them.
```scala
// Don't do the following, as users cannot selectively import only one of the methods.
object ImplicitHolder {
  def toRdd(seq: Seq[Int]): RDD[Int] = ...
  def toRdd(seq: Seq[Long]): RDD[Long] = ...
}

// Do the following:
object ImplicitHolder {
  def intSeqToRdd(seq: Seq[Int]): RDD[Int] = ...
  def longSeqToRdd(seq: Seq[Long]): RDD[Long] = ...
}
```

### <a name='symbol'>Symbol Literals</a>

__Avoid using symbol literals__. Symbol literals (e.g. `'column`) were deprecated as of Scala 2.13 by [Proposal to deprecate and remove symbol literals](https://contributors.scala-lang.org/t/proposal-to-deprecate-and-remove-symbol-literals/2953). Apache Spark used to leverage this syntax to provide DSL; however, now it started to remove this deprecated usage away. See also [SPARK-29392](https://issues.apache.org/jira/browse/SPARK-29392).


## <a name='exception'>Exception Handling (Try vs try)</a>

- Do NOT catch Throwable or Exception. Use `scala.util.control.NonFatal`:
  ```scala
  try {
    ...
  } catch {
    case NonFatal(e) =>
      // handle exception; note that NonFatal does not match InterruptedException
    case e: InterruptedException =>
      // handle InterruptedException
  }
  ```
  This ensures that we do not catch `NonLocalReturnControl` (as explained in [Return Statements](#return-statements)).

### <a name='option'>Options</a>

- Use `Option` when the value can be empty. Compared with `null`, an `Option` explicitly states in the API contract that the value can be `None`.
- When constructing an `Option`, use `Option` rather than `Some` to guard against `null` values.
  ```scala
  def myMethod1(input: String): Option[String] = Option(transform(input))

  // This is not as robust because transform can return null, and then
  // myMethod2 will return Some(null).
  def myMethod2(input: String): Option[String] = Some(transform(input))
  ```
- Do not use None to represent exceptions. Instead, throw exceptions explicitly.
- Do not call `get` directly on an `Option`, unless you know absolutely for sure the `Option` has some value.

## <a name='testing'>Testing</a>

### <a name='testing-intercepting'>Intercepting Exceptions</a>

When testing that performing a certain action (say, calling a function with an invalid argument) throws an exception, be as specific as possible about the type of exception you expect to be thrown. You should NOT simply `intercept[Exception]` or `intercept[Throwable]` (to use the ScalaTest syntax), as this will just assert that _any_ exception is thrown. Often times, this will just catch errors you made when setting up your testing mocks and your test will silently pass without actually checking the behavior you want to verify.

  ```scala
  // This is WRONG
  intercept[Exception] {
    thingThatThrowsException()
  }

  // This is CORRECT
  intercept[MySpecificTypeOfException] {
    thingThatThrowsException()
  }
  ```

If you cannot be more specific about the type of exception that the code will throw, that is often a sign of code smell. You should either test at a lower level or modify the underlying code to throw a more specific exception.

## <a name='misc'>Miscellaneous</a>

### <a name='misc_currentTimeMillis_vs_nanoTime'>Prefer nanoTime over currentTimeMillis</a>

When computing a *duration* or checking for a *timeout*, avoid using `System.currentTimeMillis()`. Use `System.nanoTime()` instead, even if you are not interested in sub-millisecond precision.

`System.currentTimeMillis()` returns current wallclock time and will follow changes to the system clock. Thus, negative wallclock adjustments can cause timeouts to "hang" for a long time (until wallclock time has caught up to its previous value again).  This can happen when ntpd does a "step" after the network has been disconnected for some time. The most canonical example is during system bootup when DHCP takes longer than usual. This can lead to failures that are really hard to understand/reproduce. `System.nanoTime()` is guaranteed to be monotonically increasing irrespective of wallclock changes.

Caveats:
- Never serialize an absolute `nanoTime()` value or pass it to another system. The absolute value is meaningless and system-specific and resets when the system reboots.
- The absolute `nanoTime()` value is not guaranteed to be positive (but `t2 - t1` is guaranteed to yield the right result)
- `nanoTime()` rolls over every 292 years. So if your Spark job is going to take a really long time, you may need something else :)

### <a name='misc_well_tested_method'>Prefer existing well-tested methods over reinventing the wheel</a>

When there is an existing well-tesed method and it doesn't cause any performance issue, prefer to use it. Reimplementing such method may introduce bugs and requires spending time testing it (maybe we don't even remember to test it!).

  ```scala
  val beginNs = System.nanoTime()
  // Do something
  Thread.sleep(1000)
  val elapsedNs = System.nanoTime() - beginNs

  // This is WRONG. It uses magic numbers and is pretty easy to make mistakes
  val elapsedMs = elapsedNs / 1000 / 1000

  // Use the Java TimeUnit API. This is CORRECT
  import java.util.concurrent.TimeUnit
  val elapsedMs2 = TimeUnit.NANOSECONDS.toMillis(elapsedNs)

  // Use the Scala Duration API. This is CORRECT
  import scala.concurrent.duration._
  val elapsedMs3 = elapsedNs.nanos.toMillis
  ```

Exceptions:
- Using an existing well-tesed method requires adding a new dependency. If such method is pretty simple, reimplementing it is better than adding a dependency. But remember to test it.
- The existing method is not optimized for our usage and is too slow. But benchmark it first, avoid premature optimization.
