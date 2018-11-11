---
type: doc
layout: reference
category: "Syntax"
title: "Reflection"
---

# Reflection

Reflection ：反射

Reflection is a set of language and library features that allows for introspecting the structure of your own program at runtime. Kotlin makes functions and properties first-class citizens in the language, and introspecting them (i.e. learning a name or a type of a property or function at runtime) is closely intertwined with simply using a functional or reactive style.

反射是一套語言和函式庫功能，允許在運行時反思你擁有的程式結構。 Kotlin 在語言上產生函數和屬性為一等公民，並反思它們 (即是在運行時學習名稱，或函數和屬性類型) 與簡單的使用函數和反應式風格密切相關。

> On the Java platform, the runtime component required for using the reflection features is distributed as a separate JAR file (`kotlin-reflect.jar`). This is done to reduce the required size of the runtime library for applications that do not use reflection features. If you do use reflection, please make sure that the .jar file is added to the classpath of your project.
>
> 在 Java 平台上，運行時元件需要使用反射功能，反射功能被分發作為單獨 JAR 檔案 (`kotlin-reflect.jar`) 。這樣做為了減少應用程式不使用反射功能運行時函式庫的大小。如果你使用反射，請確保 .jar 檔案被新增到你專案的類別路徑。

## Class References

Class References ：類別類別參照 (引用)

The most basic reflection feature is getting the runtime reference to a Kotlin class. To obtain the reference to a statically known Kotlin class, you can use the _class literal_ syntax:

最基本反射功能是取得 Kotlin 類別運行時的引用。去獲取靜態已知的 Kotlin 類別引用，你可以使用類別文字語法：

``` kotlin
val c = MyClass::class // MyClass 為自己寫好的已知靜態類別
```

The reference is a value of type [KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html).

參照是類型 [KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html) 的值。

Note that a Kotlin class reference is not the same as a Java class reference. To obtain a Java class reference, use the `.java` property on a `KClass` instance.

注意： Kotlin 類別引用與 Java 類別引用不同。獲取 Java 類別引用，在 Kotlin 實例使用 `.java` 屬性。

## Bound Class References (since 1.1)

Bound Class References (since 1.1) ：受約束的類別參照 (引用) (從 1.1)

You can get the reference to a class of a specific object with the same `::class` syntax by using the object as a receiver:

你可以透過使用物件作為 receiver ，指定物件使用相同 `::class` 語法取得類別引用，以下`widget::class.qualifiedName`。

``` kotlin
val widget: Widget = ...
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

You obtain the reference to an exact class of an object, for instance `GoodWidget` or `BadWidget`, despite the type of the receiver expression (`Widget`).

你可以獲取物件準確的類別引用，例如 `GoodWidget` 或 `GoodWidget` 類別，雖然 receiver 表達式 (`Widget`) 的類型

## Callable references

Callable references ：可調用的參照 (引用)

References to functions, properties, and constructors, apart from introspecting the program structure, can also be called or used as instances of [function types](lambdas.md#function-types).

除非反思程式結構之外，函數、屬性、建構元的引用也可以被調用或用作[函數類型](lambdas.md#function-types)的實例。

The common supertype for all callable references is [`KCallable<out R>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-callable/index.html), where `R` is the return value type, which is the property type for properties, and the constructed type for constructors. 

常見的所有可調用參照的超 (父) 類型是 [`KCallable<out R>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-callable/index.html) ，當中的 `R` 是回傳值，代表屬性的屬性類型，以及建構元的建構類型。

### Function References

Function References ：函數參照 (引用)

When we have a named function declared like this:

當我們有像這樣宣告已命名函數時：

``` kotlin
fun isOdd(x: Int) = x % 2 != 0
```

We can easily call it directly (`isOdd(5)`), but we can also use it as a function type value, e.g. pass it to another function. To do this, we use the `::` operator:

我們可以容易直接的調用它 (`isOdd(5)`) ，但我們也可以使用它為函數類型值，例如傳遞它到其他的函數。要做到這點，我們使用 `::` 運算符：

``` kotlin
fun isOdd(x: Int) = x % 2 != 0

fun main(args: Array<String>) {
//sampleStart
    val numbers = listOf(1, 2, 3)
    println(numbers.filter(::isOdd)) //調用函數參照
//sampleEnd
}
```

Here `::isOdd` is a value of function type `(Int) -> Boolean`.

這裡 `::isOdd` 是函數類型 `(Int) -> Boolean` 的值。

Function references belong to one of the [`KFunction<out R>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-function/index.html) subtypes, depending on the parameter count, e.g. `KFunction3<T1, T2, T3, R>`.

函數參照屬於 [`KFunction<out R>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-function/index.html) 子類型之一，取決於參數的數量，例如 `KFunction3<T1, T2, T3, R>` 。

`::` can be used with overloaded functions when the expected type is known from the context.
For example:

當從內文中知道預期的類型時， `::` 可以與多載函數一起使用。例如：

``` kotlin
fun main(args: Array<String>) {
    //sampleStart
    //多載函數
    fun isOdd(x: Int) = x % 2 != 0
    fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"
    
    val numbers = listOf(1, 2, 3)
    println(numbers.filter(::isOdd)) // refers to isOdd(x: Int)
    
    val strings = listOf("brillig", "slithy", "tove")
	println(strings.filter(::isOdd)) // refers to isOdd(x: String)
    //sampleEnd
}

//ans: 
//[1, 3]
//[brillig, slithy, tove]
```

Alternatively, you can provide the necessary context by storing the method reference in a variable with an explicitly specified type:

或者，你可以透過儲存方法參照在變數，變數使用明確的指定類型，提供必要的內文：

``` kotlin
val predicate: (String) -> Boolean = ::isOdd   // refers to isOdd(x: String)
```

If we need to use a member of a class, or an extension function, it needs to be qualified, e.g. `String::toCharArray`.

如果我們需要使用類別的成員，或擴展函數，它需要被修飾，例如 `String::toCharArray` 。

Note that even if you initialize a variable with a reference to an extension function, the inferred function type will have no receiver (it will have an additional parameter accepting a receiver object). To have a function type with receiver instead, specify the type explicitly:

注意：即使你使用擴展函數的參照初始化變數，推斷函數類型將沒有 receiver (它將接收 receiver 物件有額外的參數) 。反而 receiver 有函數類型，明確的指定類型：

``` kotlin
val isEmptyStringList: List<String>.() -> Boolean = List::isEmpty //List 是 receiver 類型，直接指定
```

### Example: Function Composition

Example: Function Composition ：範例：函數組合

Consider the following function:

考慮以下函數：

``` kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}
```

It returns a composition of two functions passed to it: `compose(f, g) = f(g(*))`. Now, you can apply it to callable references:

它回傳傳遞給它的兩個函數組合： `compose(f, g) = f(g(*))` 。現在，你可以應用於可調用的參照：

``` kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}

fun isOdd(x: Int) = x % 2 != 0

fun main(args: Array<String>) {
//sampleStart
    fun length(s: String) = s.length
    
    val oddLength = compose(::isOdd, ::length) //先調用 ::length 再調用 ::isOdd
    val strings = listOf("a", "ab", "abc")
    
    println(strings.filter(oddLength)) //filter 用調用參照 oddLength = compose(...)
//sampleEnd
}
```

### Property References

Property References ：屬性參照 (引用)

To access properties as first-class objects in Kotlin, we can also use the `::` operator:

<div class="sample" markdown="1" theme="idea">
​``` kotlin
val x = 1

fun main(args: Array<String>) {
​    println(::x.get())
​    println(::x.name) 
}
```
</div>

The expression `::x` evaluates to a property object of type `KProperty<Int>`, which allows us to read its
value using `get()` or retrieve the property name using the `name` property. For more information, please refer to
the [docs on the `KProperty` class](/api/latest/jvm/stdlib/kotlin.reflect/-k-property/index.html).

For a mutable property, e.g. `var y = 1`, `::y` returns a value of type [`KMutableProperty<Int>`](/api/latest/jvm/stdlib/kotlin.reflect/-k-mutable-property/index.html),
which has a `set()` method:

<div class="sample" markdown="1" theme="idea">
​``` kotlin
var y = 1

fun main(args: Array<String>) {
​    ::y.set(2)
​    println(y)
}
```
</div>            

A property reference can be used where a function with one parameter is expected:

<div class="sample" markdown="1" theme="idea">
​``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val strs = listOf("a", "bc", "def")
    println(strs.map(String::length))
//sampleEnd
}
```
</div>

To access a property that is a member of a class, we qualify it:

<div class="sample" markdown="1" theme="idea">
``` kotlin
fun main(args: Array<String>) {
//sampleStart
    class A(val p: Int)
    val prop = A::p
    println(prop.get(A(1)))
//sampleEnd
}
```
</div>

For an extension property:

<div class="sample" markdown="1" theme="idea" auto-indent="false">
``` kotlin
val String.lastChar: Char
    get() = this[length - 1]

fun main(args: Array<String>) {
​    println(String::lastChar.get("abc"))
}
```
</div>

### Interoperability With Java Reflection

On the Java platform, standard library contains extensions for reflection classes that provide a mapping to and from Java
  reflection objects (see package `kotlin.reflect.jvm`).
For example, to find a backing field or a Java method that serves as a getter for a Kotlin property, you can say something like this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
import kotlin.reflect.jvm.*
 
class A(val p: Int)
 
fun main(args: Array<String>) {
    println(A::p.javaGetter) // prints "public final int A.getP()"
    println(A::p.javaField)  // prints "private final int A.p"
}
```
</div>

To get the Kotlin class corresponding to a Java class, use the `.kotlin` extension property:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```
</div>

### Constructor References

Constructors can be referenced just like methods and properties. They can be used wherever an object of function type 
is expected that takes the same parameters as the constructor and returns an object of the appropriate type. 
Constructors are referenced by using the `::` operator and adding the class name. Consider the following function 
that expects a function parameter with no parameters and return type `Foo`:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
class Foo

fun function(factory: () -> Foo) {
​    val x: Foo = factory()
}
```
</div>

Using `::Foo`, the zero-argument constructor of the class Foo, we can simply call it like this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
function(::Foo)
```
</div>

Callable references to constructors are typed as one of the 
[`KFunction<out R>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-function/index.html) subtypes
, depending on the parameter count.

## Bound Function and Property References (since 1.1)

You can refer to an instance method of a particular object:

<div class="sample" markdown="1" theme="idea">
``` kotlin 
fun main(args: Array<String>) {
//sampleStart
    val numberRegex = "\\d+".toRegex()
    println(numberRegex.matches("29"))

    val isNumber = numberRegex::matches
    println(isNumber("29"))
//sampleEnd
}
```
</div>

Instead of calling the method `matches` directly we are storing a reference to it.
Such reference is bound to its receiver.
It can be called directly (like in the example above) or used whenever an expression of function type is expected:

<div class="sample" markdown="1" theme="idea">
​``` kotlin 
fun main(args: Array<String>) {
//sampleStart
    val numberRegex = "\\d+".toRegex()
    val strings = listOf("abc", "124", "a70")
    println(strings.filter(numberRegex::matches))
//sampleEnd
}
```
</div>

Compare the types of bound and the corresponding unbound references.
Bound callable reference has its receiver "attached" to it, so the type of the receiver is no longer a parameter:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
val isNumber: (CharSequence) -> Boolean = numberRegex::matches

val matches: (Regex, CharSequence) -> Boolean = Regex::matches
```
</div>

Property reference can be bound as well:

<div class="sample" markdown="1" theme="idea">
​``` kotlin 
fun main(args: Array<String>) {
//sampleStart
    val prop = "abc"::length
    println(prop.get())
//sampleEnd
}
```
</div>

Since Kotlin 1.2, explicitly specifying `this` as the receiver is not necessary: `this::foo` and `::foo` are equivalent.

### Bound constructor references

A bound callable reference to a constructor of an [*inner*{: .keyword} class](nested-classes.html#inner-classes) can 
be obtained by providing an instance of the outer class:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
class Outer {
    inner class Inner
}

val o = Outer()
val boundInnerCtor = o::Inner
```
</div>


```