---
type: doc
layout: reference
category: "Syntax"
title: "Delegated Properties"
---

# Delegated Properties

Delegated Properties ：委外屬性

There are certain common kinds of properties, that, though we can implement them manually every time we need them, 
would be very nice to implement once and for all, and put into a library. Examples include:

有某些常見的屬性種類，雖然我們可以每次在我們需要它們時手動的實作它們，一勞永逸的實作它們並且放入函式庫非常好。範例包括：

* lazy properties: the value gets computed only upon first access;
  懶惰屬性：只根據第一次存取時才運算獲取值；
* observable properties: listeners get notified about changes to this property;
  可觀察的屬性：監聽者會收到這個屬性改變的通知；
* storing properties in a map, instead of a separate field for each property.
  在一個 map 儲存屬性，代替每個屬性的一個單獨欄位。

To cover these (and other) cases, Kotlin supports _delegated properties_:

為了涵蓋這些 (和其他) 情況， Kotlin 支援委外屬性：

``` kotlin
class Example {
    var p: String by Delegate()
}
```

The syntax is: `val/var <property name>: <Type> by <expression>`. The expression after *by* is the _delegate_, because `get()` (and `set()`) corresponding to the property will be delegated to its `getValue()` and `setValue()` methods. Property delegates don’t have to implement any interface, but they have to provide a `getValue()` function (and `setValue()` --- for *var*s). For example:

語法是： `val/var <property name>: <Type> by <expression>` 。在 `by` 之後的表達式是委外的類別，因為對應屬性的 `get()` (和 `set()`) 將委外給它的 `getValue()` 和 `setValue` 方法。屬性委外不必實作任何介面，但是它們必須提供一個 `getValue()` 函數 (和 `setValue()`) ---於 `var` 可變的屬性。範例：

``` kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

When we read from `p` that delegates to an instance of `Delegate`, the `getValue()` function from `Delegate` is called, so that its first parameter is the object we read `p` from and the second parameter holds a description of `p` itself (e.g. you can take its name). For example:

當我們從 `p` 讀取時，委外給一個 `Delegate` 實例，調用 `Delegate` 的 `getValue()` 函數，因此 `getValue()` 第一個參數是我們讀取 `p` 的物件，第二個參數持有 `p` 本身的描述 (例如：你可以取得它的屬性名子) 。範例：

``` kotlin
val e = Example()
println(e.p)
```

This prints:

這印出：

```
Example@33a17727, thank you for delegating ‘p’ to me!
```

Similarly, when we assign to `p`, the `setValue()` function is called. The first two parameters are the same, and the third holds the value being assigned:

類似地，當我們賦值給 `p` ，調用 `setValue()` 函數。首先兩個參數相同，且第三個參數保存被分配的值 "NEW"： 

``` kotlin
e.p = "NEW"
```

This prints

```
NEW has been assigned to ‘p’ in Example@33a17727.
```

The specification of the requirements to the delegated object can be found [below](delegated-properties.md#property-delegate-requirements).

可以從[下面](delegated-properties.md#property-delegate-requirements)找到對委外物件的要求規範。

Note that since Kotlin 1.1 you can declare a delegated property inside a function or code block, it shouldn't necessarily be a member of a class. Below you can find [the example](delegated-properties.md#local-delegated-properties-since-11).

注意：從 Kotlin 1.1 版，你可以在一個函數或代碼區塊內宣告一個委外屬性，它不一定是一個類別的成員，你可以在下面找到該[範例](delegated-properties.md#local-delegated-properties-since-11)。

## Standard Delegates

Standard Delegates ：標準化委外

The Kotlin standard library provides factory methods for several useful kinds of delegates.

Kotlin 標準函式庫有一些有用的委外種類提供工廠方法。

### Lazy

Lazy ：惰性初始化，只在第一次調用時才執行初始化程序，後續都是取得快取

[`lazy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html) is a function that takes a lambda and returns an instance of `Lazy<T>` which can serve as a delegate for implementing a lazy property: the first call to `get()` executes the lambda passed to `lazy()` and remembers the result, subsequent calls to `get()` simply return the remembered result. 

[`lazy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html) 是一個函數，參數為一個函數 `{}` ，函數帶一個 Lambda 表達式 `{...}` 並且在表達式 `{...}` 最後回傳一個 `Lazy<T>` 的實例，這個實例可以擔任為實作一個惰性屬性的委外：首次調用 `get()` 執行傳遞給 `lazy()` 的 Lambda `{}` 並記住這個結果，後續調用到 `get()` 只是回傳已記住的結果。

``` kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello" // return Lazy<String>
}

fun main(args: Array<String>) {
    println(lazyValue)
    println(lazyValue)
}
//ans：
//computed! 只在第一次執行
//Hello //回傳值
//Hello //回傳值
```

By default, the evaluation of lazy properties is **synchronized**: the value is computed only in one thread, and all threads will see the same value. If the synchronization of initialization delegate is not required, so that multiple threads can execute it simultaneously, pass `LazyThreadSafetyMode.PUBLICATION` as a parameter to the `lazy()` function. And if you're sure that the initialization will always happen on a single thread, you can use `LazyThreadSafetyMode.NONE` mode, which doesn't incur any thread-safety guarantees and the related overhead.

預設上，惰性屬性的執行結果是同步的：這個值是只在一個線程中計算，且所有的線程將看到相同的值。如果不需要初始化委外的同步，以便多線程同時執行它，傳遞 `LazyThreadSafetyMode.PUBLICATION` 為參數到 `lazy()` 函數。並且如果你是確定初始化將總是發生在單線程，你可以使用 `LazyThreadSafetyMode.NONE` 模式，此模式不會遭遇任何線程-安全保證和相關開銷。

**原始碼，第一個參數可以設定線程處理**

``` kotlin
actual fun <T> lazy(
    mode: LazyThreadSafetyMode, 
    initializer: () -> T
): Lazy<T> (source)

// example : val lazyValue: String by lazy(LazyThreadSafetyMode.PUBLICATION){}
```

### Observable

Observable ：被觀察者，負責發送事件

[`Delegates.observable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/observable.html) takes two arguments: the initial value and a handler for modifications. The handler gets called every time we assign to the property (_after_ the assignment has been performed). It has three parameters: a property being assigned to, the old value and the new one:

[`Delegates.observable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/observable.html) 帶兩個參數：初始化值和修改後的處理程序表達式 `{}` 。每次我們分配值給屬性時都會調用處理程序 (在執行分配值之後) 。處理程序 `{prop, old, new -> ...}` 有三個參數：一個屬性被分配 (屬性本身) ，舊值和新值：

``` kotlin
import kotlin.properties.Delegates
class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main(args: Array<String>) {
    val user = User()
    user.name = "first"
    user.name = "second"
}
//ans:
//<no name> -> first
//first -> second
```

If you want to be able to intercept an assignment and "veto" it, use [`vetoable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html) instead of `observable()`. The handler passed to the `vetoable` is called _before_ the assignment of a new property value has been performed.

如果我們想要能攔截分配並否決它，使用 [`vetoable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html) 替代 `observable()` 。處理程序傳遞給 `vetoable` 在執行新的屬性值給值前調用。

**`vetoable` 在分配值之「前」執行，可用於滿足某些條件才能分配值，例如：設定數量時要大於 1 否則就不執行**

**`Observable` 在分配值之「後」執行，用於通知接下來的處理，例如：數量輸入完計算總價**

## Storing Properties in a Map

Storing Properties in a Map ：儲存在映射的屬性

One common use case is storing the values of properties in a map. This comes up often in applications like parsing JSON or doing other “dynamic” things. In this case, you can use the map instance itself as the delegate for a delegated property.

一個常見的使用情況是存儲在 `Map` 中的屬性值。這通常出現在解析 JSON 或執行其他的 "動態" 事情的應用程序中。在這種情況，你可以為委外屬性使用 `Map`  實例本身為派外處理 `by map`。

``` kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

In this example, the constructor takes a map:

在這範例，建構元帶一個 `Map` `mapof(..)`：

``` kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```

Delegated properties take values from this map (by the string keys --– names of properties):

委外屬性從這個 `Map` 帶值到屬性 (透過字串關鍵字對應到屬性的名稱) ：

``` kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}

fun main(args: Array<String>) {
    val user = User(mapOf(
        "name" to "John Doe",
        "age"  to 25
    ))
//sampleStart
    println(user.name) // Prints "John Doe"
    println(user.age)  // Prints 25
//sampleEnd
}
```

This works also for *var*’s properties if you use a `MutableMap` instead of read-only `Map`:

如果你使用一個可變的 `MutableMap` 代替 唯讀的 `Map`，這也適用 `var` 屬性：

``` kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```

## Local Delegated Properties (since 1.1)

You can declare local variables as delegated properties.
For instance, you can make a local variable lazy:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)

    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
```
</div>

The `memoizedFoo` variable will be computed on the first access only.
If `someCondition` fails, the variable won't be computed at all.

## Property Delegate Requirements

Here we summarize requirements to delegate objects. 

For a **read-only** property (i.e. a *val*{:.keyword}), a delegate has to provide a function named `getValue` that takes the following parameters:

* `thisRef` --- must be the same or a supertype of the _property owner_ (for extension properties --- the type being extended);
* `property` --- must be of type `KProperty<*>` or its supertype.
 
this function must return the same type as property (or its subtype).

For a **mutable** property (a *var*{:.keyword}), a delegate has to _additionally_ provide a function named `setValue` that takes the following parameters:
 
* `thisRef` --- same as for `getValue()`;
* `property` --- same as for `getValue()`;
* new value --- must be of the same type as a property or its supertype.
 
`getValue()` and/or `setValue()` functions may be provided either as member functions of the delegate class or extension functions.
The latter is handy when you need to delegate property to an object which doesn't originally provide these functions.
Both of the functions need to be marked with the `operator` keyword.

The delegate class may implement one of the interfaces `ReadOnlyProperty` and `ReadWriteProperty` containing the required `operator` methods.
These interfaces are declared in the Kotlin standard library:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
​``` kotlin
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```
</div>

### Translation Rules

Under the hood for every delegated property the Kotlin compiler generates an auxiliary property and delegates to it.
For instance, for the property `prop` the hidden property `prop$delegate` is generated, and the code of the accessors simply delegates to this additional property:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
``` kotlin
class C {
    var prop: Type by MyDelegate()
}

// this code is generated by the compiler instead:
class C {
    private val prop$delegate = MyDelegate()
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```
</div>

The Kotlin compiler provides all the necessary information about `prop` in the arguments: the first argument `this` refers to an instance of the outer class `C` and `this::prop` is a reflection object of the `KProperty` type describing `prop` itself.

Note that the syntax `this::prop` to refer a [bound callable reference](reflection.html#bound-function-and-property-references-since-11) in the code directly is available only since Kotlin 1.1.  

### Providing a delegate (since 1.1)

By defining the `provideDelegate` operator you can extend the logic of creating the object to which the property implementation is delegated.
If the object used on the right hand side of `by` defines `provideDelegate` as a member or extension function, that function will be
called to create the property delegate instance.

One of the possible use cases of `provideDelegate` is to check property consistency when the property is created, not only in its getter or setter.

For example, if you want to check the property name before binding, you can write something like this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
class ResourceDelegate<T> : ReadOnlyProperty<MyUI, T> {
    override fun getValue(thisRef: MyUI, property: KProperty<*>): T { ... }
}
    
class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(
            thisRef: MyUI,
            prop: KProperty<*>
    ): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, prop.name)
        // create delegate
        return ResourceDelegate()
    }

    private fun checkProperty(thisRef: MyUI, name: String) { ... }
}

class MyUI {
    fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { ... }

    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```
</div>

The parameters of `provideDelegate` are the same as for `getValue`:

* `thisRef` --- must be the same or a supertype of the _property owner_ (for extension properties --- the type being extended);
* `property` --- must be of type `KProperty<*>` or its supertype.

The `provideDelegate` method is called for each property during the creation of the `MyUI` instance, and it performs the necessary validation right away.

Without this ability to intercept the binding between the property and its delegate, to achieve the same functionality
you'd have to pass the property name explicitly, which isn't very convenient:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
// Checking the property name without "provideDelegate" functionality
class MyUI {
    val image by bindResource(ResourceID.image_id, "image")
    val text by bindResource(ResourceID.text_id, "text")
}

fun <T> MyUI.bindResource(
        id: ResourceID<T>,
        propertyName: String
): ReadOnlyProperty<MyUI, T> {
   checkProperty(this, propertyName)
   // create delegate
}
```
</div>

In the generated code, the `provideDelegate` method is called to initialize the auxiliary `prop$delegate` property.
Compare the generated code for the property declaration `val prop: Type by MyDelegate()` with the generated code 
[above](delegated-properties.html#translation-rules) (when the `provideDelegate` method is not present):

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
​``` kotlin
class C {
    var prop: Type by MyDelegate()
}

// this code is generated by the compiler 
// when the 'provideDelegate' function is available:
class C {
    // calling "provideDelegate" to create the additional "delegate" property
    private val prop$delegate = MyDelegate().provideDelegate(this, this::prop)
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```
</div>

Note that the `provideDelegate` method affects only the creation of the auxiliary property and doesn't affect the code generated for getter or setter.
